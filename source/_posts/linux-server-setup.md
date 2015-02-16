title: (记录)在ubuntu服务器上搭建全套开发环境
date: 2014-11-25 14:04:50
categories: Linux
tags: [linux,ubuntu,server]
---

资料收集，在ubuntu服务器上搭建全套开发环境。

<!--more-->

#  前言

最近打算更换开发用的服务器，需要将全套开发测试环境搬过去。坦白说这个过程挺麻烦的，记录一下全过程，以备下次使用。

# 服务器配置

在选择linux服务器版本，犹豫了一下，最后还是选择了自己最熟悉的ubuntu server 14.04版本。主要是这些年用apt-get用的比较爽...

开发环境而已，要求不用高，符合自己习惯就好，就不折腾其他高级货色了。

## 增加user

增加自己习惯的user，这个user需要拥有sudo的权限：

	adduser -g sudo sky

## 设置apt源

如果服务器在国内，则可以考虑设置apt源为国内代理，这样速度要好很多。

首先备份源列表:

    sudo cp /etc/apt/sources.list /etc/apt/sources.list_backup

然后修改/etc/apt/sources.list文件，我选择用163的镜像:

    deb http://mirrors.163.com/ubuntu/ trusty main restricted universe multiverse
    deb http://mirrors.163.com/ubuntu/ trusty-security main restricted universe multiverse
    deb http://mirrors.163.com/ubuntu/ trusty-updates main restricted universe multiverse
    deb http://mirrors.163.com/ubuntu/ trusty-proposed main restricted universe multiverse
    deb http://mirrors.163.com/ubuntu/ trusty-backports main restricted universe multiverse
    deb-src http://mirrors.163.com/ubuntu/ trusty main restricted universe multiverse
    deb-src http://mirrors.163.com/ubuntu/ trusty-security main restricted universe multiverse
    deb-src http://mirrors.163.com/ubuntu/ trusty-updates main restricted universe multiverse
    deb-src http://mirrors.163.com/ubuntu/ trusty-proposed main restricted universe multiverse
    deb-src http://mirrors.163.com/ubuntu/ trusty-backports main restricted universe multiverse

记得执行update命令：

	sudo apt-get update

# 开发环境搭建

## Java相关

### 安装JDK

安装jdk7，暂时不上jdk8. 懒得手工安装了，直接apt-get。

	sudo add-apt-repository ppa:webupd8team/java
	sudo apt-get update
	sudo apt-get install oracle-java7-installer
	sudo apt-get install oracle-java7-set-default

