> 注意事项：基于centos6

#### 修改密码
> passwd

passwd 作为普通用户和超级权限用户都可以运行，但作为普通用户只能更改自己的用户密码，但前提是没有被root用户锁定；如果root用户运行passwd ，可以设置或修改任何用户的密码

passwd 命令后面不接任何参数或用户名，则表示修改当前用户的密码

#### 修改主机名
 > hostname

  1.使用hostname修改主机名
  hostname hongyangliao-pc

  2.修改/etc/sysconfig/network配置文件
  修改HOSTNAME=hongyangliao-pc

  3.修改本机域名解析文件/etc/hosts
  将原来的主机名换为hongyangliao-pc

#### 修改SSH端口
  1.修改SSH端口配置文件/etc/ssh/sshd_config,将Port的值换为想要的端口号

  2.重启SSH服务，service sshd restart

####  iptabels配置
  1.修改/etc/sysconfig/iptbles 文件
  ```
  *filter
  # Allow loopback (lo0) traffic and drop all traffic to 127/8 that doesn't use the lo0 interface
  -A INPUT -i lo -j ACCEPT
  -A INPUT -i ! lo -d 127.0.0.0/8 -j REJECT
  # Accept established inbound connections
  -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
  # Allow all outbound traffic
  -A OUTPUT -j ACCEPT
  # Allow HTTP and HTTPS connections
  -A INPUT -p tcp --dport 80 -j ACCEPT
  -A INPUT -p tcp --dport 443 -j ACCEPT
  # Allow SSH/SFTP
  # Change the value 22 if you are using a non-standard port
  -A INPUT -p tcp -m state --state NEW --dport 22 -j ACCEPT
  # Allow FTP
  # Purely optional, but required for WordPress to install its own plugins or update itself.
  -A INPUT -p tcp -m state --state NEW --dport 21 -j ACCEPT
  # Allow PING
  # Again, optional. Some disallow this altogether.
  -A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
  # Reject ALL other inbound
  -A INPUT -j REJECT
  -A FORWARD -j REJECT
  COMMIT
  ```
  2.保存
  ```
  service iptables save
  ```

  3.重启
  ```
  service iptables restart
  ```

  4.添加到自启动
  ```
  chkconfig iptables on
  ```

#### ssh scp 免密码登录
  1.本机创建公钥、密钥
  ssh-keygen -t rsa
  一路回车到底

  2.把公钥id_rsa.pub复制到远程服务器上~/.ssh目录并命名为authorized_keys

#### 远程复制scp
  1.从本地复制到远程服务器
  ```
  # scp local_file remote_username@remote_ip:remote_folder
  ```

  2.从远程服务器复制到本地
  ```
  # $scp remote_username@remote_ip:remote_file local_folder
  ```

#### 网络设置
  1.修改ip地址
  修改对应网卡的ip地址
  ```
  # vim /etc/sysconfig/network-scripts/ifcfg-eth0
  ```
  修改以下内容
  ```
  DEVICE=eth0 #描述网卡对应的设备别名，例如ifcfg-eth0的文件中它为eth0
  BOOTPROTO=static #设置网卡获得ip地址的方式，可能的选项为static，dhcp或bootp，分别对应静态指定的 ip地址，通过dhcp协议获得的ip地址，通过bootp协议获得的ip地址
  BROADCAST=192.168.0.255 #对应的子网广播地址
  HWADDR=00:07:E9:05:E8:B4 #对应的网卡物理地址
  IPADDR=12.168.1.2 #如果设置网卡获得 ip地址的方式为静态指定，此字段就指定了网卡对应的ip地址
  IPV6INIT=no
  IPV6_AUTOCONF=no
  NETMASK=255.255.255.0 #网卡对应的网络掩码
  NETWORK=192.168.1.0 #网卡对应的网络地址
  ONBOOT=yes #系统启动时是否设置此网络接口，设置为yes时，系统启动时激活此设备
  ```
  2.修改网关
  修改对应网卡的网关
  ```
  # vim /etc/sysconfig/network
  ```
  修改以下内容
  ```
  NETWORKING=yes #表示系统是否使用网络，一般设置为yes。如果设为no，则不能使用网络，而且很多系统服务程序将无法启动
  HOSTNAME=centos #设置本机的主机名，这里设置的主机名要和/etc/hosts中设置的主机名对应
  GATEWAY=192.168.1.1 #设置本机连接的网关的IP地址。例如，网关为10.0.0.2
  ```

  3.修改DNS
  修改对应网卡的DNS
  ```
  # vim /etc/resolv.conf
  ```
  修改以下内容
  ```
  nameserver 8.8.8.8 #google域名服务器
  nameserver 8.8.4.4 #google域名服务器
  ```

  4.重启网卡
  ```
  #  /etc/init.d/network restart
  ```

  5.简单设置(推荐)
  修改对应网卡的网关
  ```
  # vim /etc/sysconfig/network-scripts/ifcfg-eth0
  ```
  修改以下内容
  ```
  BOOTPROTO=static #静态指定ip
  ONBOOT=yes #系统启动时是否设置此网络接口，设置为yes时，系统启动时激活此设备
  IPADDR=192.168.149.10 #ip地址
  NETMASK=255.255.255.0 #子网掩码
  GATEWAY=192.168.149.2 #网关
  DNS1=8.8.8.8 #DNS
  DNS2=4.4.4.4
  IPV6INIT=no
  DEVICE=eth0 #描述网卡对应的设备别名，例如ifcfg-eth0的文件中它为eth0
  ```
  重启网卡
  ```
  #  /etc/init.d/network restart
  ```

