---
title: Hexo的目录结构及作用（备忘）
date: 2017-02-28 09:20:14
tags: Hexo
categories: 博客搭建
description: Hexo目录下各文件夹的作用，方便管理自己的博客。
---
## 一、主目录结构
```
|-- _config.yml
|-- package.json
|-- scaffolds/
|-- scripts/
|-- source/
   |-- _drafts/
   |-- _posts/
|-- themes/
```

<!-- more -->

## 二、主目录介绍

### _config.yml

全局配置文件

网站的很多信息都在这里配置，诸如网站名称，副标题，描述，作者，语言，主题，部署等等参数。配置文件里已经有相关的注释。

### package.json

hexo框架的参数，不用管，里面参数是固定的。如果不小心把它删掉了，没关系，新建一个文件，讲内容写入文件，保存就OK了。如下：
```
{
  "name": "hexo",
  "version": "2.4.5",
  "private": true,
  "dependencies": {}
}
```
参数也很容易理解，该文件基本上也不需要操作。

### scaffolds

scaffolds是“脚手架、骨架”的意思，当你新建一篇文章（hexo new 'title'）的时候，hexo是根据这个目录下的文件进行构建的。相当于是一个模版。

### scripts

脚本目录，此目录下的JavaScript文件会被自动执行。

### source

这个目录很重要，新建的文章都是在保存在这个目录下的，有两个子目录： `_drafts` ， `_posts` 。需要新建的博文都放在 `_posts` 目录下。

`_posts` 目录下是一个个 markdown 文件。文章就在这个文件中编辑。

`_posts` 目录下的md文件，会被编译成html文件，放到 `public` （此文件现在应该没有，因为你还没有编译过）文件夹下。

### themes

网站主题目录，hexo有非常好的主题拓展，支持的主题也很丰富。该目录下，每一个子目录就是一个主题。

<!-- more -->