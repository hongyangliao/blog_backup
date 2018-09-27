### 简介
alias命令用来设置指令的别名。我们可以使用该命令可以将一些较长的命令进行简化。使用alias时，用户必须使用单引号''将原来的命令引起来，防止特殊字符导致错误。
在开发中时常要进入centos服务器某些比较深的目录，每次cd比较烦，后来在网上发现了alias命令，可以设置别名，这样就可以使用一个命令进入想要进的目录了。

### 常用参数
#### 显示当前设置的别名
```
# 两个都可以
alias
alias -p
```
如我本机的结果为
```
alias cp='cp -i'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias rm='rm -i'
alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
```

#### 显示某个别名
```
alias 别名
```

#### 设置别名
```
alias 别名='命令 参数'
```
如我将别名cdw设置cd /opt/work
```
alias cdw='cd /opt/work'
```
就可以使用cdw执行cd /opt/work命令了

### 设置永久生效
刚刚使用直接在shell中使用alias，只对当前shell生效，要想永久生效，需要将其保存在文件中
#### 当前用户生效
需要将
```
alias 别名='命令 参数'
```
保存在 **~/.bashrc** 文件中，下次使用这个用户登录shell，也可使用刚刚设置的别名，另外 **~/.bashrc** 文件中已经默认设置一些别名，如下所示
```
# .bashrc

# User specific aliases and functions

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi
```

#### 所有用户生效
跟‘当前用户生效’的配置差不多，不过需要保存在 **/etc/bashrc** 下

