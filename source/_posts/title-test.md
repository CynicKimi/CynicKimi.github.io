---
title: 基础
categories:
- hexo 
tags:
  - Testing
  - Another Tag  
---
### nginx
##### 基本命令
启动位置：`/usr/local/nginx/sbin/nginx`
-c参数加载指定配置文件,不指定加载默认配置文件

重载配置：`/usr/local/nginx/sbin/nginx -s reload`
停止：`/usr/local/nginx/sbin/nginx -s stop`

配置文件位置：`/usr/local/nginx/conf/nginx.conf`

配置多server：配置文件引入多份配置：`include vhost/*.conf`

基本配置
```
server{
    server_name ***域名（多域名空格隔开）
    index **** #允许加载的默认页面
    root ****#指定的项目路径
}
```

##### 修改nginx监听不同的php-fpm

- 修改php-fpm配置文件：www.conf 文件，listen = 127.0.0.1:9000
- 修改nginx配置文件：
```nginx
 location ~ .php$ {
          fastcgi_pass   127.0.0.1:9000;#修改处
          fastcgi_index  index.php;
          fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
          include        fastcgi_params;
 }
```

#### Nginx日志分隔
nginx的日志文件没有rotate功能。编写每天生成一个日志，我们可以写一个nginx日志切割脚本来自动切割日志文件。

第一步就是重命名日志文件，不用担心重命名后nginx找不到日志文件而丢失日志。在你未重新打开原名字的日志文件前，nginx还是会向你重命名的文件写日志，Linux是靠文件描述符而不是文件名定位文件。

第二步向nginx主进程发送USR1信号。nginx主进程接到信号后会从配置文件中读取日志文件名称，重新打开日志文件(以配置文件中的日志名称命名)，并以工作进程的用户作为日志文件的所有者。重新打开日志文件后，nginx主进程会关闭重名的日志文件并通知工作进程使用新打开的日志文件。工作进程立刻打开新的日志文件并关闭重名名的日志文件。然后你就可以处理旧的日志文件了。[或者重启nginx服务]。
- shell脚本处理
```nginx
#nginx日志切割脚本
#author: http://www.nginx.cn

#!/bin/bash
#设置日志文件存放目录
logs_path="/usr/local/nginx/logs/"
#设置pid文件
pid_path="/usr/local/nginx/nginx.pid"

#重命名日志文件
mv ${logs_path}access.log ${logs_path}access_$(date -d "yesterday" +"%Y%m%d").log

#向nginx主进程发信号重新打开日志
kill -USR1 `cat ${pid_path}`
```
- 设置定时任务
`0 0 * * * bash /usr/local/nginx/nginx_log.sh`

这样就每天的0点0分把nginx日志重命名为日期格式，并重新生成今天的新日志文件。

### xdebug
var_dump显示不完全解决方法：
```php
xdebug.var_display_max_depth = -1 
xdebug.var_display_max_children = -1
xdebug.var_display_max_data = -1 
```

### php
开启报错：
```php
ini_set('display_errors', 1);
error_reporting(E_ALL^E_NOTICE);
```

### mongodb
#### 配置文件方式启动

- 创建配置文件：mongodb.conf
```vim
dbpath=/data/mongodb
logpath=/var/log/logconfig.log
port=27017
logappend=true
fork=true
```
- 启动
`./mongod -f ./mongodb.conf`

#### 远程连接失败
- 防火墙检查端口
- 配置文件追加：`bind_ip=0.0.0.0`

#### 关闭mongodb方法
```mongodb
use admin
db.shutdownServer()
```

### vagrant
- 添加box
```
vagrant init box_name    #指定一个名称,会生成一个配置文件Vagrantfile
vagrant box add box_name centos-7.0-x86_64.box #添加一个box
vagrant up  #启动
vagrant ssh #连接
```
- 修改配置文件Vagrantfile
```
#
config.ssh.username = "root"
config.ssh.password = "vagrant"
#指定虚拟机的ip地址，注意网段要与宿主机的ip网段相同
config.vm.network "public_network", ip: "192.168.8.128"
#配置挂在文件
config.vm.synced_folder "../workspace", "/vagrant_data"
```

### linux
查看内存：`cat /proc/meminfo -u`


### nodejs
NPM是随同NodeJS一起安装的包管理工具，理解：类似composer

- 默认路径：`/usr/local/nodejs/bin/npm`

- npm升级到最新版本：`npm install npm -g`
不升级可能会导致错误`npm ERR! ETXTBSY: text file is busy, rename.....`

- `-g`参数表示全局安装

全局安装
1. 将安装包放在 /usr/local 下或者你 node 的安装目录。
2. 可以直接在命令行里使用。

本地安装
1. 将安装包放在 ./node_modules 下（运行 npm 命令时所在的目录），如果没有 node_modules 目录，会在当前执行 npm 命令的目录下生成 node_modules 目录。
2. 可以通过 require() 来引入本地安装的包。


### mysql
拼接字符串用CONCAT:`CONCAT(",", column_name, ",");`




