title: (记录)配置Jenkins服务器
date: 2014-11-27 16:22:27
categories: CI
tags: [CI,jenkins]
---

资料收集，在ubuntu服务器上搭建并配置Jenkins CI服务器。

Last Update: 2015-11-26

<!--more-->

#  安装Jenkins

在上一个Blog中已经介绍了如何在ubuntu服务器上安装Jenkins并配置用户登录。


# 安装插件

## 取消不需要的已安装插件

以下插件暂时用不上，先取消，万一需要了再启用不迟：

1. CVS plugin
2. LDAP plugin
3. PAM Authentication Plugin
4. Subversion Plugin
5. Translation Assistance Plugin
6. Windows Slaves Plugin

Update: 在最新的Jenkins版本中, 这些plugin不容许被取消, 只好继续保留.

## 安装新插件

需要安装以下插件：

### git相关

1. Git Client Plugin
2. Git Parameter Plugin

### view 排版

1. Nested View Plugin
2. Sectioned View Plugin

### 运行状态

1. Green Balls


# 配置Jenkins

## 修改系统配置

登录后，系统管理 -> 系统设置。

在这里配置好JDK/ant/maven/git, 注意必须配置, 否则跑Job的时候会出错.

## 准备git

需要为jenkins用户准备git访问权限：

	sudo su jenkins
	ssh-keygen -t rsa

将得到的pub key提交到gitlab/github的ssh key中.

## 准备maven

### 配置artifactory

在 artifactory 中增加一个新的jenkins 用户，设置权限为可以读和upload。

Update: 在新版本的artifactory中, 默认permission里面居然没有带upload权限的permission, 需要自己先创建一个permission, 选择带有read和upload权限,然后再关联到group或用户.

如果没有upload权限, 会报"Access denied"错误:

> [ERROR] Failed to execute goal org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy (default-deploy) on project ut-cidemo: Failed to deploy artifacts: Could not transfer artifact io.xuexi.ut:ut-cidemo:jar:0.1.0-20151126.090302-1 from/to snapshots (http://localhost:8081/artifactory/libs-snapshot-local): Access denied to: http://localhost:8081/artifactory/libs-snapshot-local/io/xuexi/ut/ut-cidemo/0.1.0-SNAPSHOT/ut-cidemo-0.1.0-20151126.090302-1.jar, ReasonPhrase: Forbidden. -> [Help 1]


### 配置maven

修改maven的配置文件，server配置段：

```xml
<servers>
    <server>
      <username>jenkins</username>
      <password>****</password>
      <id>releases</id>
    </server>
    <server>
      <username>jenkins</username>
      <password>****</password>
      <id>snapshots</id>
    </server>
</servers>
```

然后试着su 到 jenkins用户，试找打包代码和发布到artifactory，如果一切顺利，说明jenkins用户的git/maven/artifactory都可以正常使用。这样就可以去jenkins上创建相应的job了。

### 常见问题

配置后，在deploy打包好的jar包到artifactory时遇到两个问题，解决后才反应过来，原来N年前配置的时候也遇到过......

真是无语，所以还是记录下来避免日后再浪费时间。


1. 报错为"repository element was not specified in the POM inside distributionManagement element or in -DaltDeploymentRepository=id::layout::url parameter"

	这个报错的原因是pom.xml里面没有distributionManagement配置，这个是用来告诉maven要deploy打包之后的文件到artifactory的哪个仓库，没有这个信息artifactory无法工作。我不大喜欢在pom.xml文件中制定，因为可能在不同的机器上发布的artifactory是不一样的。所以一般用-DaltDeploymentRepository参数来制定。

2. 报错为"Return code is: 405, ReasonPhrase: Method Not Allowed."

	这个错误有点低级了，前面maven的settings.xml文件里面有一个release(或者snapshort)的仓库地址，如 http://localhost:8081/artifactory/libs-release 。 但是这个仓库是一个虚拟仓库，是不能直接deploy到这里的。应该修改为 http://localhost:8081/artifactory/libs-release-local 。 这个libs-release-local仓库才是满足我们要求的仓库。

下面是一个执行deploy的maven命令：

	mvn -DaltDeploymentRepository=releases::default::http://localhost:8081/artifactory/libs-release-local -DperformRelease=true -Dgpg.skip -DskipTests deploy

### id的对应关系

有三个地方的id是必须一一对应的:

1. profile设置中repository的id

    ```xml
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
    </profile>
    ```
2. server设置中的id

    ```xml
    <servers>
    <server>
      <username>jenkins</username>
      <password>****</password>
      <id>releases</id>
    </server>
    <server>
      <username>jenkins</username>
      <password>****</password>
      <id>snapshots</id>
    </server>
    </servers>
    ```

3. mvn命令行参数"-DaltDeploymentRepository=id::layout::url"中的id

	> mvn -DaltDeploymentRepository=releases::default::http://localhost:8081/artifactory/libs-release-local