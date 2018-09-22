### 简介
Nginx是一个Web服务器，同时也能够用作反向代理，负载均衡器，HTTP缓存，邮件服务器等
#### 特点
(1) Nginx使用异步事件驱动的方法来处理请求。Nginx的模块化事件驱动架构可以在高负载下提供更可预测的性能
(2) Nginx是一款面向性能设计的HTTP服务器，相较于Apache、lighttpd具有占有内存少，稳定性高等优势。与旧版本（<=2.2）的Apache不同，Nginx不采用每客户机一线程的设计模型，而是充分使用异步逻辑从而削减了上下文调度开销，所以并发服务能力更强。整体采用模块化设计，有丰富的模块库和第三方模块库，配置灵活。 在Linux系統下，Nginx使用epoll事件模型，因此，Nginx在Linux系統下效率相当高
[特点来自维基百科](https://zh.wikipedia.org/wiki/Nginx)

### 基本配置
#### 文件结构
默认的nginx.conf文件一般放在Nginx安装目录下的conf文件夹下，nginx.conf主要分为以下个部分，
全局块、events块、http块、server块、location块 ，如下图所示为省略具体配置的文件结构
```
# 全局块
··· 

events { # events块
	···
}


http { # http块
	# http全局块
    ···
	
    server { # server块
	    ···
	    
        location [pattern] { # location块
           ···
        }
# 全局块
···
}
```
下面简单介绍下各个部分的作用：
（1）主机块
影响Nginx服务器整体运行的配置指定
（2）events块
影响Nginx服务器与用户的网络连接
（3）http块
缓存、代理、日志和第三方模块等的配置
（4）server块
虚拟主机的配置，这个待会详细讲
（5）location块
数据缓存，请求重定向，负载均衡，代理和应答控制等

#### 常用配置
如下图所示，贴出默认的nginx.conf文件
```
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

events {
	#use epoll;
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    sendfile_max_chunk 10m;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
		
		#error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        
        location / {
            root   html;
            index  index.html index.htm;
            # allow 192.168.3.3;
			# allow 193.168.4.4/24;
			# deny all;
        }
		
	     location / {
		     root /data;
	         # 显示索引
	         autoindex on;
	         # 显示大小
	         autoindex_exact_size on;
	         # 显示时间
	         autoindex_localtime on;
	        }
	        
	# include /opt/work/nginx/conf/my.conf
}
```
##### 全局块配置
（1）配置服务器用户（组）
user [user] [group]
只有被设置的用户或者用户组成员才有权限运行nginx，如果希望所用用户都能运行需要设置为
```
user nobody nobody;
```

或者使用“#“注释掉
（2）配置允许生成的work process数
```
worker_processes  2;
```
即最多生成多少工作进程进行工作还可以设置为auto,让nginx自动检测
```
worker_processes  auto;
```

（3）配置错误日志的输出级别和存放的位置,如
```
error_log  /var/log/nginx/error.log  info;
```

（4）配置Nginx主进程id存放的位置
```
pid /run/logs/nginx.pid;
```

（5）引入配置文件
```
include /opt/work/nginx/conf/my.conf
```
配置文件可以使用相对路径，需要user对该配置文件具有读权限

##### events块配置
（1）配置事件驱动模型
```
use epoll;
```
处理网络消息，可选择的内容有select 、poll、 kqueue 、epoll 、rtsig,一般 linux配置epoll

（2）配置一个工作进程的最大连接数
```
worker_connections  1024;
```

#### http块全局配置
（1）配置能够识别的前端请求类型
```
include mimi.types;
```
mimi.types是与nginx.conf相同目录下的配置文件,们来看一看有哪些
```
types {
    text/html                             html htm shtml;
    text/css                              css;
    text/xml                              xml;
    image/gif                             gif;
    image/jpeg                            jpeg jpg;
    application/javascript                js;
    application/atom+xml                  atom;
    application/rss+xml                   rss;
	...省略一万字
}
```

（2）配置用于处理前端请求的MIME类型
```
default_type  application/octet-stream;
```
如果不加此指令，默认为text/html，同时还可以在http块，server块或者location块配置

（3）配置用于记录日志的格式后面讲有什么用
```
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
```
格式化字符串用单引号括起来，还有一些$开头的是Nginx的内置变量

（4） 配置Nginx提供服务过程中前端请求的日志
```
access_log  /var/log/nginx/access.log  main;
```
其中main就是刚刚配置的 log_format

（5）配置开启或关闭使用sendfile()传输文件
```
sendfile        on;
```

（6）配置Nginx每个worker process使用sendfile()函数传输文件的大小
```
sendfile_max_chunk 10m;
```
默认值为0,如果设为0则表示无限制

（7） 配置http长连接超时时间，单位为s
```
keepalive_timeout  65;
```
同时还可以在server和location中配置
```
keepalive_timeout  120s 100s;
```
表示服务端保持120s，发送到用户端的Http响应头中Keep-Alive字段为100s

（8）配置是否开启gzip压缩
```
gzip  on;
```

##### server块配置
（1）监听端口
```
listen       80;
```
下面给出一些例子
```
listen 192.168.2.2:8080; # 监听某个ip访问某个端口的链接
listen 192.168.2.2; # 监听某个ip访问所有端口的链接
listen 8080; # 监听所有ip访问某个端口的链接
```

（2）基于名称的虚拟主机配置
```
server_name  hongyangliao.com;
```
如将hongyangliao.com这个域名解析到这个服务器，那么配置完server_name后通过域名hongyangliao.com访问的就会被这个虚拟主机接受，同时支持配置多个域名，并且支持使用通配符和正则表达式，如
```
server_name hongyangliao.com test.hongyangliao.*
server_name ~^www\.hongyangliao\d+\.com$

```

（3）配置网站的错误页面
```
error_page   500 502 503 504  /50x.html;
location = /50x.html {
	root   html;
}
```
没啥好说的,也可以在location里面配置

##### location块配置
（1）配置location
在Nginx官方文档中location的语法为
```
location [ = | ~ | ~* | ^~ ] uri [...]
```
其中
“=”表示 严格匹配
“~"表示匹配uri包含正则表达式，区分大小写
“~*”表示匹配uri包含正则表达式，不区分大小写
"~^"表示找到匹配uri的程度最高的location后立即使用次location

（2）配置请求的根路径
```
root /opt/work/html
```
Nginx接到请求后，在指定目录下寻找资源，root就是配置根目录的

（3）设置网站的默认首页
```
index  index.html index.htm;
```
没啥好说的，从根路径中找

（4）配置访问控制
```
allow 192.168.3.3;
allow 193.168.4.4/24;
deny all;
```
（5）配置静态文件服务器
如第二个location所示，只需将root设为一路径下，接着设置以下三个参数即可
```
# 显示索引
autoindex on;
# 显示大小
autoindex_exact_size on;
# 显示时间 
autoindex_localtime on;	        
```
使用浏览器访问，效果如图所示
![这里写图片描述](https://img-blog.csdn.net/20180822012114521?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JMVUU1OTQ1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
