#### 修改/etc/my.cnf或者/etc/mysql/my.cnf文件
```
[client]
default-character-set = utf8
[mysqld]
default-storage-engine = INNODB
character-set-server = utf8
collation-server = utf8_general_ci
```

#### 重启mysql,使用mysql客户端检查编码
```
show variables like '%char%';
```

#### 创建新数据库时使用UTF-8编码
```
create database 'test' default character set utf8 collate utf8_general_ci;
```

#### 创建表,创建字段使用UTF-8编码
```
create table test (
  'id' int(10) unsigned not null auto_increment,
  'name' varchar(50) character set utf8 default '',
  primary key('id')
  ) default charset=utf8;
```
