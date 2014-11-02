title: (资料)linux初始化之upstart VS systemd
date: 2014-11-02 23:41:38
categories: Linux
tags: [linux,systemd,upstart,ubuntu]
---

资料收集，关于linux初始化系统，主要是upstart和systemd。

<!--more-->

##  前言

在配置 ubuntu server 14.04中，无意间发现ubuntu 使用了upstart。翻了一下资料，发现其实ubuntu早在几年前就引入了upstart，只有我很无知的一直只知道init.d,惭愧......

看了一下upstart的信息，发现这个东东的确是比init.d要好，正打算好好研究一下，结果发现一个比upstart更niubility的systemd冒出来了，而且从目前看基本是奔着统一linux世界的目标去的。

upstart被无情碾压，连Ubuntu自己都宣布要服从debian的决定，准备在后续版本中用systemd替换upstart。

刚刚又看到说最新发现的 ubuntu 14.10版本已经在这么做了。

## 资料

收集了一点相关资料，以备以后深入研究

1. 浅析 Linux 初始化 init 系统 （**强烈推荐**）

[浅析 Linux 初始化 init 系统，第 1 部分: sysvinit](http://www.ibm.com/developerworks/cn/linux/1407_liuming_init1/index.html)

[浅析 Linux 初始化 init 系统，第 2 部分: UpStart](http://www.ibm.com/developerworks/cn/linux/1407_liuming_init2/index.html)

[浅析 Linux 初始化 init 系统，第 3 部分: Systemd](http://www.ibm.com/developerworks/cn/linux/1407_liuming_init3/index.html)


