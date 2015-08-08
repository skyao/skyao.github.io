title: (记录)使用mint linux 17.2作为日常操作系统
date: 2015-08-07 20:34:01
categories: Linux
tags: [linux,mint linux]
---

打算使用mint linux 作为平时工作开发的日常操作系统，谨以此文记录在整个搭建和使用过程中的点点滴滴。

后续发生变化或者增加新的内容时，会持续更新此文(最后更新于2015-08-08)。

<!--more-->

#  前言

虽然windows 10很是不错，但是还是想试试用linux做桌面。坦白说这个过程挺麻烦的，记录一下全过程，以备下次使用。

注: 我最后选定的linux发型版本时mint linux 17.2, 因为看评论这个发行版本比较容易上手. 安装过程不介绍了,挺简单,一路轻松安装好.

# 优先设置

安装完成之后, 在安装软件或者设置前,下面的这些操作是务必先执行的.

## 设置软件源并更新

打开 菜单 -> 系统管理 -> 软件源，修改源设置:

- 主要选   nchc
- 基础 aliyun

然后刷新。点右下角的更新管理器图标，"选择全部" -> "安装更新", 直接将系统更新到最新.

## 修改触摸板设置

打开 系统设置 -> 鼠标和触摸板，点触摸板，开启 "打字时禁用触摸板"。实在不能忍受打字时不小心碰到触摸板,然后光标跑不知道哪里去了的感觉....

注：随即发现打字时还是容易被触摸板影响，只好选择关闭。

## 安装搜狗中文输入法

默认不带中文输入法，这个当然不能容忍，第一时间执行以下命令安装搜狗中文输入法：

	sudo add-apt-repository ppa:fcitx-team/nightly  
	sudo aptitude update  
	sudo ptitude install fcitx fcitx-sogoupinyin fcitx-config-gtk fcitx-frontend-all fcitx-module-cloudpinyin fcitx-ui-classic
	
注意：fcitx-sogoupinyin" 会提示无法安装，忽略即可.在搜狗主页下载“搜狗for Linx”, 下载后双击安装.

打开 菜单 -> 首选项 -> 输入法，设置为“fcitx". 重启就OK

注：以上参考 http://jingyan.baidu.com/article/ca00d56c51f399e99eebcfe0.html

# 系统配置

## 硬件相关

### 安装nvidia显卡驱动

打开 系统管理 -> 驱动管理器，可以看到默认时使用备选的驱动。选择列表中最新的版本安装即可，比如我的是346.82。

为了节能，在右下角找到nvidia的图标，设置中找到"select the gpu you would like to use"，默认时NVIDIA，修改为Intel，这样平时用Intel集显比较节能。需要用到nvidia独立显卡时再改回来。

### 设置屏幕亮度

默认屏幕亮度会自动设置到最大亮度, 很明显这个行为对眼睛很不友好...... 

通过功能键或者电源设置, 修改到适当亮度之后, 会发现一旦重启一切又变回来了.

