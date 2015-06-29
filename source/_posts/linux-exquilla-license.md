title: exquilla license 破解
date: 2015-06-29 10:00:41
categories: Linux
tags: [linux,exquilla]
---

资料收集，exquilla license 破解的方式，用于在mint linux下通过Thunderbird + exquilla插件访问outlook收发邮件。

<!--more-->

#  前言

之前在mint linux下通过Thunderbird + exquilla插件访问outlook收发邮件，实验成功。最近打开Thunderbird时发现exquilla插件的免费60天license已经过去。

因此，找了以下网上破解的方式。

# 破解方式

参考这个文章：[ExQuilla 31.0 破解](http://blog.ssfighter.com/2015/01/exquilla-31-0-crack/) 

中间会需要计算字符串的md5值，需要使用到这个工具： [在线 MD5 散列计算器](http://md5calculator.chromefans.org/) 

具体做法，假如我们需要使用以下信息：

- 邮箱：abcd@163.com
- 过期时间：2017-01-18

那么我们就需要计算 MD5(“EX1,abcd@163.com,2017-01-18,356B4B5C”)的值，通过上面的工具我们得到结果是：ff45a465efe7d8f9d9912933f8128c97

因此license最后是这样一个字符串：

> EX1,abcd@163.com,2017-01-18,ff45a465efe7d8f9d9912933f8128c97

将这个作为license提供给exquilla就可以了。




