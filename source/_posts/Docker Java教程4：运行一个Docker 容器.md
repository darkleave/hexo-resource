---
title: Docker Java教程4：运行一个Docker 容器
date: 2018-05-10 23:09:42
tags: [docker]
---


使用Docker运行一个应用的方式就是运行一个容器.如果你考虑使用一个开源的软件,那么有很大的可能Docker Store 已经有对应的Docker 镜像了.Docker 客户端能够通过镜像名称轻松地运行这个容器.客户端首先会检查Docker Host是否存在这个镜像,如果存在，则直接运行容器，否则host会首先去下载镜像.

<!--more-->
[拉取镜像](#1)
[运行容器](#2)
[交互式地](#2.1)
[隔离的容器](#2.2)
[使用默认端口](#2.3)
[使用指定的端口](#2.4)
[部署一个war文件到应用服务器](#2.5)
[停止容器](#3)
[移除容器](#4)
[端口映射的额外方式](#5)

<h2 id="1">拉取镜像</h2>

查看可用镜像:

    docker image ls

输出如下:

    REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
    hellojava                    latest              8d76bf5691c4        32 minutes ago      740MB
    hello-java                   latest              93b1180c5d91        36 minutes ago      740MB
    helloworld                   2                   7fbedda27c66        41 minutes ago      1.13MB


更多关于镜像的细节可以使用`docker image history jboss/wildfly`命令:

    IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
    9adbdb00cded        8 days ago          /bin/sh -c #(nop)  CMD ["/opt/jboss/wildfl...   0B
    <missing>           8 days ago          /bin/sh -c #(nop)  EXPOSE 8080/tcp              0B
    <missing>           8 days ago          /bin/sh -c #(nop)  USER [jboss]                 0B
    <missing>           8 days ago          /bin/sh -c #(nop)  ENV LAUNCH_JBOSS_IN_BAC...   0B

<h2 id="2">运行容器</h2>

<h3 id="2.1">交互式地</h3>

以交互模式运行WildFly 容器.

    docker container run -it jboss/wildfly

输出如下:

    =========================================================================
    
      JBoss Bootstrap Environment
    
      JBOSS_HOME: /opt/jboss/wildfly
    
      JAVA: /usr/lib/jvm/java/bin/java
    
    . . .
    
    00:26:27,455 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0060: Http management interface listening on http://127.0.0.1:9990/management
    00:26:27,456 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0051: Admin console listening on http://127.0.0.1:9990
    00:26:27,457 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0025: WildFly Full 10.1.0.Final (WildFly Core 2.2.0.Final) started in 3796ms - Started 331 of 577 services (393 services are lazy, passive or on-demand)


这个输出说明服务器已经正常启动了,皮卡丘!

默认，Docker会运行在前台.`-i` 允许与STDIN(standard input,stdin是标准输入，一般指键盘输入到缓冲区里的东西)进行交互,`-t` 附加一个 TTY(通常使用tty来简称各种类型的终端设备terminal) 到进程. 两个开关能够合并为`-it`.

点击 Ctrl+C 来停止容器.

<h3 id="2.2">隔离的容器</h3>

以detached(隔离)模式启动容器:

    docker container run -d jboss/wildfly
    254418caddb1e260e8489f872f51af4422bc4801d17746967d9777f565714600

这次使用`-d`来替换`-it`,以隔离模式来运行容器.

输出是一个唯一的id 分配给容器.通过`docker container logs <CONTAINER_ID>`命令可以查看容器的日志记录,`<CONTAINER_ID>` 代表容器的id.

`docker container ls` 命令能够查看容器的状态:

    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
    254418caddb1        jboss/wildfly       "/opt/jboss/wildfl..."   2 minutes ago       Up 2 minutes        8080/tcp            gifted_haibt

`docker container ls -a`命令能够查看主机上的所有容器.

<h3 id="2.3">使用默认端口</h3>

如果你想要容器接受请求链接,那么你需要在调用`docker run`命令时提供特殊的参数选项.否则，启动的容器无法被我们的浏览器所访问.我们需要再次停止它并且以不同的选项重新启动.

    docker container stop `docker container ps | grep wildfly | awk '{print $1}'`

以如下命令重启容器:

    docker container run -d -P --name wildfly jboss/wildfly

`-P` 代表在Docker host 上暴露 对应的端口.另外,`--name`一般用来给予这个容器一个名称.这个名称可以用来查询更多关于容器的信息或作为一个id来停止它.通过`docker container ls`命令来验证容器名称:

    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                     NAMES
    89fbfbceeb56        jboss/wildfly       "/opt/jboss/wildfl..."   9 seconds ago       Up 8 seconds        0.0.0.0:32768->8080/tcp   wildfly

端口映射显示在 PORTS列.通过http://localhost:32768地址防卫 WildFly服务器.确保使用正确的端口来进行访问.

页面展示如下:

【我是图片】

<h3 id="2.4">使用指定的端口</h3>

停止并且移除最近运行的容器:

    docker container stop wildfly
    docker container rm wildfly

也可以用`docker container rm -f wildfly`命令来同时完成停止和移除容器的工作.需要谨慎小心，因为`-f`使用`SIGKILL`来kill容器.

重启容器:

    docker container run -d -p 8080:8080 --name wildfly jboss/wildfly

这种格式表示`-p hostPort:containerPort`.这允许我们访问主机上容器的特定端口.

现在通过http://localhost:8080 链接来测试.确认运行正常，接着如下同样停止并移除容器.

<h3 id="2.5">部署一个war文件到应用服务器</h3>

现在你的应用服务器正在运行,接下来展示如何部署war包到上面.

创建一个新目录`hellojavaee`.用如下内容创建`Dockerfile`文件:

    FROM jboss/wildfly:latest

    RUN curl -L https://github.com/javaee-samples/javaee7-simple-sample/releases/download/v1.10/javaee7-simple-sample-1.10.war -o /opt/jboss/wildfly/standalone/deployments/javaee-simple-sample.war

创建镜像:

    docker image build -t javaee-sample .

启动容器:

    docker container run -d -p 8080:8080 --name wildfly javaee-sample

访问网站路由：

    curl http://localhost:8080/javaee-simple-sample/resources/persons

输出如下:
    
    <persons>
    	<person>
    		<name>
    		Penny
    		</name>
    	</person>
    </persons>

可选的:`brew install XML-Coreutils`命令将会在Mac上安装xml格式的工具.这个输出能够通过`xml-fmt`选项来展示格式化结果.

<h2 id="3">停止容器</h2>

通过容器id或者名称来停止容器:

    docker container stop <CONTAINER ID>
    docker container stop <NAME>

停止所有正在运行的容器:

    docker container stop $(docker container ps -q)

只停止已经退出的容器:

    docker container ps -a -f "exited=-1"

<h2 id="4" >移除容器</h2>

通过id或者名称移除指定的容器:

    docker container rm <CONTAINER_ID>
    docker container rm <NAME>
    
通过表达式匹配来移除容器:

    docker container ps -a | grep wildfly | awk '{print $1}' | xargs docker container rm

无条件移除所有容器:

    docker container rm $(docker container ps -aq)

<h2 id="5">端口映射的额外方式</h2>

确切的端口映射能够通过docker port命令进行设置:

    docker container port <CONTAINER_ID> or <NAME>

输出如下:

    8080/tcp -> 0.0.0.0:8080

端口映射情况能够通过docker inspect命令进行查看:

    docker container inspect --format='{{(index (index .NetworkSettings.Ports "8080/tcp") 0).HostPort}}' <CONTAINER ID>

 

[官网链接](https://github.com/docker/labs/blob/master/developer-tools/java/chapters/ch04-run-container.adoc)