#### 本地项目关联远程仓库
##### 在GitHub上创建一个空项目

##### 进入本地项目的目录,初始化项目为Git项目
```
git init
```

##### 将本地项目与远程仓库关联
```
git remote add origin git@github.com:hongyangliao/test.git
```
origin为远程仓库别名，git@github.com:hongyangliao/test.git为远程仓库地址


##### 将项目中的代码推到远程仓库中
```
# 添加到暂存区
git add .
# 提交到本地仓库
git commit -m 'init porject'
# 推送代码到远程仓库
git push -u origin master
```

#### 关联远程分支
```
git checkout -b dev origin/dev
```
此命令作用是创建一个本地分支并关联远程分支，同时转换到刚刚创建的本地分支

dev为本地分支

origin/dev为远程分支


#### 创建本地分支并推送到远程
```
# 创建并切换到创建的本地分支
git checkout -b dev
# 提交本地分支作为远程分支
git push origin dev:dev
```

#### 远程仓库的使用
##### 查看当前配置的远程仓库
```
git remote -v
```

#### 分支的使用
##### 创建分支
```
git branch dev
```

##### 删除本地分支
```
git branch -d dev
```

##### 删除远端分支
```
git push origin :dev
```

##### 切换分支
```
git checkout dev
```

##### 查看远程分支
```
git branch -a
```

##### 查看本地分支
```
git branch
```

#### git clone非22端口
命令clone项目时，如果repository的SSH端口不是标准22端口时（例如，SSH tunnel模式，等等），可以使用如下命令：

```
git clone ssh://git@hostname:port/.../xxx.git
```

#### 迁移gitolite
1. 在新的服务器上安装gitolite,并为其指定管理员
2. 进入gitolite初始化后的目录gitolite-admin内，将之前线上config和keydir内文件全部拷贝过来
3. git push,提交
4. 将原有的giolite的repositories下的文件拷贝到新服务器的repositories下
5. 在gitolite-admin中git pull获取最新的数据
