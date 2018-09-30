### 简介
Secure Shell（安全外壳协议，简称SSH）是一种加密的网络传输协议，可在不安全的网络中为网络服务提供安全的传输环境。Git使用的是ssh协议基于密钥的安全验证，即需要使用ssh-keygen生成一对密钥（公钥和私钥），将公钥添加到gitlab中或者github中即可。平时不注意都是工作和学习的密钥共用，这样不太好，于是便学习了如何配置多个ssh key，使用这篇文章记录一下。

### 准备工作
#### 生成两个ssh-key
```
# 生成一个密钥
ssh-keygen -t rsa -C 'hongyangliao163@163.com' -f ~/.ssh/github_id_rsa
# -t 指定密钥类型，如果没有指定，则默认使用RSA密钥
# -C  提供一个新注释
# -f 指定密钥文件名，如果不指定，默认使用i会在~/.ssh目录下以id_rsa作为密钥文件名（这里由于要生成多个，所以自己自定，不然会覆盖）
# 再生成一个密钥
ssh-keygen -t rsa -C 'hongyangliao163@163.com' -f ~/.ssh/gitee_id_rsa
```
注意：每次生成密钥会询问你是否需要输入密码，输入密码之后，以后每次都要输入密码。请根据你的安全需要决定是否需要密码，如果不需要，直接回车。这样两次生成就会生成四个文件
```
gitee_id_rsa
gitee_id_rsa.pub  
github_id_rsa  
github_id_rsa.pub
```
其中github_id_rsa和github_id_rsa.pub为第一次生成的公钥和密钥，而gitee_id_rsa和gitee_id_rsa.pub为第二次生成的公钥和密钥

### 配置ssh key
本地生成了两对密钥，将上述四个文件中的两个公钥分别配置到github和gitee上（这里是为了测试）

### 配置
准备工作做好了，我们可以来重头戏了，当你clone一个项目，这里有两对密钥，git如何知道你使用的是哪一对密钥呢？
当你直接使用git clone拉取代码时，会给你报一个错
```
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```
很明显了，git不知道私钥是哪个

#### 配置config文件
在~/.ssh下创建一个config文件,内容如下
```
Host gitee
    HostName gitee.com
    Port 22
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/gitee_id_rsa

Host github
    HostName github.com
    Port 22
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/github_id_rsa
```
下面来解释一下
1. Host:可以和HostName配合，如图上述内容所示可以用gitee代替gitee.com，这样就可以使用
```
git@gitee:hongyangliao/mapSource.git
```
来代替
```
git@gitee.com:hongyangliao/mapSource.git
```
2. HostName: 已经讲了
3. Port:ssh的端口号，这个都知道吧
4. PreferredAuthentications: 强制使用什么验证，这里使用的是publickey
5. IdentityFile: 密钥位置
将上述配置好以后，下次再使用Host对应的名字时，就会使用其对应的配置，这样也就能实现配置多个ssh key 你也可以对同一个代码仓库如github做两次配置，只有Host不同如一个将github01
一个github02这样就可以使用的公钥

### 配置用户信息
配置完多个ssh key之后，有的同学想要实现不同的key对应不同的用户信息，这个我现在还没找到解决办法，但是可以变相实现
可以先使用
```
git config --global user.name "hongyangliao"
git config --global user.email hongyangliao163@163.com
```
设置全局的信息，然后可以在每个项目中单独设置,如
```
git config user.name "hongyangliao_test"
git config user.email hongyangliao_test@163.com
```
这样单独设置的项目就会覆盖全局的配置，这样name和email也就能和全局分开了，没有单独设置的会使用全局的设置
