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

修改后再次执行"sudo gitlab-ctl reconfigure"以便配置修改生效。

注意：除了这个端口外，还有一个unicorn用的端口，默认是8080，如果8080端口被其他程序占用。那么unicorn就会无法启动，显示为502错误，"GitLab is not responding"。

这种情况下修改unicorn的配置：

	unicorn['listen'] = '127.0.0.1'
	unicorn['port'] = 8801

# 集成

## 使用外部nginx服务器

gitlab默认使用自带的Nginx，如果需要使用外部Nginx，可以修改配置

TBD： 居然没有配置成功......下次再试

# 使用


