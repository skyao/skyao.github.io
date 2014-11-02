title: (记录)Ubuntu下安装Redis
date: 2014-09-25 15:01:29
categories: Redis
tags: [redis,install,ubuntu,linux]
---

记录Redis在ubuntu server上的安装过程。

<!--more-->

##  自动安装

使用 apt-get自动安装redis-server是ubuntu下最简单的方式了：

	sudo apt-get install redis-server

但是这个方案有个很大的问题，就是无法控制安装的版本。像我的ubuntu机器上跑出来的结果居然是要安装redis 2.6.7 版本，而目前最新的是 2.8.17 版本。

所以放弃这个方式。

## 手工安装

手工安装的步骤如下：

1. 下载

		wget http://download.redis.io/releases/redis-2.8.17.tar.gz
		gunzip redis-2.8.17.tar.gz
		tar xvf redis-2.8.17.tar

2. 编译

		make

3. 安装

		sudo make install

## 后台启动

如果需要后台启动，则可以使用redis提供的install_server.sh脚本：

	cd utils
	sudo ./install_server.sh

下面是运行过程，全程使用默认配置：

	$ sudo ./install_server.sh
	Welcome to the redis service installer
	This script will help you easily set up a running redis server
	
	Please select the redis port for this instance: [6379]
	Selecting default: 6379
	Please select the redis config file name [/etc/redis/6379.conf]
	Selected default - /etc/redis/6379.conf
	Please select the redis log file name [/var/log/redis_6379.log]
	Selected default - /var/log/redis_6379.log
	Please select the data directory for this instance [/var/lib/redis/6379]
	Selected default - /var/lib/redis/6379
	Please select the redis executable path [/usr/local/bin/redis-server]
	Selected config:
	Port           : 6379
	Config file    : /etc/redis/6379.conf
	Log file       : /var/log/redis_6379.log
	Data dir       : /var/lib/redis/6379
	Executable     : /usr/local/bin/redis-server
	Cli Executable : /usr/local/bin/redis-cli
	Is this ok? Then press ENTER to go on or Ctrl-C to abort.
	Copied /tmp/6379.conf => /etc/init.d/redis_6379
	Installing service...
	 Adding system startup for /etc/init.d/redis_6379 ...
	   /etc/rc0.d/K20redis_6379 -> ../init.d/redis_6379
	   /etc/rc1.d/K20redis_6379 -> ../init.d/redis_6379
	   /etc/rc6.d/K20redis_6379 -> ../init.d/redis_6379
	   /etc/rc2.d/S20redis_6379 -> ../init.d/redis_6379
	   /etc/rc3.d/S20redis_6379 -> ../init.d/redis_6379
	   /etc/rc4.d/S20redis_6379 -> ../init.d/redis_6379
	   /etc/rc5.d/S20redis_6379 -> ../init.d/redis_6379
	Success!
	Starting Redis server...
	Installation successful!

可以看到名为redis_6379的service已经安装并启动，之后可以通过以下方式操作：

	sudo service redis_6379 start
	sudo service redis_6379 stop

有些文档说还需要执行以下命令来实现开机自动启动redis：

	sudo update-rc.d redis_6379 defaults

但是我安装时，上述命令失败，提示：

*System start/stop links for /etc/init.d/redis_6379 already exist.*

说已经安装过了，好吧，不管它。

## 其他

上述操作完成之后，以下命令就可以直接使用了

1. redis-server
2. redis-cli

