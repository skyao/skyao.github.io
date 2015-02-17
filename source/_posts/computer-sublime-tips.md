title: sublime 3 tips
date: 2015-02-17 23:39:19
categories: 杂谈
tags: [sublime,computer]
---

Sublime 3 使用中的一些TIPS。

# 需要解决的问题

## 中文乱码

步骤：

在Sublime Text里，按ctrl+`，打开Console，一次性输入如下代码：

> import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())

这样Sublime Text就会安装我们需要的Package Control。否则后面会找不到Package。

重启Sublime Text。

在Sublime Text中，按Ctrl+Shift+P打开命令行模式，输入Install Package关键字，然后点击自动出现的下拉菜单里的第一项：Package Control: Install Package。

此时你会看到左下角有个=号来回动，稍等一会，会再次在命令行下弹出一个下拉菜单。输入“ConvertToUTF8”或者“GBK Encoding Support”，选择匹配项。

中文字符就可以正常显示了。
