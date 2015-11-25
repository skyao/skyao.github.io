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

### 开机自动装载windows分区

更新时间: 2015-10-30

linux mint安装之后自带了ntfs-3g, 天然支持ntfs格式, 默认情况在开机的时候就会在桌面出现两个文件(我的windows C盘和D盘), 双击之后就可以安装windows分区. 虽然还是需要自己动手双击一下, 但是至少还算方便, 就这样一直用着.

某天, 貌似是做了一个大的更新之后, 突然上述功能没有了, 导致访问windows分区非常不方便. 搞不懂出了什么状况, 只好自己动手. 参考[网上找到的文章](http://blog.csdn.net/cjw517/article/details/42586213).

在开机启动程序中增加两个开机启动的命令, 内容为下面这两个命令, 分别装载两个ntfs分区:

    udisksctl mount -p block_devices/sdb4
    udisksctl mount -p block_devices/sdb5

终于又可以愉快的访问windows了......

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

## 网络设置

### 设置DNS

设置阿里的DNS，提高网络响应速度, 实测效果明显.

打开终端执行:

	sudo vim /etc/resolvconf/resolv.conf.d/tail

添加下面两行Alidns：

    nameserver 223.5.5.5
    nameserver 223.6.6.6

保存后执行:

	sudo resolvconf -u

查看/etc/resolv.conf可以看到多了上面两条nameserver.

# 日常软件

## 工具

### 截图程序 shutter

直接用软件管理器安装, 另外需要安装几个配套的软件才能使用全面的功能:

- gnome-web-photo: a tool to generate full-size image files and thumbnails from HTML files and web pages 用于抓取整个网页
- libgoo-canvas-perl: canvas widget 用于编辑抓图文件,增加标注等

用了一下勉强还算凑合, 最主要的三个截图方式都支持: 特定区域、窗口、 整个屏幕。

### 翻译软件 有道词典

从[有道词典官网下载](http://cidian.youdao.com/index-linux.html) 到deb安装包, 直接安装即可.

安装过程中需要下载众多依赖包, 请耐心等待完成.

注意: 不要安装deepin的版本, 在下面有for ubuntu的版本.

## 系统软件

### windows兼容层 wine

wine是Microsoft windows compatibility layer,让我们可以在linux上跑起来一些windows下的软件,对于某些只有windows版本的软件也是一种选择.

用mint linux的软件管理器, 搜索安装以下内容: 

- wine
- wine mono: microsoft .net framework 支持
- wine gecho
- playonelinux
- ttf-mscorefonts-installer: PlayOnLinux 第一次开启时会建议安装这个包

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

### 使用终端做ssh client

发现putty和Remmina做ssh客户端都不是太好用, 远不如windows平台上的securyCrt和putty. 后来看到大家一般都推荐直接用linux的终端做sshclient, 简单敲个"ssh server_name"就连上去了.

为了减少每次敲击密码的麻烦, 还可以通过authorized_keys的方式来登录.

1. 上传本机的.ssh/id_isa.pub文件到服务器端
2. 在远程服务器上运行

	> cd .ssh
	> cat ~/id_rsa.pub >> authorized_keys

3. 在本机终端中输入"ssh server_address"即可直接登录远程服务器

### google chrome 浏览器

在[以下地址](http://www.google.cn/chrome/browser/thankyou.html?platform=linux)下载适合的64位debian dev 版本，然后直接安装即可

或者直接用这个下载链接:

https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb

### 翻墙软件 shadowsocks

之前有单独的 blog 谈到这个话题: [使用shadowsocks实现科学上网](../../../../2015/07/14/computer-fanqiang/), linux上我继续使用 shadowsocks.

### SSH快捷翻墙

在本地执行以下命令

> ssh -D 10085 server_address

然后就可以在浏览器中设置代理服务器连接为socket4/127.0.0.1/10085端口, 就可以做代理上网. 如果远程服务器在国外, 就是翻墙了. 原理和用putty设置dynamic是一样的.

### chrome插件SwitchyOmega

在google chrome中安装SwitchyOmega 插件,然后加入场景设置使用本地shadowsocks作为代理.

为了方便, SwitchyOmega 中设置自动切换,规则列表设置格式为AutoProxy,规则列表网址为下面地址,保存后更新:

https://autoproxy-gfwlist.googlecode.com/svn/trunk/gfwlist.txt

这样用chrome上网就可以自动切换代理,需要翻墙时自动连shadowsocks,不需要时直接走.

### 远程桌面软件 Remmina

需要使用软件管理器安装remmina和几个插件:

- remmina
- remmina-common
- remmmina-plugin-gnome
- remmina-plugin-rdp: 这个一定要安装,连接windows桌面就是走rdp协议
- remmina-plugin-vnc

安装完成之后,打开remmina, "connection" -> "new", Protocol 选 "RDP - Remote Desktop Protocol", 设置链接参数和账号,就可以连接到windows桌面.

## 编辑器

### 文本编辑器VIM

直接用软件管理器安装Vim / Vim-gnome.

但是郁闷的是操作系统中找不到安装好的vim,只能通过命令行的方式来启动vim.

### 文本编辑器sublime text 3

从[官网下载](http://www.sublimetext.com/3) 对应的deb包直接安装即可.

但是发现安装完成之后找不到程序, 只能命令行启动.

卸载后用软件管理器, 发现没有这个问题. 因此推荐使用软件管理器安装, 安装完成后系统和gnome do都可以识别.

### markdown编辑器cutemarked

详细见单独的blog: [推荐linux下的markdown编辑器cutemarked](../../../../2015/08/07/cutemarked-install/)

补充: 最后由于无法在cutemarked中使用搜狗输入法,不得已替换为haroopad.

### markdown编辑器 Haroopad

从 http://pad.haroopress.com/ 下载, 比较奇怪的是下载速度奇慢无比, 只有10k-20k的速度, 好不容易才下载好.

安装简单,直接运行下载好的 haroopad-v0.13.1-x64.deb 就可以了.

简单试用了一下,这个软件还是比较好用的,不错的markdown编辑器,以后就用它了.

## 影音娱乐

虽然是在linux下,但是我们也不能放弃音乐对不对?

### 网易云音乐

我最喜欢的在线音乐播放器了, 但是很遗憾的发现没有linux版本......

只能想其他办法, 试着用wine安装了,结果打开时整个软件窗口都是黑的, 倒,估计时缺少某些包. 换成playonlinux,在默认的default虚拟盘上安装了两个包(mono/gecho)之后,就可以顺利安装网易云音乐了.

试用了一下, 功能都正常,只是音量大小调节无效,只能通过linux系统来调节音量,其他都OK.

致命问题: 

1. 似乎音质一般, 猜测可能是跑在wine上?
2. 居然会卡......

看来还是需要linux原生的音乐播放器.

### 酷我音乐linux客户端

baidu时发现的好东东, 没有想到linux下居然有这么好的在线音乐播放器, 真是喜出望外![]()

[网站在这里](https://github.com/LiuLang/kwplayer), 这是它的介绍: kwplayer是一款运行在linux桌面的网络音乐播放器, 它使用 酷我音乐盒 的音乐资源.

[下载在这里](https://github.com/LiuLang/kwplayer-packages),建议在页面右边直接下载整个zip包.

然后按照要求的顺序先后安装下面三个deb文件:

- python3-xlib
- python3-keybinder
- kwplayer

# 编程开发软件

## 工具软件

### Git

直接apt-get:

	sudo add-apt-repository ppa:git-core/ppa
	sudo apt-get update
	sudo apt-get install git

## Java相关

### JDK

安装jdk7，直接apt-get:

	sudo add-apt-repository ppa:webupd8team/java
	sudo apt-get update
	sudo apt-get install oracle-java7-installer
	sudo apt-get install oracle-java7-set-default

之前都是这么搞定的, 但是今天遇到问题, 卡在这里无法继续:

Resolving edelivery.oracle.com (edelivery.oracle.com)... 104.90.218.20
Connecting to edelivery.oracle.com (edelivery.oracle.com)|104.90.218.20|:443... connected.

只好放弃,手工安装JDK:

	gunzip jdk-7u75-linux-x64.gz
	gunzip jdk-7u75-linux-x64.gz
	sudo mkdir /usr/lib/jvm
	sudo mv jdk-7u75-linux-x64 /usr/lib/jvm/jdk7

修改/etc/profile文件，在最后加入以下内容：

	# java
	export JAVA_HOME=/usr/lib/jvm/jdk7
	export PATH=$JAVA_HOME/bin:$PATH

安装好后检验一下：

	source /etc/profile
	java -version

### java打包工具

以下工具的安装参考之前的blog[在ubuntu服务器上搭建全套开发环境](../../../../2015/07/03/linux-server-setup/#打包发布工具),安装方式是一样的.

- ant
- maven
- gradle

### IDE

- intellij idea

	从官网下载linux 版本例如 ideaIU-14.1.4.tar.gz, 解压后 然后依照 Install-Linux-tar.txt 的提示运行 idea.sh 就安装好了.

	安装过程中记得选择"create desktop entry", 这样可以让操作系统知道有安装intellij idea并创建快捷方式. 

	也可以在安装完成后通过菜单项 Tools -> "create desktop entry" 来处理.

- webstorm

	安装方式和intellij idea 一样.

- eclipse

	从[eclipse 官网下载](http://www.eclipse.org/downloads/) for linux 64 的版本如 eclipse-jee-mars-R-linux-gtk-x86_64.tar.gz.

	解压即可直接使用,为了方便我们需要为eclipse 增加desktop entry,执行命令:

		sudo gedit /usr/share/applications/eclipse.desktop

	输入以下内容并保存:

		[Desktop Entry]
		Name=eclipse
		Name[zh_CN]=Eclipse
		Comment=eclipse Client
		Exec=/home/sky/work/soft/java/eclipse/eclipse
		Icon=/home/sky/work/soft/java/eclipse/icon.xpm
		Terminal=false
		Type=Application
		Categories=Application;Development;
		Encoding=UTF-8
		StartupNotify=true

	备注: 暂时还没有找到方法可以让gnome do这样的应用启动工具可以支持eclipse.

# 参考资料

期间陆陆续续百度/谷歌, 部分信息参考自以下文章:

- [安装Linux Mint 17后要做的20件事](http://linux.it.net.cn/e/Linuxit/2014/0710/2825.html)
- [多快好省：10个技巧加速你的LinuxMint/Ubuntu](http://www.mintos.org/config/speedup-mint.html)
- [查看linux系统CPU信息的经验](http://jingyan.baidu.com/article/6525d4b150b472ac7d2e94a6.html)
- [如何在 Ubuntu 中检查笔记本 CPU 的温度](http://www.linuxidc.com/Linux/2015-06/119201.htm)
-[Ubuntu常用软件合集](http://www.lvzejun.cn/2015/03/31/ubuntu-software/)

