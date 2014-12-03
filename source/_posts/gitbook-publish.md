title: 轻松将gitbook内容发布到github pages上
date: 2014-04-22 23:45:19
categories: 杂谈
tags: [gitbook,grunt]
---

总结并分享一下将gitbook生成的html page发布github pages上的简便方法。

<!--more-->

在将gitbook生成的html page发布github pages上(实际是提交内容到gh-pages 分支)时，发现比较麻烦。

而gitbook的文档和资料都实在太少，好在还有万能的stackoverflow，顺利找到解决方法。

## 问题 ##

具体内容请参考一下页面，我遇到的问题和提问的人完全一致：

[http://stackoverflow.com/questions/23094647/difficulty-in-getting-gitbook-site-to-show-up-in-github-page](http://stackoverflow.com/questions/23094647/difficulty-in-getting-gitbook-site-to-show-up-in-github-page)

## 解决方案 ##

解决方案就是如答复中所说使用grunt来处理这个问题:

> Take a look at this book: https://github.com/GitbookIO/git.

> It uses grunt to set up a few tasks: test, publish and build.

## 具体方式 ##

1. 安装grunt

	`npm install -g grunt`

2. 安装grunt-cli，否则执行grunt命令时会报错说"command not found"

	`npm install -g grunt-cli`

3. 复制文件package.json, Gruntfile.js到当前book的根目录下

	Grentfile.js文件需要根据实际数的情况稍作改动：

> 	grunt.initConfig({
>         'gitbook': {
>             development: {
>                 input: "./",
>                 title: "Action In Java Performance Tuning",
>                 description: "The Java Performance Tuning book.",
>                 github: "skyao/action-in-java-performance-tuning"
>             }
>         }

可以修改port来改变本地预览时的端口号：

>         'http-server': {
>             'dev': {
>                 // the server root directory
>                 root: '_book',
> 
>                 port: 4000,
>                 host: "127.0.0.1",

package.json文件也需要做一些小的改动，主要是提供一些必要信息如git仓库：

>     "name": "Action-In-Java-Performance-Tuning",
>     "version": "0.0.1",
>     "description": "the Java Performance Tuning book",
>     "repository": {
>         "type": "git",
>         "url": "https://github.com/skyao/action-in-java-performance-tuning"
>     },
>     "author": "Sky Ao <aoxiaojian@gmail.com>",

4. 执行npm install

	`npm install .`

3. 测试时可以直接执行

    `grunt test`

	这样可以直接在浏览器中预览到当前的最新内容。记得如果上述命令报错，"Fatal error: Unable to find local grunt."，说明是上一步npm install没有执行。

4. 发布内容，执行

    `grunt publish`

	可以看到如下的日志：

> $ grunt publish
> 
> Running "gitbook:development" (gitbook) task
>
> Running "gh-pages:src" (gh-pages) task
> 
> Cloning git@github.com:skyao/java-performance-tuning.git into .grunt\grunt-gh-pages\gh-pages\src
>
> Cleaning
> 
> Fetching origin
> 
> Checking out origin/gh-pages
> 
> Removing files
> 
> Copying files
> 
> Adding all
> 
> Committing
> 
> Pushing
> 
> Running "clean:files" (clean) task
> 
> Cleaning _book...OK
> 
> Done, without errors.

使用grunt之后，发布内容就方便多了。