[参考地址](http://ubuntuhandbook.org/index.php/2014/02/install-oracle-java-6-7-or-8-ubuntu-14-04/)。

运行java --version的结果：

	java version "1.7.0_72"
	Java(TM) SE Runtime Environment (build 1.7.0_72-b14)
	Java HotSpot(TM) 64-Bit Server VM (build 24.72-b04, mixed mode)

执行 echo $JAVA_HOME看了一下JAVA_HOME的设置：

	/usr/lib/jvm/java-7-oracle

如果手工安装，可以从oracle网站下载安装文件，如jdk-7u75-linux-x64.gz（对于ubuntu不要下载rpm版本）。

手工解压缩到安装路径:

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

## 版本控制

### 安装Git

为了拿到最新版本的git，增加仓库：

	sudo add-apt-repository ppa:git-core/ppa
	sudo apt-get update
	sudo apt-get install git

[参考地址](http://stackoverflow.com/questions/19109542/installing-latest-version-of-git-in-ubuntu)

### 安装gitolite

先增加git的group和user：

	addgroup git
	adduser --ingroup git git

然后用其他账号登录服务器，先准备一份ssh key：
	
	ssh-keygen -t rsa

生成的~/.ssh/id_rsa.pub 文件后面要用到。

然后su 到git用户，执行下面的命令：

	su git
	git clone git://github.com/sitaramc/gitolite
	mkdir -p $HOME/bin
	gitolite/install -to $HOME/bin
	$HOME/bin/gitolite setup -pk path/to/above/id_rsa.pub

输出如下：

	Initialized empty Git repository in /home/git/repositories/gitolite-admin.git/
	Initialized empty Git repository in /home/git/repositories/testing.git/
	WARNING: /home/git/.ssh missing; creating a new one
	    (this is normal on a brand new install)
	WARNING: /home/git/.ssh/authorized_keys missing; creating a new one
	    (this is normal on a brand new install)

退出git账号，回到刚才的账号，试试git clone。

	git clone git@localhost:gitolite-admin.git
	git clone git@localhost:testing.git

不出意外的话应该可以clone下来，说明	gitolite 安装完成。还可以这样检查，执行ssh git@localhost命令：

	ssh git@localhost
	PTY allocation request failed on channel 0
	hello root, this is git@sky1 running gitolite3 v3.6.2-4-g2471e18 on git 2.1.3
	
	 R W    gitolite-admin
	 R W    testing
	Connection to localhost closed.

安装时忘了先创建非root账号了，所以上面看到的用户是root。后来创建了sky账号，需要用root账号将sky的ssh key复制到gitolite-admin/keydir，然后提交。另外需要修改gitolite-admin/conf/gitolite.conf文件，增加sky的仓库访问权限。为了方便起见，建立group admin：

	@admin=root sky
	
	repo gitolite-admin
	    RW+     =   @admin
	
	repo testing
	    RW+     =   @all

### 导入原有git仓库

将原有gitolite下的git 仓库打成tar包，然后传到新机器。解开tar，将tar包中repositories下各个git仓库复制到/home/git/repositories/下(记得gitolite-admin除外)，然后修改gitolite-admin/conf/gitolite.conf，增加各个仓库的对应访问信息即可。

### 安装gitlab

在配置另外一台ubuntu server时，决定试试gitlab，因为功能比gitolite强大。

安装方式参考gitlab官方资料，打开 https://about.gitlab.com/downloads/ 页面，"select operating system"选"ubuntu14.04"，然后按照指示执行安装：

	sudo apt-get install openssh-server
	sudo apt-get install postfix
	wget https://downloads-packages.s3.amazonaws.com/ubuntu-14.04/gitlab_7.7.2-omnibus.5.4.2.ci-1_amd64.deb
	sudo dpkg -i gitlab_7.7.2-omnibus.5.4.2.ci-1_amd64.deb
	sudo gitlab-ctl reconfigure

完成后通过浏览器访问(地址？)默认管理员密码如下：

	Username: root 
	Password: 5iveL!fe

## 打包发布工具

### 安装ant

执行命令：

	sudo apt-get install ant

检查安装之后的版本:

	ant -version
	Apache Ant(TM) version 1.9.3 compiled on April 8 2014

默认没有设置ANT_HOME，还是自己手工加上吧。打开/etc/profile,在最后加入：

	export ANT_HOME=/usr/share/ant

### 安装ivy

理论上从apache ivy的网站下载ivy的jar包放到ant安装目录的lib文件夹下即可。复制时看到ant/lib下的jar包都是link到/usr/share/java下，所以也同样将ivy-2.4.0-rc1.jar复制到/usr/share/java，然后link过去：

	sudo ln -s ../../java/ivy-2.4.0-rc1.jar .

### 安装maven

[参考地址](http://www.sysads.co.uk/2014/05/install-apache-maven-3-2-1-ubuntu-14-04/)，步骤如下：

1. Remove older version
2. Add following lines to sources.list
3. Update Repository and Install
4. Add Symbolic Link

具体执行命令依次如下：

	sudo apt-get remove maven2
	sudo add-apt-repository "deb http://ppa.launchpad.net/natecarlson/maven3/ubuntu precise main"
	sudo apt-get update
	sudo apt-get install maven3
	sudo ln -s /usr/share/maven3/bin/mvn /usr/bin/mvn

### 安装artifactory

从[http://www.jfrog.com/download.php](http://www.jfrog.com/download.php)下载到最新的artifactory，将zip包解压。

安装前需要确保JAVA_HOME有正确设置，可以修改 /etc/environment，加入JAVA_HOME：

	JAVA_HOME=/usr/lib/jvm/java-7-oracle

将目录复制到/usr/lib，执行安装:

	sudo mv artifactory*** /usr/lib/
	sudo mv artifactory*** artifactory
	cd artifactory/bin
	sudo ./installService.sh
	service artifactory check
	sudo service artifactory start

安装成功后就可以通过http://localhost:8081访问artifactory的页面了，默认管理员账号和密码为admin/password。

[参考地址](http://www.softwarepassion.com/install-artifactory-on-ubuntu-box/)。

安全起见，登录后先修改admin密码，点击Admin -> Security -> Users -> User List -> admin，修改密码即可。 

另外，Admin -> Security -> General, 取消"Allow Anonymous Access"。

### 配置maven

为了让mvn能连接到本地的artifactory，需要配置maven。

修改maven的配置文件，全局配置文件在maven3安装路径 /usr/share/maven3/conf/ 下，需要更新server配置信息和profile 配置信息。

server配置段：

	  <servers>
	   <server>
	      <username>****</username>
	      <password>****</password>
	      <id>releases</id>
	    </server>
	    <server>
	      <username>****</username>
	      <password>****</password>
	      <id>snapshots</id>
	    </server>
	  </servers>

profile配置段：

	  <profiles>
	    <profile>
	      <repositories>
	        <repository>
	          <snapshots>
	            <enabled>false</enabled>
	          </snapshots>
	          <id>releases</id>
	          <name>libs-release</name>
	          <url>http://localhost:8081/artifactory/libs-release</url>
	        </repository>
	        <repository>
	          <snapshots />
	          <id>snapshots</id>
	          <name>libs-snapshot</name>
	          <url>http://localhost:8081/artifactory/libs-snapshot</url>
	        </repository>
	      </repositories>
	      <pluginRepositories>
	        <pluginRepository>
	          <snapshots>
	            <enabled>false</enabled>
	          </snapshots>
	          <id>releases</id>
	          <name>plugins-release</name>
	          <url>http://localhost:8081/artifactory/plugins-release</url>
	        </pluginRepository>
	        <pluginRepository>
	          <snapshots />
	          <id>snapshots</id>
	          <name>plugins-snapshot</name>
	          <url>http://localhost:8081/artifactory/plugins-snapshot</url>
	        </pluginRepository>
	      </pluginRepositories>
	      <id>artifactory</id>
	    </profile>
	  </profiles>

	  <activeProfiles>
	    <activeProfile>artifactory</activeProfile>
	  </activeProfiles>

### 安装gradle

为了安装最新的版本，增加gradle的源后再安装：

	sudo add-apt-repository ppa:cwchien/gradle
	sudo apt-get update
	sudo apt-get install gradle

执行gradle -version：
	
	------------------------------------------------------------
	Gradle 2.2
	------------------------------------------------------------
	
	Build time:   2014-11-10 13:31:44 UTC
	Build number: none
	Revision:     aab8521f1fd9a3484cac18123a72bcfdeb7006ec
	
	Groovy:       2.3.6
	Ant:          Apache Ant(TM) version 1.9.3 compiled on December 23 2013
	JVM:          1.7.0_72 (Oracle Corporation 24.72-b04)
	OS:           Linux 2.6.32-042stab090.5 amd64

版本2.2，看了官网最新版本是2.2.1,3天前发布的，估计源还没有来得及更新。

## 持续集成环境

### 安装Jenkins

执行以下命令安装jenkins：

	wget -q -O - https://jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
	sudo sh -c 'echo deb http://pkg.jenkins-ci.org/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
	sudo apt-get update
	sudo apt-get install jenkins

[参考地址](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+Ubuntu)。

安装完成后Jenkins 运行于8080端口。用PS命令可以看到一些Jenkins运行的配置信息，如JENKINS_HOME.

	ps -ef |grep jenkins
	jenkins   7158     1  0 07:50 ?        00:00:00 /usr/bin/daemon --name=jenkins --inherit --env=JENKINS_HOME=/var/lib/jenkins --output=/var/log/jenkins/jenkins.log --pidfile=/var/run/jenkins/jenkins.pid -- /usr/bin/java -Djava.awt.headless=true -jar /usr/share/jenkins/jenkins.war --webroot=/var/cache/jenkins/war --httpPort=8080 --ajp13Port=-1
	jenkins   7159  7158 99 07:50 ?        00:00:24 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/jenkins/jenkins.war --webroot=/var/cache/jenkins/war --httpPort=8080 --ajp13Port=-1

同样安全起见，不能让匿名用户有太多权限。

1. Jenkins -> Configure Global Security -> 启用安全 勾上，访问控制选 Jenkins专用用户数据库（千万记得选上“容许用户注册”！）， 授权策略选 登录用户可以做任何事。
2. 保存之后页面跳转要求登录，这个时候选注册，填写表单后注册成功，用刚注册的用户名登录
3. Jenkins -> Configure Global Security，取消 “容许用户注册”

# 运行环境搭建

虽然是开发环境，好歹搭点东西可以跑个Demo什么的。

## web服务器

### 安装nginx

执行命令：

	sudo apt-get update
	sudo apt-get install nginx

默认nginx安装完成后自动启动，监听80端口，而且开机自启也是默认做好的。

默认配置文件在/etc/nginx/nginx.conf，默认site配置在/etc/nginx/sites-enabled/default。

默认site的文件在/usr/share/nginx/html。

## 数据库服务器

### 安装mongodb

执行以下命令（[参考地址](http://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/)）：

	sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
	echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | sudo tee /etc/apt/sources.list.d/mongodb.list
	sudo apt-get update
	sudo apt-get install -y mongodb-org

安装完成后，看看mongodb的版本，执行mongod --version：

	db version v2.6.5
	2014-11-28T06:50:30.454+0000 git version: e99d4fcb4279c0279796f237aa92fe3b64560bf6

### 安装redis

### 安装mysql