解决的方法参考这个文章 [Ubuntu Does Not Remember Brightness Settings](http://itsfoss.com/ubuntu-mint-brightness-settings/) :

	sudo add-apt-repository ppa:nrbrtx/sysvinit-backlight
	sudo apt-get update
	sudo apt-get install sysvinit-backlight

### 增加对exfat的支持

移动硬盘一般用的文件格式exfat，mint linux默认不支持，需要安装exfat-fuse / exfat-utils

	sudo apt-get install exfat-fuse exfat-utils

重启后生效。

### cpu节能cpufreqd

	sudo add-apt-repository ppa:artfwo/ppa
	sudo apt-get update
	sudo apt-get install cpufreqd

配置中发现cpufreqd 提供了非常灵活的调节方式, 惊为天人! 

为此单独写了一个blog来记录, 详情请参考这里:  [linux下的cpu节能神器cpufreqd](../../08/linux-cpufreqd/)

注: 看到有人推荐indicator-cpufreq, 实际安装后发现不好用,后来选择了 Conky.

### 轻量级系统监控工具Conky

安装命令:

	sudo apt-add-repository -y ppa:teejee2008/ppa
	sudo apt-get update
	sudo apt-get install conky-manager

之后打开conky-manager,选择需要的weget,可以通过编辑weget的属性来调整在桌面的位置. 

### 硬件信息查看工具hardinfo

	sudo apt-get install hardinfo

然后执行 hardinfo 命令就可以在图形界面上看到硬件信息. 

### Intel cpu睿频查看工具i7z

参考这个文章: 

[i7z: Monitor Intel i7, i5 And i3 Frequencies, Multipliers And More Under Linux](http://www.tuicool.com/articles/UnQzie)

安装方式:

	sudo apt-get install i7z
	sudo apt-get install i7z-gui

运行命令: 

	sudo i7z

或者看图形界面:

	gksu i7z_GUI

注: 这个没有温度显示, 所以后面用Psensor了.

### 硬件温度监控工具Psensor

参考这个文章:  [如何在 Ubuntu 中检查笔记本 CPU 的温度](http://www.linuxidc.com/Linux/2015-06/119201.htm).

运行安装命令:

	sudo apt-get install lm-sensors hddtemp
	
检测硬件传感器：

	sudo sensors-detect
	
确保已经工作，运行下面的命令：

	sensors

使用下面的命令安装Psensor：

	sudo apt-get install psensor

之后执行Psensor命令就可以看到各个传感器的温度,包括cpu温度,还可以设置温度报警.

## 系统工具

注: 以下工具大都可以通过apt-get安装, 后来发现通过软件管理器安装更加简单,所以如果没有特别说明后面的软件都是通过软件管理器安装的.

### 应用程序启动器 gnome do

无需多言, 最喜欢的小应用了, 有了它, 打开程序或者文件就太方便了.

gnome do还支持插件, 有几个是非常使用的:

- files and folds: 可以用来指定若干个目录, 然后可以方便的用gnome do 来查找这些目录和它的子目录 (有一个深度参数可以订制子目录层次) 
- chromium: 可以搜索google chrome 浏览器的书签, 这个特别方便
- putty: 可以访问保存的putty session

### 终端 guake 

快捷的终端，安装完之后，记得在开机启动里面增加它的启动项. 

补充: 发现开机自动启动guake也不见得是好事, 还是取消开机启动项吧.第一次用的时候,可以考虑用gnome do启动guake.

# 日常软件

## 工具

### 截图程序 shutter 

直接用软件管理器安装, 另外需要安装几个配套的软件才能使用全面的功能:

- gnome-web-photo: a tool to generate full-size image files and thumbnails from HTML files and web pages 用于抓取整个网页
- libgoo-canvas-perl: canvas widget 用于编辑抓图文件,增加标注等

用了一下勉强还算凑合, 最主要的三个截图方式都支持: 特定区域、窗口、 整个屏幕。

## 系统软件

### windows兼容层 wine 

Microsoft windows compatibility layer 

### vmware workstation

http://down.tech.sina.com.cn/page/3748.html

下载下来的bundle文件，打开文件属性设置为可执行，然后用root账号执行就可开始安装界面。

系列号： 1F04Z-6D111-7Z029-AV0Q4-3AEH8

## 网络软件

### 常用网络工具软件

这几个算是最常用的网络工具软件了: 

- ftp客户端 filezilla  
- curl
- putty: 注意gnome do有putty的插件,可以非常方便的打开putty保存的session

### google chrome 浏览器

在[以下地址](http://www.google.cn/chrome/browser/thankyou.html?platform=linux)下载适合的64位debian dev 版本，然后直接安装即可

或者直接用这个下载链接:

https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb


# 参考资料

期间陆陆续续百度/谷歌, 部分信息参考自以下文章:

- [安装Linux Mint 17后要做的20件事](http://linux.it.net.cn/e/Linuxit/2014/0710/2825.html)
- [多快好省：10个技巧加速你的LinuxMint/Ubuntu](http://www.mintos.org/config/speedup-mint.html)
- [查看linux系统CPU信息的经验](http://jingyan.baidu.com/article/6525d4b150b472ac7d2e94a6.html)
- [如何在 Ubuntu 中检查笔记本 CPU 的温度](http://www.linuxidc.com/Linux/2015-06/119201.htm)


