#### 关闭安全子系统
```
sed -i "s/SELINUX=enforcing/SELINUX=disabled/" /etc/selinux/config
setenforce 0
```

#### 安装wget
```
yum -y install wget
```

#### 更改默认镜像为阿里镜像
```
cd /etc/yum.repos.d && mv CentOS-Base.repo CentOS-Base.repo.bak
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
yum makecache
yum clean all
```

#### 安装zip
```
yum install -y unzip zip
```

#### 安装vim
```
yum install -y vim
```

#### 安装scp
```
yum -y install openssh-clients
```

#### 安装相关依赖
```
yum -y install gcc-c++ tcl gettext-devel pcre-devel openssl openssl-devel bison flexcurl-devel expat-devel zlib-devel autoconf automake libtool python ncurses-devellibjpeg-devel e2fsprogs-devel sqlite-devel libcurl-devel speex-devel ldns-devel libeditdevelreadline-devel ncurses-devel pam-develnumactl
```

#### 安装screen会话
```
yum -y install screen
```

#### 安装ntpdate时间同步工具
```
yum -y install ntpdate
echo '0 1 * * * ntpdate cn.pool.ntp.org;hwclock -w' >> /var/spool/cron/root
```

#### 安装java
```
mkdir -p /opt/java
cd ~ && wget http://oss.hongyangliao.com/jdk-8u151-linux-x64.tar.gz
mv jdk-8u151-linux-x64.tar.gz /opt/java/
cd /opt/java
tar -xzvf jdk-8u151-linux-x64.tar.gz
rm -rf jdk-8u151-linux-x64.tar.gz
echo 'export JAVA_HOME=/opt/java/jdk1.8.0_151' >> /etc/profile
echo 'export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib/rt.jar' >> /etc/profile
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> /etc/profile
cd /usr/bin
ln -s -f /opt/java/jdk1.8.0_151/jre/bin/java
ln -s -f /opt/java/jdk1.8.0_151/bin/javac
```

#### 安装Tomcat
```
cd ~ && wget http://oss.hongyangliao.com/apache-tomcat-8.5.24.tar.gz
mv apache-tomcat-8.5.24.tar.gz /usr/local/
cd /usr/local
tar -xzvf apache-tomcat-8.5.24.tar.gz
mv apache-tomcat-8.5.24 tomcat8
mv apache-tomcat-8.5.24.tar.gz /usr/local/src
echo 'export CATALINA_HOME=/usr/local/tomcat8' >> /etc/profile
source /etc/profile
```

#### 设置Tomcat自启
```
cd /etc/init.d
touch tomcat8 && chmod 755 tomcat8 && vim tomcat8
```
##### 添加如下命令,注意JAVA_HOME,CATALANA_HOME

```
#!/bin/bash    
#    
# tomcat startup script for the Tomcat server    
#    
# chkconfig: 345 80 20    
# description: start the tomcat deamon    
#    
# Source function library    
. /etc/rc.d/init.d/functions

prog=tomcat8
JAVA_HOME=/opt/java/jdk1.8.0_151
export JAVA_HOME    
CATALANA_HOME=/usr/local/tomcat8
export CATALINA_HOME    

case "$1" in
start)
    echo "Starting Tomcat..."    
    $CATALANA_HOME/bin/startup.sh
    ;;

stop)
    echo "Stopping Tomcat..."    
    $CATALANA_HOME/bin/shutdown.sh
    ;;

restart)
    echo "Stopping Tomcat..."    
    $CATALANA_HOME/bin/shutdown.sh
    sleep 2
    echo    
    echo "Starting Tomcat..."    
    $CATALANA_HOME/bin/startup.sh
    ;;

*)
    echo "Usage: $prog {start|stop|restart}"    
    ;;
esac
exit 0
```

##### 执行以下命令
```
chkconfig --add tomcat8
chkconfig --level 345 tomcat8
```

#### 安装nginx
##### 下载并安装nginx
```
cd && wget http://oss.hongyangliao.com/nginx-1.8.1.tar.gz
mv nginx-1.8.1.tar.gz /usr/local/src
cd /usr/local/src && tar -xzvf nginx-1.8.1.tar.gz
cd nginx-1.8.1 && ./configure --prefix=/usr/local/nginx
make && make install
```

##### 设置nginx自启
```
cd /etc/init.d
touch nginx
vim nginx
```

添加如下内容

