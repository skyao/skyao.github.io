title: 配置Jenkins服务器
date: 2014-11-27 16:22:27
categories: CI
tags: [CI,jenkins]
---

资料收集，在ubuntu服务器上搭建并配置Jenkins CI服务器。

<!--more-->

#  安装Jenkins

在上一个Blog中已经介绍了如何在ubuntu服务器上安装Jenkins并配置用户登录。


# 安装插件

## 取消不需要的已安装插件

以下插件暂时用不上，先取消，万一需要了再启用不迟：

1. CVS plugin
2. LDAP plugin
3. PAM Authentication Plugin
4. Subversion Plugin
5. Translation Assistance Plugin
6. Windows Slaves Plugin

## 安装新插件

需要安装以下插件：

### git相关插件

1. Git Client Plugin
2. Git Parameter Plugin

# 配置Jenkins

## 修改系统配置

登录后，系统管理 -> 系统设置。

在这里配置好JDK/ant/maven/git.

## 准备git

需要为jenkins用户准备git访问权限：

	sudo su jenkins
	ssh-keygen -t rsa

将得到的pub key提交到gitolite-admin，文件名为jenkins@local.pub

然后配置jenkins的git权限，这个需要设置为只读：

	@ci=jenkins
	
	repo task-engine
	    RW+     =   @admin 
	    R       =   @ci

这样jenkins用户就可以用git来获取代码了。

## 准备maven

### 配置artifactory

### 配置upload账号