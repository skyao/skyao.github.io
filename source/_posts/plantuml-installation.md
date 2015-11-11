title: (记录)plantuml安装配置
date: 2014-12-05 12:13:15
categories: Design
tags: [plantuml,uml]
---

记录plantuml的安装配置，以及和其他工具的集成。

<!--more-->

#  安装plantuml

## 安装graphviz

### windows安装

从 [graphviz网站](http://www.graphviz.org/Download_windows.php) 下载到最新的graphviz windows安装版本，如graphviz-2.38.msi。

安装好后，设置环境变量GRAPHVIZ_DOT，指向dot.exe,如 “GRAPHVIZ_DOT=D:\soft\tools\graphviz\bin\dot.exe”.

## 下载plantuml

从 [plantuml网站](http://plantuml.sourceforge.net/download.html) 下载最新的plantuml.jar。理论上只要运行一下命令就可以得到需要的uml输出：

`java -jar plantuml.jar sample.uml`

貌似一般我们不会直接从命令行运行，而是集成到其他工具。

# 集成

## 和IntelliJ集成

从下面地址下载IntelliJ IDEA  的plantuml插件：

`https://plugins.jetbrains.com/plugin/7017?pr=`

然后在IntelliJ 中, "File" -> "Settings" -> plugins, 选择"Install plugin from disk..."，指向刚才下载的zip文件，完成安装之后重启即可。

## 和eclipse集成

参考一下页面：

[http://plantuml.sourceforge.net/eclipse.html](http://plantuml.sourceforge.net/eclipse.html)`

# Chrome插件

## UML Diagram Editor

同事推荐的plantuml的Chrome插件, 试用了一下非常的不错, 强烈推荐.

安装方式: 在chrome的网上应用店中搜索"uml diagram editor"即可找到.
