title: 自动出书-实现gitbook的持续集成
date: 2015-02-17 15:18:54
categories: 杂谈
tags: [gitbook]
---

总结并分享一下将为gitbook搭建的持续集成环境，实现书籍编写提交后，CI自动获取最新内容生成页面并发布到web服务器的全过程。

<!--more-->

之前将gitbook生成的html page发布github pages上，这种方式grunt提供了很好的支持，github也给力。

但是对于一些不适合发布到github的内容，比如公司内部的一些文档或者知识分享，就需要考虑其他的方式。

# 思路

解决问题的思路比较直接：

1. 我们需要一个地方存放源文件，这个自然是git了
2. 我们需要一个CI服务器，这个自然是jenkins了
3. 我们需要一个web服务器用于发布生成的内容，因为都是静态页面，这个选择有很多，个人喜欢nginx

# 准备工作

## 准备git

有github的时候用github，没有github的时候用gitlab。

gitlab的安装配置见这里。

gitlab安装好之后，新建一个project，然后将书的内容提交到这个project，做好准备。

## 准备CI服务器

## 准备nginx 服务器

## 准备运行环境

安装node.js运行环境：

	sudo apt-get install nodejs
	sudo apt-get install nodejs-legacy

注意在linux上需要安装nodejs-legacy，否则会报错：

> /usr/bin/env: node: No such file or directory

安装npm：

	sudo apt-get install npm

安装grunt:

	sudo npm install -g grunt
	sudo npm install -g grunt-cli

TBD
	npm install .

## 手工测试

在用jenkins跑之前，先手工试试能不能工作。



# 搭建CI Job


