title: 使用shadowsocks实现科学上网
date: 2015-07-14 10:08:24
categories: 杂谈
tags: [翻墙,shadowsocks]
---

如何搬梯子翻墙实现科学上网，对每一个码农来说都是必备的技能。

而翻墙的方式有很多，之前一直用puff商业版，最近换了一个思路，改用shadowsocks做梯子。

<!--more-->

# 前言

shadowsocks有提供商业服务，购买账号就可以使用它们提供的服务器端，自己只要安装客户端就好了。但是，对于有国外主机的同学，也可以自己动手搭建自己的shadowsocks服务器。

当然，用这种方式意味着：首先，你得有一台国外的主机！

# 安装shadowsocks服务器

下面是从网上找来的一键式安装脚本，用这个比自己apt-get再手工配置要轻松的多：

	wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh 
	chmod +x shadowsocks.sh 
	./shadowsocks.sh 2>&1 | tee shadowsocks.log

安装成功之后，显示以下信息：

	Congratulations, shadowsocks install completed!
	Your Server IP:  192.121.***.*** 
	Your Server Port:  8989 
	Your Password:  ***** 
	Your Local IP:  127.0.0.1 
	Your Local Port:  1080 
	Your Encryption Method:  aes-256-cfb 
	
	Welcome to visit:http://teddysun.com/342.html
	Enjoy it!

默认使用8989端口。安装完成之后，使用以下命令可以操作shadowsocks服务器：

- 启动：/etc/init.d/shadowsocks start
- 停止：/etc/init.d/shadowsocks stop
- 重启：/etc/init.d/shadowsocks restart
- 查看状态：/etc/init.d/shadowsocks status

# 安装windows客户端

发现网上下载的版本很多是在原版基础上加装广告的版本，太无耻了。

找了一下最好的下载网站就是shadowsocks在github的网页：

https://github.com/shadowsocks/shadowsocks-csharp/releases

对于win8以上操作系统请选择 Shadowsocks-win-dotnet4.0-2.4.zip 版本。无需安装，直接解压运行Shadowsocks.exe即可。

配置时注意一个细节：**Shadowsocks的socks代理一定要设置为 SOCKS v5**。

# 安装android客户端

在这里下载最新的android客户端，请继续忽略各处app store。

https://github.com/shadowsocks/shadowsocks-android/releases

安装配置好就可以在手机上看u2b啦！

# 安装linux客户端

从 [shadowsocks-gui](https://github.com/shadowsocks/shadowsocks-gui) 得知，linux客户端是shadowsocks-qt5，居然还提供中文版本的[安装指南](https://github.com/librehat/shadowsocks-qt5/wiki/%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97) 。赞一个！

按照安装指南，执行安装：

	sudo add-apt-repository ppa:hzwhuang/ss-qt5
	sudo apt-get update
	sudo apt-get install shadowsocks-qt5

安装完成后，执行命令启动：

	ss-qt5

图形界面上可以找到shadowsocks-qt5的图标，或者gnome do 之类的工具也可以。

启动后配置和windows版本类似。
