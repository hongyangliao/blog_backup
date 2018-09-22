### 简介
终于到了负载均衡这一步了，这是Nginx反向代理功能中非常重要的一部分。那么什么是负载均衡呢？维基百科中的解释，负载平衡（Load balancing）是一种计算机技术，用来在多个计算机（计算机集群）、网络连接、CPU、磁碟驱动器或其他资源中分配负载，以达到最佳化资源使用、最大化吞吐率、最小化响应时间、同时避免过载的目的。 使用带有负载平衡的多个伺服器组件，取代单一的组件，可以通过冗余提高可靠性。负载平衡服务通常是由专用软件和硬件来完成。那么Nginx其实就是软件实现形式。Nginx实现了静态的基于优先级的加权轮询算法，主要是基于proxy_pass和upstream两个指令实现，那么我们就来看看常见的一些配置。

### 负载均衡配置
Nginx的负载均衡可以分为两大类：
（1）内置策略
轮询：就是按请求顺序依次分配到不同后端节点
加权轮询：按照权重的大小来分配请求
ip hash：将请求的ip作hash操作，根据hash结果将请求分配到不同的节点，如果分布式系统不想自己实现分布式session，那么这个方法较为简单，因为一个ip的所有的请求最终都会落到同一个后端节点
（2）扩展策略
主要是fair和url hash
fair：按后端服务器的响应时间来分配请求，响应时间短的优先分配
url hash：跟ip hash类似，可以提高缓存效率同时也能解决分布式session的问题，缺点跟ip hash一样，不能自动排除节点

#### 轮询
```
···
upstream my_load_balancing_server {
	server 192.168.0.2;
	server 192.168.0.3;
	server 192.168.0.4;
}

server {
	listen 80;
	server_name www.hongyangliao.com;
	location / {
		proxy_pass http://my_load_balancing_server;
		···
	}
	···
}
```
如配置所示，实际上upstream中的server默认权重weight=1，这样一般他们会按照轮询策略依次接受请求

#### 加权轮询
```
···
upstream my_load_balancing_server {
	server 192.168.0.2 weight=6;
	server 192.168.0.3 weight=3; 
	server 192.168.0.4;
}

server {
	listen 80;
	server_name www.hongyangliao.com;
	location / {
		proxy_pass http://my_load_balancing_server;
		···
	}
	···
}
```
如配置所示只需要在upstream中的server中配置wight权重即可，http://192.168.0.2权重最高，优先接受和处理客户端请求，http://192.168.0.4权重最低，默认为1

#### ip hash
```
···
upstream my_load_balancing_server {
	server 192.168.0.2;
	server 192.168.0.3; 
	server 192.168.0.4;
	ip hash;
}

server {
	listen 80;
	server_name www.hongyangliao.com;
	location / {
		proxy_pass http://my_load_balancing_server;
		···
	}
	···
}
```
可以看到ip hash非常简单，只需要在upstream中配置ip hash指令即可

#### fair
```
···
upstream my_load_balancing_server {
	server 192.168.0.2;
	server 192.168.0.3; 
	server 192.168.0.4;
	fair;
}

server {
	listen 80;
	server_name www.hongyangliao.com;
	location / {
		proxy_pass http://my_load_balancing_server;
		···
	}
	···
}
```
按后端服务器的响应时间来分配请求，响应时间短的优先分配，注意fai是第三方的，因此需要安装模块，否则会报错
```
nginx: [emerg] unknown directive "fair" in /opt/work/nginx/./conf/nginx.conf:37
```
需要在安装nginx添加第三方模块upstream-fair或者后期安装，这里就不说了，上网查询一下即可

#### url hash
```
···
upstream my_load_balancing_server {
	server 192.168.0.2;
	server 192.168.0.3; 
	server 192.168.0.4;
	hash $request_uri; 
	hash_method crc32; 
}

server {
	listen 80;
	server_name www.hongyangliao.com;
	location / {
		proxy_pass http://my_load_balancing_server;
		···
	}
	···
}
```
url hash也是第三方模块，需要先安装才能使用，其中server语句中不能写入weight等其他的参数，hash_method是使用的hash算法


