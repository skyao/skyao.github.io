title: (记录)在ubuntu服务器上搭建全套开发和运行环境
date: 2014-11-25 14:04:50
categories: Linux
tags: [linux,ubuntu,server]
---

资料收集，在ubuntu服务器上搭建全套开发和运行环境。

<!--more-->

#  前言

一直都是在家里的下载机上，使用vmware安装linux服务器，搭建自己的开发测试环境。

前段时间偶尔看到vpsdime推出的高内存VPS方案，4核+6G内存的VPS才一个月7美元。这个价格，结果只能是买买买啊。

现在打算将全套开发测试环境搬过去，记录一下全过程，以备下次使用。

# 服务器配置

在选择linux服务器版本，犹豫了一下，最后还是选择了自己最熟悉的ubuntu server 14.04版本。主要是这些年用apt-get用的比较爽...

## 增加user

增加自己习惯的user，这个user需要拥有sudo的权限：

	adduser -g sudo sky

# 开发环境搭建

## Java相关

### 安装JDK

安装jdk7，暂时不上jdk8. 懒得手工安装了，直接apt-get。

	sudo add-apt-repository ppa:webupd8team/java
	sudo apt-get update
	sudo apt-get install oracle-java7-installer
	sudo apt-get install oracle-java7-set-default

[参考地址](http://ubuntuhandbook.org/index.php/2014/02/install-oracle-java-6-7-or-8-ubuntu-14-04/)。

运行java --version的结果：

	java version "1.7.0_72"
	Java(TM) SE Runtime Environment (build 1.7.0_72-b14)
	Java HotSpot(TM) 64-Bit Server VM (build 24.72-b04, mixed mode)

执行 echo $JAVA_HOME看了一下JAVA_HOME的设置：

	/usr/lib/jvm/java-7-oracle

## 版本控制

### 安装Git

为了拿到最新版本的git，增加仓库：

	sudo add-apt-repository ppa:git-core/ppa
	sudo apt-get update
	sudo apt-get install git

[参考地址](http://stackoverflow.com/questions/19109542/installing-latest-version-of-git-in-ubuntu)

### 安装gitolite

先增加git的group和user：

	addgroup git
	adduser --ingroup git git

然后用其他账号登录服务器，先准备一份ssh key：
	
	ssh-keygen -t rsa

生成的~/.ssh/id_rsa.pub 文件后面要用到。

然后su 到git用户，执行下面的命令：

	su git
	git clone git://github.com/sitaramc/gitolite
	mkdir -p $HOME/bin
	gitolite/install -to $HOME/bin
	$HOME/bin/gitolite setup -pk path/to/above/id_rsa.pub

输出如下：

	Initialized empty Git repository in /home/git/repositories/gitolite-admin.git/
	Initialized empty Git repository in /home/git/repositories/testing.git/
	WARNING: /home/git/.ssh missing; creating a new one
	    (this is normal on a brand new install)
	WARNING: /home/git/.ssh/authorized_keys missing; creating a new one
	    (this is normal on a brand new install)

退出git账号，回到刚才的账号，试试git clone。

	git clone git@localhost:gitolite-admin.git
	git clone git@localhost:testing.git

不出意外的话应该可以clone下来，说明	gitolite 安装完成。还可以这样检查，执行ssh git@localhost命令：

	ssh git@localhost
	PTY allocation request failed on channel 0
	hello root, this is git@sky1 running gitolite3 v3.6.2-4-g2471e18 on git 2.1.3
	
	 R W    gitolite-admin
	 R W    testing
	Connection to localhost closed.

安装时忘了先创建非root账号了，所以上面看到的用户是root。后来创建了sky账号，需要用root账号将sky的ssh key复制到gitolite-admin/keydir，然后提交。另外需要修改gitolite-admin/conf/gitolite.conf文件，增加sky的仓库访问权限。为了方便起见，建立group admin：

	@admin=root sky
	
	repo gitolite-admin
	    RW+     =   @admin
	
	repo testing
	    RW+     =   @all

### 倒入原有git仓库

将原有gitolite下的git 仓库打成tar包，然后传到新机器。解开tar，将tar包中repositories下各个git仓库复制到/home/git/repositories/下(记得gitolite-admin除外)，然后修改gitolite-admin/conf/gitolite.conf，增加各个仓库的对应访问信息即可。

## 打包发布工具

### 安装ant

### 安装ivy

### 安装maven

[参考地址](http://www.sysads.co.uk/2014/05/install-apache-maven-3-2-1-ubuntu-14-04/)，步骤如下：

1. Remove older version
2. Add following lines to sources.list
3. Update Repository and Install
4. Add Symbolic Link

具体执行命令依次如下：

	sudo apt-get remove maven2
	sudo add-apt-repository "deb http://ppa.launchpad.net/natecarlson/maven3/ubuntu precise main"
	sudo apt-get update
	sudo apt-get install maven3
	sudo ln -s /usr/share/maven3/bin/mvn /usr/bin/mvn


### 安装artifactory

### 安装gradle


