---
title: windows下的命令行工具Cmder
date: 2019/6/19
toc: true
categories:
- tools
tags:
- cmder
- tools
---

windows的神器，传言不虚。可以在同一个窗口打开多个Tab，还可以一个Tab分屏多个终端，简直不要太棒。
<!--more-->

## 添加cmder here
右键时可以cmder here，就不用每次打开都项目都去`cd`
以**管理员身份**运行`Cmder.exe /REGISTER ALL`
(注：没有配置环境变量时，要到该文件的目录去执行该命令，不然会找不到cmder.exe文件。但我没有配置到环境变量，因为我安装了`Listary`,要启动时也可以快速找到)
![cmder](https://github.com/CynicKimi/images/raw/master/cmder_here.PNG)

## 改变命令提示符`λ`为`##  

### cmder
- 找到文件`cmder\vender\clink.lua`
- 将`local lambda = "λ"`修改为`local lambda = "$"`

![cmder](https://github.com/CynicKimi/images/raw/master/prompt.png)
(注：看了好多文章都是修改上一句的`{lamb}`为`$`,也可以实现一样的效果，但是不建议这么改。
虽然没学过lua，但是也可以看懂下面语法，最终用了`string.gsub`函数进行了一个字符串替换，将`{lamb}`替换成`lambda`变量的内容，而且上面的注释语句也说明了，有时候可能提示符会变成`(env)$`,如果照网上的改法，就永远只能是`$`了)

### bash
进入bash还是发现提示符还是`λ`。。。
- 找到文件`cmder\vender\git-for-windows\etc\profile.d\git-prompt.sh`
-   将`PS1="$PS1"'λ '` 改为`PS1="$PS1"'$ '`

## 设置 ll命令
cmder没有`ll`命令，可以通过设置alias来实现
找到`cmder\config\user-aliases.cmd`文件
添加以下几行带，代码、
```
la=ls -aF --show-control-chars -F --color $*
ll=ls -alF --show-control-chars -F --color $*
```

## 设置启动的默认目录
- `win+alt+p`打开设置
- 到Startup-Tasks菜单下，选择对应的终端，加入` -new_console:d:<dirname>`
![设置启动目录](https://github.com/CynicKimi/images/raw/master/start_dir.png)


## 配置git bash
如果下载了full版本，自带了git-for-window,就不需要配置了。
具体配置方法如下：
![cmder](https://github.com/CynicKimi/images/raw/master/git_cmder.png)

## 快捷键命令

| 快捷键   | 说明 |
| :------: | :--: |
| ctrl + ` | 调出和隐藏cmder |
| win + alt + p | 打开设置面板 |
| ctrl + t | 新打开终端窗口 |
| shift + alt + number | 新打开终端窗口(具体见setting-startup-tasks) |
| alt + enter | 窗口全屏显示 |
| Shift + mouse | 从缓冲区中选择并复制文本 |
| Right click / Ctrl + Shift + v | 粘贴文本 |
| ctrl+u | 删除当前命令行内容 |
| ctrl+l | 清屏 |




