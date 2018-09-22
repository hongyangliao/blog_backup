#### 第一种方式
  将项目打包成war，然后放入tomcat根目录/webapps下，启动tomcat，tomcat自动将war解压，然后删除war包,或者之间放入项目编译后的文件

#### 第二种方式
  项目放在任意目录，如/apps/hello-world
  在tomcat根目录/conf中的server.xml的<host/>节点中加入
  ```
  <Context path="/hello-world" docBase="/apps/hello-world"/>
  ```

#### 第三种方式(推荐)
  项目放在任意目录，如/apps/hello-world
  在tomcat根目录/conf中新建Catalina/localhost目录,在localhost目录中创建一个xml文件，文件名称任意(最好使用下项目名),在文件中加入
  ```
    <Context path="/hello-world" docBase="/apps/hello-world"/>
  ```
#### 注意
第三种方法的优点，可以定义别名。服务器端运行的项目名称为path，外部访问的URL则使用XML的文件名
