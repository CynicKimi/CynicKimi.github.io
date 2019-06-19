---
title: WIN10下使用自带的SSH和SCP命令
date: 2019/6/18
toc: true
categories:
- tools
tags:
- ssh
- scp
---

发现村里刚通网，才知道win10下的命令行已经支持ssh和scp命令了
<!--more-->
![ssh_and_scp](https://github.com/CynicKimi/images/raw/master/ssh_and_scp.PNG)

### SCP 
使用：
`scp -P <端口> <filename> <用户名>@<IP地址>:<目标目录> `

![scp](https://github.com/CynicKimi/images/raw/master/scp_1.PNG)

<filename>参数还有另外几种写法：
- 在当前目录：`./filename`
- 在其他盘下：`D:/filename`

### SSH
使用：
`ssh -p <端口> <用户名>@<IP地址> `

![ssh_login](https://github.com/CynicKimi/images/raw/master/ssh_login.PNG)

#### 公钥方式登录
ssh登录linux一般有两种方式：
- 用户名和密码（每次登录都需要输入密码）
- 用户名公钥

1.创建公钥私钥：cmd上执行ssh-keygen命令创建密钥对

```
ssh-keygen -t rsa -C  'your email@domain.com'

-t 指定密钥类型，默认即 rsa ，可以省略
-C 设置注释文字，比如你的邮箱，可以省略
```
生成过程中会提示输入密码两次，如果不想在使用公钥的时候输入密码，可以回车跳过；
密钥默认保存位置在 ~/.ssh 目录下，打开后会看到私钥文件 id_rsa 和公钥文件 id_rsa.pub；

![ssh_rsa](https://github.com/CynicKimi/images/raw/master/ssh_rsa.png)

2.复制公钥至服务器
使用 scp 命令将本地的公钥文件 id_rsa.pub 复制到需要连接的Linux服务器的·`~/.ssh`目录下：
`scp -P <端口号>  ~/.ssh/id_rsa.pub  <用户名>@<IP地址>:~/.ssh`

3.把公钥内容追加到`authorized_keys `文件里（不存在则创建一个）：
`cat id_rsa.pub >> authorized_keys`

期间可能出现问题的解决办法：
.权限问题：

```
chmod 700 ~/.ssh/
chmod 600 authorized_keys
```
.ssh问题：
找到/etc/ssh/sshd_config ，把RSAAuthentication和PubkeyAuthentication两行前面的#注释去掉。

4.重启sshd服务(centos7):

`systemctl restart sshd.service`

5.以后无需输入密码，用`-i`参数来使用公钥连接：(缺点：须在./ssh目录下)

`ssh -p <端口> -i id_rsa <用户名>@<IP地址>`

#### 配置config文件进行连接
优点：
- 可以不用输入密码
- 可以不用输入端口号和IP地址
- 可以在任何目录下直接连接
- 可以配置多个服务器

1.在./ssh目录下创建config文件
2.config文件添加配置，格式如下：

```
Host            alias            #自定义别名
HostName        hostname         #替换为你的ssh服务器ip或domain
Port            port             #ssh服务器端口，默认为22
User            root             #ssh服务器用户名
IdentityFile    ~/.ssh/id_rsa    #第一个步骤生成的公钥文件对应的私钥文件
```

![ssh_config](https://github.com/CynicKimi/images/raw/master/ssh_config.png)

以后，就可以用 自定义的别名来登录：`ssh alias`

![ssh_config_login](https://github.com/CynicKimi/images/raw/master/ssh_config_login.png)













