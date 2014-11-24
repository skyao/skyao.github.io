title: (记录)升级hexo和pacman到最新版本
date: 2014-11-24 17:39:26
categories: 杂谈
tags: [hexo]
---

记录hexo和pacman主题的升级。

<!--more-->

最近重装了操作系统，包括公司笔记本和家里开发机，在安装hexo时，发现hexo的版本已经升级，新版本下运行hexo命令会报错。

懒得去debug到底是哪里出错，干脆将hexo和我使用的pacman主题都升级到最新版本好了。

# hexo 安装

重新安装了一把hexo，从node.js安装开始，然后通过npm安装

	npm install -g hexo

如果访问网络需要使用代理服务器，则需要先设置npm：

	npm config set proxy http://proxy.company.com:8080
	npm config set https-proxy http://proxy.company.com:8080

顺便安装一下hexo插件（其实这里我不确认是否是必须安装）

	npm install hexo-generator-feed --save
	npm install hexo-generator-sitemap --save

# pacman主题安装

拿到最新的pacman文件，需要更新一些文件，上次安装时没有记录下来，导致这次我不得不重新翻了一遍文档和现有的代码。为了避免重蹈覆辙，我决意将修改列表记录下来：

	/_config.yml
	/themes/pacman
	/source/*			 这个目录下的东西全部需要保留
	/themes/pacman/_config.yml
	/themes/pacman/source/img

