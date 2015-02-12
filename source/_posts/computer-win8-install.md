title: (记录)windows8.1安装配置
date: 2015-02-09 11:59:38
categories: 杂谈
tags: [windows]
---

资料收集，关于安装配置windows8.1的TIPS。

<!--more-->

#  设置UEFI + GPT

全新安装Win7或者Win8系统过程中，在选择目标磁盘时，按住“Shift+F10”，在弹出的命令窗口中输入"diskpart"。

具体内容，参考这个文章： [利用diskpart命令创建GPT磁盘分区图文教程](http://www.beihaiting.com/a/XTJC/XTJQ/2014/0815/5127.html)


# 转移用户文件夹

最安全的方式：在安装完WIN8系统后，配置新用户之前，先别急着创建用户，按住“Shift+F10”，在弹出的命令窗口中输入“Regedit”进入注册表编辑器。

依次按以下顺序打开：

> HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList 

特别提醒：这里是“**Windows NT**”。

找到名为 ProfilesDirectory 的字符串值,将 %SystemDrive%\Users 改为你期望的目录,如 D:\Users。

修改完毕后，关闭注册表编辑器，再来创建用户也不迟。

# 修改应用默认安装路径

安装网上方案修改，老是后面安装应用时出错，折腾了挺久。

不浪费时间了，就让应用默认安装在C盘好，大不了将C盘空间放大一点就是。

# 软件设置

## 自动隐藏outlook到托盘

打开outlook后，在右下角托盘中找到outlook的图标，用鼠标左键点击，弹出菜单时点选“最小化时隐藏”。

之后不用outlook的时候，只要点击最小化，就会隐藏到托盘，而不是默认的那种会最小化到任务栏。

