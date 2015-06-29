title: (记录)在mint linux 下安装nodejs
date: 2015-06-29 11:00:41
categories: Linux
tags: [linux,nodejs,mint]
---

简单记录在mint linux 下安装nodejs的过程。

<!--more-->

# 用apt-get安装

mint linux下通过apt-get直接安装nodejs后，发现运行npm时会报告找不到node命令行，汗一个。

不知道问题发生在哪里，将原来安装的删除：

	sudo apt-get remove nodejs
	sudo apt-get remove npm

重新安装了一次：

	sudo apt-get install curl
	curl -sL https://deb.nodesource.com/setup | sudo bash -
	sudo apt-get install nodejs

这次好了，不过版本似乎比官网最新版本低不少，比如我安装后是0.10版本，但是官网已经是0.12了。不过似乎也不影响使用。

	nodejs --version
	v0.10.38

# 安装最新版本

在[nodejs官网 ](https://nodejs.org/) 看到目前最新版本已经是0.12.5，虽然上面0.10版本也能工作，但是总是不爽。

在官网找到这个文章 [Installing Node.js via package manager](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager#debian-and-ubuntu-based-linux-distributions) 

发现里面安装的方法和上面类似，但是curl的地址不同：

	curl --silent --location https://deb.nodesource.com/setup_0.12 | sudo bash -

这里是setup_0.12，上面是setup，估计是这个原因，而且上面命令执行完成之后，有下面的提示，呵呵。

	Run `apt-get install nodejs` (as root) to install Node.js 0.12 and npm

继续安装：

	sudo apt-get install --yes nodejs
	......
	nodejs --version
	v0.12.5

可以看到现在终于是最新的0.12.5了。

