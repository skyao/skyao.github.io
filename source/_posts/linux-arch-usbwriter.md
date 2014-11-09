title: (记录)用usbwriter来制作archlinux启动U盘
date: 2014-11-09 08:19:17
categories: Linux
tags: [linux,archlinux]
---

解决U盘安装archlinux时遇到的无法从u盘启动的问题。

<!--more-->

##  问题

下载了archlinux 201411的iso安装盘，用UltraISO将iso文件写入U盘，然后用这个U盘启动启动机器准备安装archliunx。

结果启动时报错

	boot/syslinux/shichsys_c32: not a COM32R image

google了一番，找到解决办法。

## 解决方法

改用usbwriter这个小软件(200K)，替代UltraIso来制作usb启动盘，问题解决。

下载地址 [http://sourceforge.net/projects/usbwriter](http://sourceforge.net/projects/usbwriter)，当前版本1.2.

这个软件很小巧，使用也足够简单，强烈推荐！

## 参考资料

1. [https://bbs.archlinux.org/viewtopic.php?id=172250](https://bbs.archlinux.org/viewtopic.php?id=172250)




