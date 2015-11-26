title: (记录)gitlab安装配置
date: 2015-02-16 15:44:29
categories: Git
tags: [git,gitlab]
---

资料记录，关于gitlab的安装配置。

<!--more-->

#  前言

最近发现gitlab这个类似github的git管理系统，感觉比gitolite要好用，所以决定以后用gitlab来替代gitolite。

这里记录gitlab相关的安装和配置过程，备忘备用。

# 安装

安装方式参考gitlab官方资料，打开 https://about.gitlab.com/downloads/ 页面，"select operating system"选"ubuntu14.04"，然后按照指示执行安装：

	sudo apt-get install openssh-server
	sudo apt-get install postfix
	wget https://downloads-packages.s3.amazonaws.com/ubuntu-14.04/gitlab_7.7.2-omnibus.5.4.2.ci-1_amd64.deb
	sudo dpkg -i gitlab_7.7.2-omnibus.5.4.2.ci-1_amd64.deb

安装过程结束之后再执行命令：

	sudo gitlab-ctl reconfigure

之后就可以通过浏览器访问了，默认是用80端口。

# 配置

## 修改端口

安装完成之后默认是用80端口，如果需要用其他端口，需要修改一下配置文件。

	sudo vi /etc/gitlab/gitlab.rb

修改external_url，直接增加端口号即可，比如我这里用8800端口：

	external_url 'http://skyserver:8800'

也可以在这里修改host为真实的公网host,比如"http://basiccloud.net:8800"

修改后再次执行"sudo gitlab-ctl reconfigure"以便配置修改生效。

注意：除了这个端口外，还有一个unicorn用的端口，默认是8080，如果8080端口被其他程序占用。那么unicorn就会无法启动，显示为502错误，"GitLab is not responding"。

这种情况下修改unicorn的配置：

	unicorn['listen'] = '127.0.0.1'
	unicorn['port'] = 8801

完成后通过浏览器访问(http://skyserver:8800).

Update(2015/11/26): 在ubuntu14.04服务器上, 发现按照上面的修改并执行"sudo gitlab-ctl reconfigure", 会发现默认的8080和新修改的8801两个端口都会同时被gitlab占用. 最后只有重启ubuntu服务器才能释放出8080端口, 原因不明.

## 账号设置

默认管理员密码如下：

	Username: root
	Password: 5iveL!fe

root账户第一次登录时会要求修改密码，为了安全我们在管理页面可以新建一个普通用户，注意新建用户过程中不能设置密码，在建立成功之后可以edit这个账号然后这里可以设置密码。

## 关闭注册功能

默认注册功能是开启的, 对于个人的gitlab, 没有对外公布的必要(有就直接上github了), 因此需要考虑关闭注册功能.

用管理员账号登录之后, 进入"Admin area", 点"settings", 取消"Signup enabled".