```
#!/bin/bash  
# nginx Startup script for the Nginx HTTP Server  
#  
# chkconfig: - 85 15  
# description: Nginx is a high-performance web and proxy server.  
# It has a lot of features, but it's not for everyone.  
# processname: nginx  
# pidfile: /usr/local/nginx/logs/nginx.pid  
# config: /usr/local/nginx/conf/nginx.conf  
nginxd=/usr/local/nginx/sbin/nginx  
nginx_config=/usr/local/nginx/conf/nginx.conf  
nginx_pid=/usr/local/nginx/nginx.pid  

RETVAL=0  
prog="nginx"

# Source function library.  
. /etc/rc.d/init.d/functions  

# Source networking configuration.  
. /etc/sysconfig/network  

# Check that networking is up.  
[ ${NETWORKING} = "no" ] && exit 0  

[ -x $nginxd ] || exit 0  


# Start nginx daemons functions.  
start() {  

if [ -e $nginx_pid ];then
   echo "nginx already running...."
   exit 1  
fi  

   echo -n $"Starting $prog: "
   daemon $nginxd -c ${nginx_config}  
   RETVAL=$?  
   echo  
   [ $RETVAL = 0 ] && touch /var/lock/subsys/nginx  
   return $RETVAL  

}  


# Stop nginx daemons functions.  
stop() {  
        echo -n $"Stopping $prog: "
        killproc $nginxd  
        RETVAL=$?  
        echo  
        [ $RETVAL = 0 ] && rm -f /var/lock/subsys/nginx /var/run/nginx.pid  
}  


# reload nginx service functions.  
reload() {  

    echo -n $"Reloading $prog: "
 $nginxd -s reload  
    #if your nginx version is below 0.8, please use this command: "kill -HUP `cat ${nginx_pid}`"
    RETVAL=$?  
    echo  

}  

# See how we were called.  
case "$1" in
start)  
        start  
        ;;  

stop)  
        stop  
        ;;  

reload)  
        reload  
        ;;  

restart)  
        stop  
        start  
        ;;  

status)  
        status $prog  
        RETVAL=$?  
        ;;  
*)  
        echo $"Usage: $prog {start|stop|restart|reload|status|help}"
        exit 1  
esac  

exit $RETVAL
```
保存,接着设置自启
```
chmod +x /etc/init.d/nginx
chkconfig --add nginx
chkconfig --level 345 nginx on
service nginx start
```

#### 安装Redis
```
cd && wget http://oss.hongyangliao.com/redis-4.0.6.tar.gz
tar -zxvf redis-4.0.6.tar.gz
cd redis-4.0.6 && make PREFIX=/usr/local/redis install
cd /usr/local/redis && mkdir conf
cp ~/redis-4.0.6/redis.conf /usr/local/redis/conf

```
##### 设置后端启动Redis
```
vim /usr/local/redis/conf/redis.conf
```
将 daemonize yes 以后端模式启动
```
################################# GENERAL #####################################

# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
daemonize yes

```
##### 设置redis服务自启
```
cd /etc/init.d && touch redis && chmod +x redis
vim redis
```
在redis文件中添加如下内容,注意redis_pid的值
```
cd /etc/init.d
touch redis
vim redis
```
```
#!/bin/bash
#
# chkconfig: - 85 15
# script_name:redisd
# description:redis daemon
# config: /usr/local/redis/conf/redisd.conf
# pidfile: /var/run/redis-server.pid
#
### BEGIN INIT INFO
# Provides: redis
### END INIT INFO

# Source function library.
. /etc/rc.d/init.d/functions

#variables
redis_port='6379'
redis_pid='/var/run/redis_'$redis_port'.pid'
redis_server="/usr/local/redis/bin/redis-server"
redis_conf="/usr/local/redis/conf/redis.conf"
redis_cli="/usr/local/redis/bin/redis-cli"

#function
function _start() {
    if [ ! -e "$redis_pid" ];then
        "$redis_server" "$redis_conf"
        echo "redis service start.......OK"
        return 0
    else
        echo "redis service is running !"
        return 1
    fi
}

function _stop() {

    if [ -e "$redis_pid" ];then
         "$redis_cli"  shutdown
         echo "redis service stop.......OK"
         sleep 1
         return 0
    else
         echo "redis service is not running !"
         return 1

    fi
}

function _status() {

    if [ -e "$redis_pid" ];then
         echo "redis service is running !"
         return 0
    else
         echo "redis service is not running !"
         return 1
    fi

}


#main
case "$1" in

        start)
               _start
                ;;
        stop)
               _stop
                ;;
        status)
               _status
               ;;
        *)
          echo  "Usage: $0 {start|stop|status}"
esac
```
接着
```
chkconfig --add redis
chkconfig --level 345 redis on
service redis start
```
##### 设置给redis客户端设置软链接
```
ln -s /usr/local/redis/bin/redis-cli /usr/local/bin/redis-cli
```

#### 安装mysql
```
wget http://repo.mysql.com/mysql-community-release-el6-5.noarch.rpm
rpm -ivh mysql-community-release-el6-5.noarch.rpm
yum install mysql-server
/etc/init.d/mysqld start
mysql_secure_installation
mysql -uroot -p
use mysql;
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%' IDENTIFIED BY 'admin' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

#### 修改mysql数据库编码为UTF-8，修改/etc/my.cnf或者/etc/mysql/my.cnf文件
```
[client]
default-character-set = utf8
[mysqld]
default-storage-engine = INNODB
character-set-server = utf8
collation-server = utf8_general_ci
```

#### 重启mysql服务，并查看数据库编码
```
service mysqld restart
mysql -uroot -p
show variables like '%char%';
```
编码为以下即可
```
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
```

#### 安装Maven
```
cd ~ && wget http://oss.hongyangliao.com/apache-maven-3.5.2-bin.tar.gz
mv apache-maven-3.5.2-bin.tar.gz /usr/local/
cd /usr/local && tar -xzvf apache-maven-3.5.2-bin.tar.gz
mv apache-maven-3.5.2 maven
echo 'M2_HOME=/usr/local/maven' >> /etc/profile
echo 'export PATH=$M2_HOME/bin:$PATH' >> /etc/profile
source /etc/profile
mvn help:system
cp /usr/local/maven/conf/settings.xml /root/.m2/
mv /usr/local/apache-maven-3.2.5-bin.tar.gz /usr/local/src/
```
#### 安装Git
```
yum -y install git
```

#### 配置公钥
```
ssh-keygen -t rsa
```
