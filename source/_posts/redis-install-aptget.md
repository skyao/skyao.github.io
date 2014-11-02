title: (记录)Ubuntu下用apt-get安装最新版本的Redis
date: 2014-11-02 10:19:01
categories: Redis
tags: [redis,install]
---

记录使用apt-get在ubuntu server上安装最新版本Redis的过程。

<!--more-->

##  前言

在ubuntu安装redis最简单的方式，就是使用apt-get

	sudo apt-get install redis-server

但是这个方式有个问题，就是仓库中的redis-server很可能不是最新的版本。比如我用ubuntu 14.04 server，apt-get udpate 到最新后，redis-server的版本也只有2.8.4。而官网最新已经到了2.8.17. 

在之前的Blog里面，因为使用这个原因，我采用了手工安装的方式。

今天在准备另外一个环境时，想了想apt-get应该更方便一些，只要仓库中的redis-server版本足够新就好了。

## 准备工作

要拿到最新的redis-server版本，就必须将redis的仓库加入到源。方法有两种：

### 方式一：修改source文件

	deb http://ppa.launchpad.net/rwky/redis/ubuntu trusty main 
	deb-src http://ppa.launchpad.net/rwky/redis/ubuntu trusty main

### 方式二：用add-apt-repository命令

	add-apt-repository -y ppa:rwky/redis

这个方式无疑要方便很多。

不过 add-apt-repository 命令一般系统是没有自带的，所以还需要自己安装一下。

这个命令的安装有点麻烦，ubuntu不同版本中这个命令的安装方式不同：

1. 对于12.04以及以下版本，需要安装python-software-properties

	sudo apt-get install python-software-properties

2. 对于12.10以及以上版本，需要安装software-properties-common

	sudo apt-get install software-properties-common

比较爽快而无需费脑的方法是两个都安装一下......

## 安装 

### 首次安装Redis

安装过程简单，update再install就好了，加上前面准备add-apt-repository，命令依次如下：

	sudo apt-get install -y python-software-properties
	sudo apt-get install software-properties-common
	sudo add-apt-repository -y ppa:rwky/redis
	sudo apt-get update
	sudo apt-get install -y redis-server

如果是第一次用apt-get安装redis-server，那么这样就搞定了。

### 更新旧版redis

如果之前已经用apt-get安装过redis-server的旧版本，再执行apt-get install时就有可能遇到问题。

我遇到的错误信息如下：

> 
Unpacking redis-server (2:2.8.17-rwky1~trusty) ...
dpkg: error processing archive /var/cache/apt/archives/redis-server_2%3a2.8.17-rwky1~trusty_amd64.deb (--unpack):
 trying to overwrite '/usr/bin/redis-check-dump', which is also in package redis-tools 2:2.8.4-2
dpkg-deb: error: subprocess paste was killed by signal (Broken pipe)
Errors were encountered while processing:
 /var/cache/apt/archives/redis-server_2%3a2.8.17-rwky1~trusty_amd64.deb
E: Sub-process /usr/bin/dpkg returned an error code (1)

为了解决问题，决定先uninstall掉老版本的redis

	apt-get remove redis-server
	apt-get autoremove

再次执行apt-get install就可以顺利安装了。

## 参考信息

1. http://tosbourn.com/install-latest-version-redis-ubuntu/
2. http://stackoverflow.com/questions/13018626/add-apt-repository-not-found
3. http://askubuntu.com/questions/88265/how-to-find-the-latest-version-of-redis-server-in-repos

	