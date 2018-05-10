---
title: Docker java教程2—— docker 基础概念
date: 2018-03-15 23:09:42
tags: [docker]
---

**目的: 这个章节主要介绍一些关于Docker 的术语.

    Docker 是一个提供给开发人员以及系统管理员构建，部署和运行应用的平台.
    Docker 令你能够快速地从组件库装配应用,并且在发布代码时排除冲突.
    Docker 使你尽可能快地测试和部署你的代码到生产环境.

— docs.docker.com/

<!--more-->

[主要组件](#1.1)
[Docker 镜像](#1.2)
[Docker 容器](#1.3)
[Docker Engine](#1.4)
[Docker 客户端](#1.5)


Docker 通过更便捷的构建和共享包含你整个应用环境或者应用的操作系统的镜像来简化软件的交付.

**一个应用操作系统意味着什么?**

你的应用一般需要一个特定版本的操作系统,应用服务器,JDK,和数据库服务器,还可能需要调整配置文件,并且同样需要多个其它的依赖.应用可能需要绑定到特定的端口和一定大小的内存.这些组件和配置组成运行你的应用的应用操作系统. 

你可以提供一个将会下载和安装这些组件的安装脚本.同样Docker 允许创建包含你应用及其基础环境的镜像,它就像一个组件一样使用，大大简化了这个过程.这些镜像接着用来创建Docker 容器,Docker 容器运行在Docker提供的虚拟平台.

<h2 id="1.1">主要组件</h2>

Docker 包含三个主要组件:

* 镜像是Docker 的构建组件, 并且是一定了一个应用操作系统的只读模板.

* 容器是Docker的运行组件,它是基于镜像创建的.容器能够运行,启动,停止,移除,或者删除.

* 镜像能够被存储，共享在注册表中进行管理，属于Docker 的分布组件.Docker Store 是一个公共可用的注册表,可用网址为:http://store.docker.com.

为了这三个组件能够正常工作,Docker Daemon(Docker Engine)在一个主机机器上运行,并且完成构建,运行和分发Docker 容器的工作.另外,客户端是一个Docker binary,能够通过Engine响应用户的命令,完成交互.

关键点 1. Docker 架构
Client 客户端与Engine交互可以是在本地同一个主机,也可以是其它任意一个.客户端使用`pull`命令请求Engine来从注册表拉取镜像.Engine接着从Docker Store下载镜像,或者任意一个配置的注册表.从注册表能够下载多个镜像并且安装到Engine上.客户端使用`run`命令来运行容器.

<h2 id="1.2">Docker 镜像</h2>
在Docerk 容器中运行的docker镜像是只读模板.每个镜像由一系列层次构成.Docker利用联合文件系统(Union file systems)来将这些层面结合到一个镜像当中.联合文件系统允许文件或者以目录为单位的独立文件系统,就像分支一样(branches),通过覆盖的方式，形成一个单一而连贯的文件系统.

Docker 如此轻量级的原因之一就是因为这些分层.当你改变一个Docker镜像-比如,更新一个应用到新的版本-一个新的分层被构建.因此,并不是替换整个镜像或者完全地重新构建,就像你可能在虚拟机中做的那样,只有那个分层被添加或者更新.现在你不需要去分配一整个全新的镜像,只需要更新,这使得Docker 镜像的分配更加快速和便捷.

每个镜像都是基于基础镜像(base image)构建的,例如 ubuntu,一个基础的Ubuntu镜像,或者 fedora,一个基础调的Fedora 镜像.你同样能使用自己的镜像作为基础镜像来构建一个新镜像,比如如果你有一个基础的Apache镜像你能将它作为你所有web应用镜像的基础镜像.

|Note|Docker 默认从Docker Store获取这些基础镜像|
|:----:|:----:|

Docker 镜像可以很简单的从这些基础镜像当中构建出来,我们将依次介绍相关指令,每个指令都会在我们的镜像当中创建一个新的分层.指令包含的动作：

* 运行一个命令

* 添加一个文件或者目录

* 创建一个环境变量

* 当容器运行时运行一个进程

这些指令被存储在一个命名为Dockerfile的文件当中.Docker 会在你请求构建一个镜像时读取这个文件,执行这些指令,并且返回最终的镜像.

<h2 id="1.3">Docker 容器</h2>

一个容器由一个操作系统,用户添加的文件，和元数据组成.就像我们看到的那样,每个容器都是从镜像中构建的.那个镜像会告诉Docker容器应该包含什么东西,当容器启动时运行什么进程,和一系列其它配置文件.Docker镜像是只读的.当Docker从一个镜像当中运行容器,它会在那个镜像的顶部添加一个读写层(使用一个联合文件系统),让你的应用能够顺利运行.

<h2 id="1.4">Docker Engine</h2>

Docker Host 作为安装在你机器上的Docker的一部分被创建.一旦你的Docker host已经被创建,它就允许你去管理镜像和容器.例如,这个镜像能够被下载，容器能够被启动,停止和重启.

<h2 id="1.5">Docker 客户端</h2>

这个客户端与Docker Host进行交互并且让你能够对镜像和容器产生影响作用.

通过以下命令检查你的客户端是否正常工作:

    docker -v

它将展示以下输出:

    Docker version 17.09.0-ce-rc3, build 2357fb2

客户端和服务器的版本能够通过`docker version`命令来进行查看.它将展示以下输出:

    Client:
     Version:      17.09.0-ce-rc3
     API version:  1.32
     Go version:   go1.8.3
     Git commit:   2357fb2
     Built:        Thu Sep 21 02:31:18 2017
     OS/Arch:      darwin/amd64
    
    Server:
     Version:      17.09.0-ce-rc3
     API version:  1.32 (minimum version 1.12)
     Go version:   go1.8.3
     Git commit:   2357fb2
     Built:        Thu Sep 21 02:36:52 2017
     OS/Arch:      linux/amd64
     Experimental: true

完整的命令说明能够通过使用`docker --help`命令进行查看.


[官方参考链接](https://github.com/docker/labs/blob/master/developer-tools/java/chapters/ch02-basic-concepts.adoc)