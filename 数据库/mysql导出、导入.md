#### mysql导出数据库、表
1.导出数据和表结构
```
# mysqldump -u用户名 -p密码 数据库名 > 文件名.sql
```

2.导出单表数据核单表
```
# mysqldump -u用户名 -p密码 数据库名 表名 > 文件名.sql
```

3.只导出表结构
```
mysqldump -u用户名 -p密码 -d 数据库名 > 文件名.sql
```

#### mysql导出sql文件
1.首先创建数据库
```
create database 数据库名
```

2.导入数据库

方法一：

（1）选择数据库
```
use 数据库名
```
（2）导入数据
```
source sql文件路径
```

方法二：
```
mysql -u用户名 -p密码 数据库名 < sql文件路径
```
