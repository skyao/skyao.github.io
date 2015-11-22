title: 将git bare reposotory导入gitlab
date: 2015-11-22 14:28:39
categories: Git
tags: [git,gitlab]
---

记录如何将已有的git bare reposotory导入到gitlab中.

<!--more-->

#  前言

在使用gitlab之前, 我使用gitolite来保存自己私有的代码. 最近发现gitlab的功能远比gitolite要强大, 使用上也方便很多. 因此考虑放弃gitolite, 而原有在gitolite中的代码就需要迁移过去, 同时希望能保留原有的commit/tag/branch等信息.

# 导入方法

发现gitlab对此有特别的支持.

假设我们有三个项目,a.git/b.git/c.git在gitolite中, 导入gitlab的操作步骤如下:

1. 打包原有的bare reposotory

	> cd /home/git/repositories/
    > tar cvf code.tar a.git b.git c.git

2. 在gitlab中为将要导入的项目做准备

	通常gitlab在/var/opt/gitlab/git-data/gitlab-satellites/目录下保存各个项目的bare reposotory.

	gitlab中对于项目存放的方式有两种:

    1. 项目属于某个用户, 比如root/abc.git项目,则保存路径为gitlab-satellites/root/abc.git
    2. 项目属于某个group, 比如group1/cde.git项目, 则保存路径为gitlab-satellites/group1/cde.git

	为了管理方便这里我们将为原来的a/b/c三个项目建立一个group, 在gitlab的管理界面上添加一个名为backup的group.

3. 将打包后的文件传到目标机器上

	然后解压后放到/var/opt/gitlab/git-data/gitlab-satellites/backup目录下,路径类似为gitlab-satellites/backup/a.git.

    执行命令:

    > sudo gitlab-rake gitlab:import:repos

    成功之后就可以在gitlab的页面上看到新导入的git项目.
