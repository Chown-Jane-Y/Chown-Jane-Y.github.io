---
title: 使用virtualenv工具构建Django开发环境
date: 2017-03-15 23:52:18
tags: [Python, Virtualenv]
categories: Python学习之路
description:
---



在开发Python应用程序的时候，系统安装的Python3只有一个版本：3.4。所有第三方的包都会被pip安装到Python3的site-packages目录下。

如果我们要同时开发多个应用程序，那这些应用程序都会共用一个Python，就是安装在系统的Python 3。如果应用A需要django1.6，而应用B需要django1.10怎么办？

这种情况下，每个应用最好需要各自拥有一套独立的Python运行环境。virtualenv就是用来为一个应用创建一套隔离的独立的的Python运行环境。

<!--more-->
## 安装virtualenv

直接在系统默认的python环境下安装virtualenv，因为virtualenv也是python的一个包。

在命令行输入：
```python
pip install virtualenv
```

## 创建独立运行环境

假定我们要开发一个新项目(基于Django1.10.6和anaconda3的web项目)，需要一套独立的运行环境，可以这么做：
1. 第一步：创建目录
```java
mkdir my_django_web
```
2. 第二步：创建一个独立的python运行环境，这里根据需求命名为`env_anaconda3_django1.10.6`：
```java
cd my_django_web   //进入项目目录
virtualenv --no-site-packages env_anaconda3_django1.10.6  //创建虚拟环境，且系统的第三方包不会复制过来
```
3. 第三步：在该环境下安装Django-1.10.6
```
cd anaconda3_django1.10.6_env   //进入该虚拟环境目录
Scripts\activate     //激活该环境
(venv_anaconda3_django1.10.6) D:\PyWorkspace\my_first_django>pip install django==1.10.6
```
执行`activate`脚本后，才能进入该虚拟环境，前面会有`(venv_anaconda3_django1.10.6)`前缀，表示已经在该环境下。接着在该环境下`pip install`你想要的第三方包，不会影响其他的运行环境。

4. 第四步：创建Django项目
```
(venv_anaconda3_django1.10.6)D:\PyWorkspace\my_first_django> django-admin startproject my_first_django
```
在`env_anaconda3_django1.10.6`环境下，用pip安装的包都被安装到`env_anaconda3_django1.10.6`这个环境下，系统Python环境不受任何影响。也就是说，`env_anaconda3_django1.10.6`环境是专门针对my_first_django这个项目创建的。接着就可以在这个环境中进行该项目的开发，不会影响其他的项目。

5. 第五步：退出该环境
```
Scripts\deactivate
```
<!--more-->