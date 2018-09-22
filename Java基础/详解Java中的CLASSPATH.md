### 背景
某天做maven项目，项目完成，需要打成jar包形式，直接运行，报找不到主类的错误，这就尴尬了，这么简单的问题竟然让我忽略了，以前都是做web项目，放在tomcat里管他呢，谁知道他是如何找到class文件运行的呢？网上查了一番，需要配置环境变量CLASSPATH，那么问题又来了，什么是CLASSPATH呢？以前倒是配置过，但是只知其然不知其所以然。那么接下来就来讲讲CLASSPATH，希望可以解答上面的问题。

### CLASSPATH
**The classpath tells Java where to look in the filesystem for files defining these classes.**
从[维基百科](https://en.wikipedia.org/wiki/Classpath_%28Java%29)里找到这一段描述，翻译过来就是**CLASSPATH是为了告诉Java这些需要导入的class文件的位置在哪里**。
Java虚拟机会按以下顺序搜索并加载类：
1. bootstrap类：Java平台基础的类（包括Java类库的公共类，以及此库可用的私有类）。
2. 扩展类：JRE或JDK的扩展目录中的包，jre / lib / ext /用户定义的包和库
3. 默认情况下，只能访问JDK 标准API和扩展包的包，而无需设置查找位置。**如果需要使用自己的class文件，则必须在命令行中或者环境变量中设置用户定义的包和库的路径。**

**因此，CLASSPATH的作用是java用来寻找class文件的**，下面我们来看看如何设置classpath

#### 命令行设置CLASSPTH
如下所示，为测试项目的目录结构，其中ClassPathTest为主类
```
classpath_test
	|--com
		|--liao
			|--path
				|--HelloPath.class
			|--ClassPathTest.class
```
1. windows
在windows上将项目放在D盘下，则需要使用如下命令运行
```
java -classpath D:\classpath_test com.liao.ClassPathTest
```
2. linux
在linux中将项目放在/opt/work目录下，则需要使用如下命令运行
```
java -cp /opt/work/classpath_test com.liao.ClassPathTest
```

#### 环境变量设置CLASSPATH
如果不想通过命令行设置class路径，设置环境变量也是可以的
1. windows
windows下设置CLASSPATH想必大家都知道
右击“计算机” -》 属性 -》高级系统设置 -》环境变量
设置用户变量或者系统变量都行，设置变量为CLASSPATH，值为D:\classpath_test即可
2. linux
使用
```
export CLASSPATH=/opt/work/classpath_test
```
即可，不过这只对当前shell生效，如果需要永久生效需要修改/etc/profile（针对所有用户）,或者用户目录下的.bash_profile文件（针对特定用户），追加上述内容即可，使用source可以立即生效
