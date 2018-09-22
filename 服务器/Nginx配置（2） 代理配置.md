### 前言
Nginx的基本配置讲完了，那么感觉平时用的都没讲到啊，代理，负载均衡，缓存，重定向以及常见的https配置，接下来我们就来看一看代理部分
### 代理配置
正向代理和反向代理的区别:正向代理隐藏真实客户端，反向代理隐藏真实服务端，上张网上的图，生动形象
![这里写图片描述](https://img-blog.csdn.net/20180822013148608?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JMVUU1OTQ1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
#### 正向代理
正向代理一般用的不多
```
server
{
	resovler 8.8.8.8 vaild=30s;
	listen 8080;
	location /
	{
		proxy_pass http://$http_host$request_uti;
	}

}
```
resovler是将域名映射为ip，vaild是域名解析超时时间，Nginx将代替你去请求服务器，之后将结果返回给你，我们利用正向代理搭vpn

#### 反向代理
反向代理非常重要，是最常用的Nginx功能之一，而且指令非常多，下面就来看一看吧
```
···
upstream my_server {
	server 192.168.2.1:8080;
	server 192.168.2.2:8080;
	server 192.168.2.3:8080;
}

server {
···
	listen 80;
	server_name www.hongyangliao.com;
	location / {
		proxy_pass http://my_server;
		proxy_hide_header Content-Type;
		proxy_pass_header Server;
		proxy_pass_request_body on;
		proxy_pass_request_headers on;
		proxy_set_header Content-Type application/json;
		proxy_set_body testbody;
		proxy_bind 192.168.1.2;
		proxy_connect_time 60s;
		proxy_read_time 60s;
		proxy_send_time 60s;
		proxy_http_version 1.1;
		proxy_method GET;
		proxy_ignore_client_abort off;
		proxy_ignore_headers Set-Cookie;
	}
	
	#location / {
		#proxy_pass http://192.168.2.1:8080;
		#proxy_pass http://localhost:8080;
		#proxy_pass http://unix:/tmp/server.socket;
	#}
}
```
（1）配置被代理的服务器地址
```
proxy_pass URL;
```
URL可以是主机名称、IP地址加端口号的形式，如果有多个服务器，还可以使用upstream 指令配置后端服务器组，如果upstream 中没有指明传输协议如http://**（这里有问题，经过实验，在nginx/1.12.2中如果在upstream中指定http://会报错，如下图）**，则需要使用 proxy_pass http://my_server;
![这里写图片描述](https://img-blog.csdn.net/2018082515144668?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JMVUU1OTQ1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](https://img-blog.csdn.net/20180825151459518?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JMVUU1OTQ1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
```
upstream my_server {
	server 192.168.2.1:8080;
	server 192.168.2.2:8080;
	server 192.168.2.3:8080;
}

server {
···
	listen 80;
	server_name www.hongyangliao.com;
	location / {
		proxy_pass http://my_server;
	}
```
对于proxy_pass 指令中的URL中是否包含URI,Nginx处理是不同的
如果URL中不包含URI
```
server {
···
	listen 80;
	server_name www.hongyangliao.com;
	location /admin/ {
		proxy_pass http://192.168.2.1:8080;
	}
}
```
使用http://www.hongyangliao.com/admin访问，则会转向http://192.168.2.1:8080/admin
如果URL包含URI
```
server {
···
	listen 80;
	server_name www.hongyangliao.com;
	location /admin/ {
		proxy_pass http://192.168.2.1:8080/user/;
	}
}
```
使用http://www.hongyangliao.com/admin访问，则会转向http://192.168.2.1:8080/user/
**注意：因此如果不想改变原地址的URI，就不要在proxy_pass 指令中的URL中包含URI**
对于proxy_pass中末尾加不加“/”,不加杠代表URL中不包含URI，加杠代表包含URI，根据上面的描述，你懂得

（2）配置Nginx服务器在发送http响应时需要隐藏的头信息
```
proxy_hide_header field;
```
field为需要隐藏的头信息，可以在http块，server块，location块配置

（3）配置一些Nginx http响应时发送之前被隐藏的头信息
```
proxy_pass_header field;
```
field为需要发送的头信息，一般Nginx发送响应时是不包含"Date"、“Server”等头信息的

（4）配置是否将客户端的请求体发送给被代理服务器
```
proxy_pass_request_body on | off;
```
默认为on，可以在http块，server块，location块配置

（5）配置是否将客户端的请求头发送给被代理服务器
```
proxy_pass_request_headers on | off
```
默认为on,可以在http块，server块，location块配置

（6）更改收到的客户端请求的请求头信息，将新的请求头发送给被代理服务器
```
proxy_set_header field value;
```
其中field代表要更改的请求头的key，value代表请求头的value，value可以是变量、文本或者他们的组合

（7）更改收到的客户端请求的请求体信息，将新的请求体发送给被代理服务器
```
proxy_set_body value;
```
value代表请求体的内容，value可以是变量、文本或者他们的组合

（8）将代理主机的连接连接绑定到指定的IP地址
```
proxy_bind address
```
address为IP地址

（9）配置Nginx与被代理服务器建立连接的超时时间
```
proxy_connect_time time;
```
其中time为设置的超时时间，默认为60s

（10）配置Nginx向后端服务器发出read请求后等待响应的超时时间
```
proxy_read_time time;
```
其中time为设置的超时时间，默认为60s

（11）配置Nginx向后端服务器发出write请求后等待响应的超时时间
```
proxy_send_time time;
```
其中time为设置的超时时间，默认为60s

（12）配置Nginx提供代理服务的http协议版本
```
proxy_http_version 1.0 | 1.1;
```
默认1.0，最好改成1.1因为http 1.1默认使用长连接

（13）配置Nginx请求被代理服务器的请求方法
```
proxy_method method;
```
设置这个指令将忽略客户端的请求方法

（14） 配置Nginx是否在客户端中断网络请求时中断对被代理服务器的网络请求
```
proxy_ignore_client_abort on | off;
```
默认为off，即在客户端中断网络请求时中断对被代理服务器的网络请求

（15）配置Nginx忽略收到的被代理服务器的响应头
```
proxy_ignore_headers field ...;
```
field为要设置的头域

（16）修改从被代理服务器响应的响应头中“Location“和“Refresh”字段
```
proxy_redirect redirect replacement | default | off
```
对于**redirect replacement**配置,如果被代理服务器返回的响应头中Location为
```
Location:http://localhost:8080/admin/hello/
```
那么使用
```
proxy_redirect http://localhost:8080/admin/ http://www.hongyangliao.com/
```
这个指令，Nginx服务器会将响应头改为
```
Location:http://www.hongyangliao.com/hello/
```
其中redirect支持变量和正则表达式，而replacement支持文本和变量

对于**default** 配置，代表location块的URI变量作为replacement，proxy_pass中的URI作为redirect ，举个例子,下面的配置是一样的
```
location /server/ {
	proxy_pass http://192.168.1.2/;
	proxy_redirect default;
}

location /server/ {
	proxy_pass http://192.168.1.2/;
	proxy_redirect http://192.168.1.2/ /server/;
}
```
对于**off**配置会使当前作用域下的proxy_redirect配置失效

（17） 配置是否开启拦截被代理服务器返回错误码的情况
```
proxy_intercept_errors on | off
```
当设置为on时，被代理服务器返回状态码大于400时，会被Nginx拦截，返回Nginx自己的错误页面，off则不拦截

（18） 配置存放HTTP请求头的hash表的大小
```
proxy_headers_hash_max_size size;
```
默认为512个字符

（19） 配置Nginx申请存放HTTP请求的hash表容量大小的单位
```
proxy_headers_hash_bucket_size size;
```
默认为64个字符

（20）配置在使用upstream中什么情况下将请求交给下一个被代理服务器
```
proxy_next_upstream status;
```
status有一下几种
**error**:在建立连接，向被代理服务器发送请求头或读取响应头时发生连接错误
**timeout**:在建立连接，向被代理服务器发送请求头或读取响应头时发生连接超时
**invaild_header**:被代理服务器响应的响应头为空或无效
**http_404 | http_500 | http_502 | http_503 | http_504**:被代理服务器返回对应的状态码时

（21）配置是否使用基于SSL协议的连接被代理服务器
```
proxy_ssl_session_reuse on | off
```
默认为on
