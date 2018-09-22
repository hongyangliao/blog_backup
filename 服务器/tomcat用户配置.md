####  进入Tomcat目录下conf/tomcat-users.xml
  1.Tomcat6配置管理员信息
  在配置文件<tomcat-users>节点下添加
  ```
  <role rolename="admin"/>
  <role rolename="manager"/>
  <user username="admin" password="admin" roles="admin,manager"/>
  ```

  2.Tomcat7配置管理员信息
  在配置文件<tomcat-users>节点下添加
  ```
  <role rolename="admin-gui"/>
  <role rolename="manager-gui"/>
  <user username="admin" password="admin" roles=" admin-gui,manager-gui "/>
  ```
