### 简介
什么是Proxy Buffer呢，我理解为缓存区，因为Nginx是作为客户端和服务端通信的桥梁，那么被代理服务器必然会将响应返回给Nginx，那么Nginx是同步的将接受到的响应发送给客户端，还是等接受到被代理服务器的响应到一定程度，一下返回给客户端呢，那么这就需要 Proxy Buffer的配置了，另外提一下，Proxy Buffer的配置不是全局的，而是每个请求都会按照这些指令的配置来设置各自的缓存。Proxy Buffer有7个指令，我们下面来看一看

### Proxy Buffer配置
（1） 配置是否启用Proxy Buffer
```
proxy_buffering on | off;
```
默认为on，开启式Nginx会异步的将被代理服务器的响应数据传递给客户端，关闭时只要Nginx收到被代理服务器的响应就会同步给地给客户端，开启和关闭Proxy Buffer还可以使用响应头的“X-Accel-Buffering”设置“yes“” 和 “no”

（2）配置接受一次响应的buffer个数和你每个buffer的大小
```
proxy_buffer number size;
```
number代表数量，size代表大小，一般size设置为内存页的大小4k或者8k
如
```
proxy_buffer 4 4k;
```
那么一次响应的Proxy Buffer总大小为4 * 4k = 16k

（3） 配置从被代理服务器获取的第一份响应数据的大小
```
proxy_buffer_size size;
```
一般第一份响应数据中都包含了http响应头，Nginx通过它来获取响应数据和被代理数据的一些必要信息，一般保持与proxy_buffer指令中的size变量相同即可

（4）配置处于busy状态的Proxy Buffer的总大小
```
 proxy_busy_buffers_size size;
```
 buzy状态的Proxy Buffer是指当一个buffer被填满后，在将所有数据都响应给客户端的过程

（5）配置如果Proxy Buffer的容量不够，响应数据的存放地方
```
proxy_temp_path path [level1 [level2 [level3]]];
```
path设置磁盘上存放临时文件的路径
level1的意思是在path路径下的第几级**hash**目录存放临时文件

（6）配置所有临时文件的总体积大小
```
proxy_max_temp_file_size size;
```
存放在磁盘上的临时文件不应该超过该大小，默认值为1024MB

（7）配置同时写入临时文件的数据量的总大小
```
proxy_temp_file_write_size size;
```
size为设置的数据量总大小上限值，根据平台的不同，可以为8KB或16KB，一般与平台的内存页大小相同
