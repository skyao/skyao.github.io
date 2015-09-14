title: chrome浏览器降级
date: 2015-09-14 09:34:52
categories: 杂谈
tags: [chrome]
---

公司的内部网站有点问题, 不能支持新版本的chrome和firfox.

但是不幸的是chrome自动升级了,所以需要手工降级回来。

<!--more-->

# 降级步骤

- 删除现有版本的chrome(v45),清理数据

		sudo apt-get remove google-chrome-stable
		rm -rf ~/.config/google-chrome

- 删除google chrome的软件源避免再次自动更新

	在软件源的"额外的仓库"中找到chrome,删掉

- 下载旧版本的chrome(v44)

	[下载地址](http://mirror.pcbeta.com/google/chrome/deb/pool/main/g/google-chrome-stable/)

- 安装旧版本的chrome(v44)

参考资料:

1. https://www.softreadwrite.com/2073/chrome-downgrade-beta-dev-to-stable.html