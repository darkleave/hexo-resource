---
title: Docker java教程1—— docker java 环境设置
date: 2018-03-15 23:08:42
tags: [docker]
---


这个章节描述了在这个教程中所需的硬件和软件及其相关的配置方式.

<!--more-->

[硬件 & 软件](#1.1)
[Docker安装](#1.2)
[下载镜像](#1.3)
[其它软件](#1.4)

<h2 id="1.1">硬件 & 软件</h2>

1. 内存: 至少 4 GB+, 推荐 8 GB

2. 操作系统: Mac OS X (10.10.3+), Windows 10 Pro+ 64-bit, Ubuntu 12+, CentOS 7+.

3. Amazon Web Services 凭证需要[以下凭证](https://docs.docker.com/docker-for-aws/iam-permissions/). 这只在这个教程的部分地方需要用到.

|Note|如果使用老版本的操作系统，安装介绍将会有些许不同，这将在下一个章节里讲到.|
|:-:|:-:|

<h2 id="1.2">Docker安装</h2>
Docker在Mac,Windows 和 Linux上运行十分顺畅.这个教程将使用[Docker Community Edition (CE)](https://www.docker.com/community-edition).从[docker Store](https://store.docker.com/search?type=edition&offering=community)下载 Docker CE edition .

Docker CE requires a fairly recent operating system version. If your machine does not meet the requirements, then you need to install Docker Toolbox.

|Note|Docker CE 需要一个相当新的操作系统版本.如果你的机器不满足要求，那你需要安装[Docker Toolbox](https://www.docker.com/products/docker-toolbox).|
|:-:|:-:|


<h2 id="1.3">下载镜像</h2>

这个教程使用到一些Docker 镜像和软件.让我们在开始教程前下载它们.

从 https://raw.githubusercontent.com/docker/labs/master/developer-tools/java/scripts/docker-compose-pull-images.yml 下载文件 并且使用以下命令拉取所需的镜像:

    docker-compose -f docker-compose-pull-images.yml pull --parallel

|Note|对于Linux来说,`docker-compose`和`docker commands`需要`sudo`访问权限.所以在所有命令前加上`sudo`前缀.|
|:-:|:-:|

<h2 id="1.4">其它软件</h2>

这个小节介绍的软件只在教程的某些部分中使用到.如果你计划尝试它们的话再进行安装.

* 安装 [git](https://git-scm.com//).

* 按照[教程](https://docs.docker.com/docker-cloud/installing-cli/)安装 Docker Cloud CLI .

* 下载 Java IDE .
  * [NetBeans 8.2](https://netbeans.org/downloads/) ("Java SE" version)

  * [IntelliJ IDEA Community or Ultimate](https://www.jetbrains.com/idea/download/)

  * [Eclipse IDE for Java EE Developers](http://www.eclipse.org/downloads/eclipse-packages/)

* 下载并且安装 [Maven](https://maven.apache.org/download.cgi).

* 下载通过 [JDK 9 for Linux x64](http://download.java.net/java/GA/jdk9/9/binaries/openjdk-9_linux-x64_bin.tar.gz)构建的OpenJDK. (同样参照 [OpenJDK JDK 9 download page](http://jdk.java.net/9/).)

* 下载通过[JDK 9 for Alpine Linux](http://jdk.java.net/9/ea)构建的先行版本Open JDK.
