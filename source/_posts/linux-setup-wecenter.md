title: (记录)在ubuntu上安装wecenter论坛
date: 2015-11-29 14:01:49
categories: Linux
tags: [linux,ubuntu]
---

资料收集，在ubuntu14.04服务器上搭建wecenter论坛。

<!--more-->

#  前言

最近打算做一个小的知识库, 关于cloud native, 最后选择了用wecenter论坛. 因此需要在ubuntu服务器上搭建wecenter论坛.

# 环境准备

## 安装nginx

执行命令：

	sudo apt-get update
	sudo apt-get install nginx

默认nginx安装完成后自动启动，监听80端口，而且开机自启也是默认做好的。

默认配置文件在/etc/nginx/nginx.conf，默认site配置在/etc/nginx/sites-enabled/default。

默认site的文件在/usr/share/nginx/html。

几个常用命令:

1. 重新配置nginx的命令: sudo dpkg-reconfigure nginx
2. 操作nginx服务: sudo service nginx start/stop/restart


## 安装php5

执行以下命令安装php和相关扩展:

	sudo apt-get install php5 php5-common php5-cli php5-fpm php5-mysql php5-gd php5-curl

安装完成后需要修改一个安全配置:

	sudo vi /etc/php5/fpm/php.ini

找到cgi.fix_pathinfo的设置, 取消注释,并将默认的1修改为0:

	cgi.fix_pathinfo=0

重启php5-fpm:

	sudo service php5-fpm restart

## 配置nginx

为enginx服务器开启php支持, 以默认vhost为例:

	sudo vi /etc/nginx/sites-available/default

修改内容如下:

	server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;

        root /usr/share/nginx/html;
        index index.php index.html index.htm;
        server_name _;

        location / {
                try_files $uri $uri/ =404;
        }

        error_page 404 /404.html;
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
                root /usr/share/nginx/html;
        }

        location ~ \.php$ {
                try_files $uri =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass unix:/var/run/php5-fpm.sock;
                fastcgi_index index.php;
                include fastcgi_params;
        }

        location ~ /\.ht {
                deny all;
        }
	}

修改完成后重启nginx:

	sudo service nginx restart

如果重启失败, 可以通过日志文件来检查问题所在, "sudo vi /var/log/nginx/error.log".

创建一个info.php文件来验证php的安装是否ok, 内容为"<?php phpinfo(); ?>".

# wecenter安装

## 下载wecenter

在服务器上下载wecenter, 然后解压缩, 如[安装文档](http://www.wecenter.com/category/help/intall/)所说将UPLOAD下的所有文件上传到/usr/share/nginx/html:

	sudo chmod 777 ./system/ ./system/config

然后访问路径http://www.cloudnative.cn/install/, 开始安装.

## 安装wecenter

第一步, 检查系统, 如无意外应该可以通过了.

第二步, 如果报错说几个目录不存在, 可以执行下面命令, 重新刷新页面:

    sudo mkdir tmp
    sudo mkdir cache
    sudo mkdir uploads
    sudo chmod 777 tmp/ cache/ uploads/

输入对应的数据库账户信息.

第三步: 设置管理员账号

安装过程到是挺轻松的.

# 放弃的准备工作

默认安装的php不支持gd/freetype等, 折腾了半天没有搞定, 最后发现可以直接apt-get安装php-gd/php-curl. 倒! 以下内容没有意义了,只是留着以备以后, 毕竟折腾了几个小时......

## 安装freetype

从[freetype网站](http://www.freetype.org/download.html)下载, 执行下面命令:

	wget http://download.savannah.gnu.org/releases/freetype/freetype-2.4.0.tar.gz
	gunzip freetype-2.4.0.tar.gz
    tar xvf freetype-2.4.0.tar
    cd freetype-2.4.0
    ./configure --prefix=/usr/local/freetype
    make
    sudo make install

## 安装GD

### libpng

从[libpng网站](http://www.libpng.org/pub/png/libpng.html)下载, 执行下面命令:

	wget wget http://downloads.sourceforge.net/project/libpng/libpng16/1.6.19/libpng-1.6.19.tar.gz?r=http%3A%2F%2Fwww.libpng.org%2Fpub%2Fpng%2Flibpng.html&ts=1448777868&use_mirror=iweb
	 mv libpng-1.6.19.tar.gz\?r\=http%3A%2F%2Fwww.libpng.org%2Fpub%2Fpng%2Flibpng.html  libpng.tar.gz
    gunzip libpng.tar.gz
    tar xvf libpng.tar
    cd libpng-1.6.19/
    ./configure --prefix=/usr/local/libpng
    make
    sudo make install

如果make的时候报错"E: Unable to locate package zlib", 则需要按照下面的方式先安装好zlib.

### zlib

从[zlib网站](http://www.zlib.net/)下载, 执行下面命令:

	wget http://zlib.net/zlib-1.2.8.tar.gz
    gunzip zlib-1.2.8.tar.gz
    tar xvf zlib-1.2.8.tar
    cd zlib-1.2.8
    ./configure
    make
    sudo make install

然后回到libpng那边继续:

    ./configure
    make
    sudo make install

### jpegsrc.v6b

从[jpegsrc网站](https://code.google.com/p/quirkysoft/downloads/detail?name=jpegsrc.v6b.tar.gz&can=2&q=)下载, 执行下面命令:

	wget https://quirkysoft.googlecode.com/files/jpegsrc.v6b.tar.gz
    gunzip jpegsrc.v6b.tar.gz
    tar xvf jpegsrc.v6b.tar
    cd jpeg-6b/

由于jpeg安装不能自动创建文件夹，所以要先手工创建:

    sudo mkdir /usr/local/jpeg6
    sudo mkdir /usr/local/jpeg6/include
    sudo mkdir /usr/local/jpeg6/lib
    sudo mkdir /usr/local/jpeg6/bin
    sudo mkdir /usr/local/jpeg6/man
    sudo mkdir /usr/local/jpeg6/man/man1

继续:

	./configure --prefix=/usr/local/jpeg6 --enable-shared --enable-static

执行过程中报错:

> checking host system type... Invalid configuration `x86_64-unknown-linux-gnu': machine `x86_64-unknown' not recognized
> make: ./libtool: Command not found

经查找是,解决方法(参考资料:[64位机器上Centos6.X下安装jpegsrc.v6b.tar.gz时出现错误](http://blog.chinaunix.net/uid-26719405-id-3510258.html)), 先安装libtool:

	sudo apt-get install libtool
    cp /usr/share/libtool/config/config.guess .
    cp /usr/share/libtool/config/config.sub .

然后继续安装jpeg6:

	./configure --prefix=/usr/local/jpeg6 --enable-shared --enable-static
    make
    sudo make install

### gd

从[GD网站](https://libgd.github.io/pages/downloads.html)下载, 执行下面命令:

	wget https://github.com/libgd/libgd/releases/download/gd-2.1.1/libgd-2.1.1.tar.gz
    gunzip libgd-2.1.1.tar.gz
    tar xvf libgd-2.1.1.tar
    cd libgd-2.1.1
    ./configure --prefix=/usr/local/gd --with-jpeg=/usr/local/jpeg6 --with-png=/usr/local/libpng --with-freetype=/usr/local/freetype
    make
    make install

[注:] 后来发现这些都没有搞定,反而是一个非常简单的命令搞定一切,简直无语了......

	sudo apt-get install php5-gd
    sudo apt-get install php5-curl
