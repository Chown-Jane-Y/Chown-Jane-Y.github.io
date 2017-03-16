---
title: 如何在不同电脑上同时写hexo博客？
date: 2017-03-15 11:04:24
tags: Hexo
categories: 博客搭建
description:
---


个人PC 和 工作PC

<!--more-->
### 在github上新建一个repository

不用初始README.md
此时只有一个空的master分支

### 在个人PC初始化一个Hexo项目

![Alt text](./1489588922319.png)

npm install
npm install hexo-deployer-git --save

然后将 source  scaffolds themes _config.yml替换成原来的。这里把themes/jacman中的.git/文件夹删除，否则推送到hexo分支后jacman为空。

### 新建分支hexo
 
 将该目录初始化为本地仓库
 git init
 
 先把hexo下所有文件推送到master分支
git remote add origin git@github:chown-jane-y/chown-jane-y.github.io.git
git add .
git commit -m "first add hexo source code"
git push origin master

再新建一个分支hexo，这样hexo分支和master分支的内容一样，都是hexo的源文件。

并把hexo设为默认分支，便于在另外一台机器上克隆下来直接进入hexo分支。

### 部署博客

hexo g -d
静态文件部署到master分支，这是到github查看两个分支的内容，hexo分支里是源文件，master里是静态文件
![Alt text](./1489590203468.png)


![Alt text](./1489590238903.png)

























### 将更新内容推送到hexo分支

git checkout -b hexo
git pull origin hexo

另外别忘了
git add .
git commit -m  ""
git push origin hexo
这样才能在另外的机器上pull下来，保持同步。


## 工作PC

### clone
git clone git@github.com:chown-jane-y/chown-jane-y.github.io.git

克隆下来的仓库里没有node_modules，所有要重新安装hexo，但不需要hexo init

npm install hexo
npm install
npm install hexo-deployer-git --save

新建一篇文章测试
hexo new "work PC test"

推送到hexo分支
git add .
git commit -m "add work PC test"
git push origin hexo

部署到master分支
hexo g -d




## 日常操作

每次写博客前
git pull origin hexo
保证同步

hexo new "title"
git add .
git commit -m "add article xxx"
git push origin hexo
hexo g -d
<!--more-->