#### 安装Apache
  1.停止并卸载系统自带的httpd服务
  ```
  # service httpd stop
  # ps -ef | grep httpd
  # kill -9 pid号（逐个删除）
  # rpm -qa |grep httpd
  # rpm -e httpd软件包
  ```

  2.安装Apache
  ```
  # wget http://mirrors.hust.edu.cn/apache//httpd/httpd-2.2.34.tar.gz
  # tar -zxvf httpd-2.2.34.tar.gz
  ```

  3.编译
  ```
  # ./configure --prefix=/usr/local/apache
  # make && make install
  ```

  4.启动、停止、重启
  启动Apache：/usr/local/apache2/bin/apachectl start
  停止Apache：/usr/local/apache2/bin/apachectl stop
  重启Apache：/usr/local/apache2/bin/apachectl restart

  5.网站放置目录
  网站放在/usr/local/apache/htdocs

#### 安装php
  1.安装依赖文件(不安装的话，自己会安装很多东西)
  ```
  # yum groupinstall "Development tools"
  # yum install libxml2-devel gd-devel libmcrypt-devel libcurl-devel openssl-devel
  ```

  2.安装php
  ```
  # wget http://php.net/get/php-5.5.38.tar.gz/from/this/mirror
  # tar -zxvf php-5.5.38.tar.gz
  ```

  3.编译
  ```
  # ./configure --prefix=/usr/local/php --with-apxs2=/usr/local/apache/bin/apxs --disable-cli --enable-shared --with-libxml-dir --with-gd --with-openssl --enable-mbstring --with-mysqli --with-mysql --enable-opcache --enable-mysqlnd --enable-zip --enable-fpm --enable-fastcgi --with-zlib-dir --with-pdo-mysql --with-jpeg-dir --with-freetype-dir --with-curl --without-pdo-sqlite --without-sqlite3 --with-mcrypt=/usr/local/libmcrypt/
  # make && make install
  ```

  4.注意事项
  如果出现 configure: error: mcrypt.h not found. Please reinstall libmcrypt
  则需安装libmcrypt
  ```
  # wget ftp://mcrypt.hellug.gr/pub/crypto/mcrypt/attic/libmcrypt/libmcrypt-2.5.7.tar.gz
  # tar -zxvf libmcrypt-2.5.7.tar.gz
  # cd libmcrypt-2.5.7
  # ./configure prefix=/usr/local/libmcrypt/
  # make && make install
  ```

  5.配置Apache中的PHP环境
  修改Apache的配置文件httpd.conf
  在LoadModule中添加：
  ```
  LoadModule php5_module modules/libphp5.so
  ```
  在AddType application/x-gzip .gz .tgz下面添加：
  ```
  AddType application/x-httpd-php .php
  AddType application/x-httpd-php-source .phps
  ```

  在DirectoryIndex增加 index.php，以便Apache识别PHP格式的index
  ```
  <IfModule dir_module>  
    DirectoryIndex index.html index.php  
  </IfModule>
  ```

  6.验证PHP环境
  ```
  # vim /usr/local/apache/htdocs/info.php
  ```
  添加如下代码
  ```
  <?php

  phpinfo();

  ?>
  ```
  访问 http://locahost/info.php 可查看很多信息


#### 安转rar
    1.安装依赖库
    ```
    # yum install -y gcc gcc-c++ autoconf wget
    ```

    2.下载源码包
    ```
    # wget http://www.rarlab.com/rar/rarlinux-x64-5.3.0.tar.gz
    ```

    3.解压编译
    ```
    tar -zxvf rarlinux-x64-5.3.0.tar.gz
    cd rar
    make && make install
    cp -f rar_static /usr/local/bin/rar && cp -f rar_static /usr/local/bin/unrar
    cd ..
    rm -rf rar
    ```

    4.使用
    ```
    解压：rar x filename.rar
    压缩：rar a targetName.rar dirName
    ```
