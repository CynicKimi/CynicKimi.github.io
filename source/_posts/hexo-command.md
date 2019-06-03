---
title: Hexo搭建
date: 2019/6/3
toc: true
categories:
- Hexo
tags:
- Hexo
---

以前博客在CSDN，自己用Adblock+Tampermonkey插件弄了一个整洁版的，但还是放弃了，理由是：广告真的太多，而且CSDN内相同的博文太多了。
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



