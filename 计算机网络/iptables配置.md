### 简介
iptables其实不是真正意义上的软件防火墙，它是一个客户端代理，将用户设定的安全规则给真正的防火墙netfilter，让其执行。netfilter是Linux操作系统的一个数据包处理模块，有以下几个功能：
1. NAT
2. 数据包内容修改
3. 数据包过滤

### 表、链、规则、动作
iptables中有如下几个概念：
1. 表
(1) filter：一般的过滤功能
(2) nat:用于nat功能（端口映射，地址映射等）
(3) mangle:用于对特定数据包的修改
(4) raw:有限级最高，设置raw时一般是为了不再让iptables做数据包的链接跟踪处理，提高性能
2.  链
(1). PREROUTING:数据包进入路由表之前
(2). INPUT:通过路由表后目的地为本机
(3). FORWARDING:通过路由表后，目的地不为本机
(4). OUTPUT:由本机产生，向外转发
(5). POSTROUTIONG:发送到网卡接口之前
3.  动作
(1). ACCEPT：接收数据包
(2). DROP：丢弃数据包
(3). REDIRECT：重定向、映射、透明代理
(4). SNAT：源地址转换
(5). DNAT：目标地址转换
(6). MASQUERADE：IP伪装（NAT），用于ADSL
(7). LOG：日志记录

如下图所示为数据包的流向（图片来自网络）

![这里写图片描述](https://img-blog.csdn.net/20180716203223912?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JMVUU1OTQ1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

基本步骤如下： 
1. 数据包到达网络接口，比如 eth0。 
2. 进入 raw 表的 PREROUTING 链，这个链的作用是赶在连接跟踪之前处理数据包。 
3. 如果进行了连接跟踪，在此处理。 
4. 进入 mangle 表的 PREROUTING 链，在此可以修改数据包，比如 TOS 等。 
5. 进入 nat 表的 PREROUTING 链，可以在此做DNAT，但不要做过滤。 
6. 决定路由，看是交给本地主机还是转发给其它主机。 

到了这里我们就得分两种不同的情况进行讨论了，一种情况就是数据包要转发给其它主机，这时候它会依次经过： 
7. 进入 mangle 表的 FORWARD 链，这里也比较特殊，这是在第一次路由决定之后，在进行最后的路由决定之前，我们仍然可以对数据包进行某些修改。 
8. 进入 filter 表的 FORWARD 链，在这里我们可以对所有转发的数据包进行过滤。需要注意的是：经过这里的数据包是转发的，方向是双向的。 
9. 进入 mangle 表的 POSTROUTING 链，到这里已经做完了所有的路由决定，但数据包仍然在本地主机，我们还可以进行某些修改。 
10. 进入 nat 表的 POSTROUTING 链，在这里一般都是用来做 SNAT ，不要在这里进行过滤。 
11. 进入出去的网络接口。完毕。 

另一种情况是，数据包就是发给本地主机的，那么它会依次穿过： 
7. 进入 mangle 表的 INPUT 链，这里是在路由之后，交由本地主机之前，我们也可以进行一些相应的修改。 
8. 进入 filter 表的 INPUT 链，在这里我们可以对流入的所有数据包进行过滤，无论它来自哪个网络接口。 
9. 交给本地主机的应用程序进行处理。 
10. 处理完毕后进行路由决定，看该往那里发出。 
11. 进入 raw 表的 OUTPUT 链，这里是在连接跟踪处理本地的数据包之前。 
12. 连接跟踪对本地的数据包进行处理。 
13. 进入 mangle 表的 OUTPUT 链，在这里我们可以修改数据包，但不要做过滤。 
14. 进入 nat 表的 OUTPUT 链，可以对防火墙自己发出的数据做 NAT 。 
15. 再次进行路由决定。 
16. 进入 filter 表的 OUTPUT 链，可以对本地出去的数据包进行过滤。 
17. 进入 mangle 表的 POSTROUTING 链，同上一种情况的第9步。注意，这里不光对经过防火墙的数据包进行处理，还对防火墙自己产生的数据包进行处理。 
18. 进入 nat 表的 POSTROUTING 链，同上一种情况的第10步。 
19. 进入出去的网络接口。完毕

细心的同学可能发现每个表中不是包含所有链的，也确实如此，如下图所示是各个表中的规则链示意图：
![这里写图片描述](https://img-blog.csdn.net/20180717100741675?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JMVUU1OTQ1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### iptables语法

iptables(选项)（参数）

##### 选项
-t<表>：指定要操纵的表；
-A：向规则链中添加条目；
-D：从规则链中删除条目；
-i：向规则链中插入条目；
-R：替换规则链中的条目；
-L：显示规则链中已有的条目；
-F：清楚规则链中已有的条目；
-Z：清空规则链中的数据包计算器和字节计数器；
-N：创建新的用户自定义规则链；
-P：定义规则链中的默认目标；
-h：显示帮助信息；
-p：指定要匹配的数据包协议类型；
-s：指定要匹配的数据包源ip地址；
-j<目标>：指定要跳转的目标；
-i<网络接口>：指定数据包进入本机的网络接口；
-o<网络接口>：指定数据包要离开本机所使用的网络接口。

iptables常用示例
```
iptables -t 表名 <-A/I/D/R> 规则链名 [规则号] <-i/o 网卡名> -p 协议名 <-s 源IP/源子网> --sport 源端口 <-d 目标IP/目标子网> --dport 目标端口 -j 动作
```

### iptables配置实例
##### 查看防火墙信息
```
iptables -n -v -L
```

#### 清除原有防火墙规则
清除预设表filter中的所有规则链的规则
```
iptables -F 
```
清除预设表filter中使用者自定链中的规则
```
iptables -X
```

##### 添加指定端口
```
iptables -A INPUT -s 127.0.0.1 -d 127.0.0.1 -j ACCEPT               #允许本地回环接口(即运行本机访问本机)
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT    #允许已建立的或相关连的通行
iptables -A OUTPUT -j ACCEPT         #允许所有本机向外的访问
iptables -A INPUT -p tcp --dport 22 -j ACCEPT    #允许访问22端口
iptables -A INPUT -p tcp --dport 80 -j ACCEPT    #允许访问80端口
iptables -A INPUT -p tcp --dport 21 -j ACCEPT    #允许ftp服务的21端口
iptables -A INPUT -p tcp --dport 20 -j ACCEPT    #允许FTP服务的20端口
iptables -A INPUT -j reject       #禁止其他未允许的规则访问
iptables -A FORWARD -j REJECT     #禁止其他未允许的规则访问
```
注意规则是有顺序的，规则的顺序需要由紧到松

##### 屏蔽IP
```
iptables -I INPUT -s 123.45.6.7 -j DROP       #屏蔽单个IP的命令
iptables -I INPUT -s 123.0.0.0/8 -j DROP      #封整个段即从123.0.0.1到123.255.255.254的命令
iptables -I INPUT -s 124.45.0.0/16 -j DROP    #封IP段即从123.45.0.1到123.45.255.254的命令
iptables -I INPUT -s 123.45.6.0/24 -j DROP    #封IP段即从123.45.6.1到123.45.6.254的命令是IP注意
```

##### 删除已经添加的规则
将所有iptables以序号标记显示，执行：
```
iptables -L -n --line-numbers
```
比如要删除INPUT里序号为8的规则，执行：
```
iptables -D INPUT 8
```

##### 保存防火墙设置
```
/etc/init.d/iptables save
```

##### 重新启动防火墙
```
/etc/init.d/iptables restart
```

