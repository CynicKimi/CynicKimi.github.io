---
title: Hexo搭建
date: 2019/6/3
toc: true
categories:
- Hexo
tags:
- Hexo
---

以前博客在CSDN，自己用Tampermonkey插件修改了html结构，弄了一个整洁版的，但还是放弃了，理由是：广告真的太多，而且CSDN内相同的博文太多了。
期间想过自己搭建一个博客，但是由于强迫症，考虑到页面的美观和数据库设计问题以及日后迁移问题，所以迟迟没有动手。最后还是决定把数据托管在GitHub，并使用Hexo。

## Hexo + Github pages

### 创建github pages
官方文档：`https://pages.github.com/`
新建一个仓库 New repository，仓库命名为 `<username>.github.io`

### 安装Hexo
```
npm install hexo-cli -g //使用npm安装hexo
hexo init <directory> //初始化博客,<directory>为要创建的文件夹名
cd <directory> //进入博客文件夹
npm install //npm安装依赖文件
hexo server //启动hexo，若失败，请先执行'npm install hexo-server --save’
```
本地访问：`localhost:4000`
(注：hexo命令可能会执行失败，用绝对路径执行命令，比如我的hexo在`/usr/local/nodejs/bin/hexo`;也可以将你的hexo路径加到环境变量，之后便可用hexo命令)

### 部署到线上GitHub pages
1.打开hexo配置文件`_config.yml`，找到deploy(注意缩进)：
```
deploy:
  type: git
  repo: git@github.com:<username>/<username>.github.io
  branch: master
```
(个人建议用`git@github.com:***`,而不是`https://github.com/***`,这样就不用每次推到github上的时候都要输入用户名和密码了)
2.推到github
```
hexo g//hexo generate生成静态文件
hexo d//hexo deploy部署到线上,`hexo d`报错时，执行`npm install hexo-deployer-git`;
```
(`hexo clean`:清除缓存文件db.json 和已经生成的静态文件夹public)

## 使用两个分支保存源文件
`hexo d`操作把编译后的静态文件推到GitHub上，但是以后换电脑了怎么办？理想状态是你还可以找的到以前源文件，但是总是有最坏的情况出现：源文件也找不到了。为了避免这种情况，不如也把源文件放在GitHub。
具体操作：
- 线上创建一个分支（目的存放源文件），并将其设为默认分支
-  git clone 分支（目的为了获得.git文件夹，即获得新分支的管理权限）
-   将.git文件夹复制到hexo文件夹下（博客根目录），以后在这个文件夹下git push，则会将源码推到默认分支，而hexo g和hexo d操作后则会将静态文件提交到master分支

以后换到新环境，只需要`git clone <默认分支地址>` ,然后执行 `npm install`安装相关依赖就行了

## Hexo主题
以前也弄过花里胡哨的博客，但现在只喜欢简单的，所以采用了大道至简的主题，并做了一些小修改。
文档地址:`https://github.com/tufu9441/maupassant-hexo`

