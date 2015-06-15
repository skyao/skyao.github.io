title: 在gitbook中实现多级导航栏的支持
date: 2015-06-15 14:36:49
categories: 杂谈
tags: [gitbook]
---

总结并分享一下如何在gitbook中实现多级导航栏的支持。

<!--more-->

# 前言

gitbook默认只支持两层的导航栏，在遇到内容比较多需要章节层次比较深时，很不方便。

好在后来gitbook终于提供了多级导航栏的支持。

# 实现

具体做法挺简单，以下是一个例子：

	bui# Summary

	* [介绍](introduction/index.md)
	    * [mercury 信息](introduction/information.md)
			* [Google Dapper](dapper/index.md)
	* [mercury实现](implementation/index.md)

就改动两个地方：

1. 第一行从 "# Summary" 修改为 "bui# Summary" 
2. 章节的层次结构按照原来的格式要求继续缩进

实际测试结果，可以支持三级和四级，再多我就没有测试了，一般也不需要。

注意：需要新版本的gitbook，具体是哪个版本开始才支持我没有去考究，反正记得用最新版本就是。

# grunt

为了方便发布到github，我选择了使用grunt。

同样为了支持多层导航栏，grunt相关的插件也需要使用新版本。

具体需要修改package.js中的以下内容，目前最新的版本是这样：

    "devDependencies": {
        "grunt": "~0.4.5",
        "grunt-gitbook": "~1.5.0",
        "grunt-gh-pages": "~0.10.0",
        "grunt-contrib-clean": "~0.6.0",
        "grunt-http-server": "~1.4.0"
    },
    "peerDependencies": {
        "grunt": "~0.4.5"
    }

对于旧有项目，为了更新插件版本，可以先删除原来已经安装的插件后重新安装：

	rm -rf node_modules/*
	npm install

# 总结

在支持多层导航栏之后，感觉gitbook更好用了，哈哈！ 

另外很惊喜的发现，新版本的gitbook，已经可以支持热加载了，即启动服务器后，再修改内容，不需要重新启动服务器就可以直接看到新的更新，方便多了。之前hexo支持热加载而gitbook不支持，我就很郁闷，现在终于与时俱进了!

强烈推荐gitbook + markdown + github来写读书笔记。