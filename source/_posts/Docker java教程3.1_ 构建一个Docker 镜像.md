---
title: "Docker java教程3.1: 构建一个Docker 镜像"
date: 2018-05-10 23:09:42
tags: [docker]
---


[Dockerfile](#1.1)
[创建你的首个镜像](#1.2)
[使用java创建你的首个镜像](#1.3)

* [创建一个普通的java应用](#1.3.1)
* [作为Docker镜像打包和运行Java应用](#1.3.2)
* [使用Docker Maven Plugin 打包和运行Java应用](#1.3.3)

[Dockerfile 命令设计模式](#1.4)

* [CMD 和 ENTRYPOINT 之间的不同](#1.4.1)
* [ADD 和 COPY 之间的不同](#1.4.2)
* [导入和导出镜像](#1.4.3)

<!--more-->



<h2 id ="1.1">Dockerfile</h2>

Docker 构建镜像是通过读取Dockerfile 文件的指令.Dockerfile 是一个包含了所有用户能在命令行执行用以装配镜像的命令的文本文件.`docker image build`命令使用这个文件并且成功执行所有的命令来创建镜像.

`build` 命令在镜像创建期间同样传递一个上下文.这个上下文可以是你本地文件系统的路径或者一个git仓库的URL地址.

**Dockerfile is usually called Dockerfile.** 
完全的命令一览查看以下链接  https://docs.docker.com/reference/builder/. 常见命令见下文:

Table 1. Dockerfile 常见命令

|Command|	Purpose|	Example|
|:----:|:----:|:----:|
|FROM|First non-comment instruction in Dockerfile|`FROM ubuntu`|
|COPY|Copies mulitple source files from the context to the file system of the container at the specified path|`COPY .bash_profile /home`|
|ENV|Sets the environment variable|`ENV HOSTNAME=test`|
|RUN|Executes a command|`RUN apt-get update`|
|CMD|Defaults for an executing container|`CMD ["/bin/echo", "hello world"]`|
|EXPOSE|Informs the network ports that the container will listen on|`EXPOSE 8093`|

<h2 id ="1.2">创建你的首个镜像</h2>

创建一个新目录 `hellodocker`.

在这个目录,创建一个新的Dockerfile 文件.使用以下内容:

    FROM ubuntu:latest
    
    CMD ["/bin/echo", "hello world"]

这个镜像使用 `ubuntu` 作为基础镜像.`CMD` 命令定义需要运行的命令.
它提供了一个不同的入口 `/bin/echo` 并且提供参数 “hello world”.

使用以下命令构建镜像:    
    docker image build . -t helloworld

`.` 在这个命令当中是作为 `docker image build` 的上下文. `-t` 添加标志到镜像当中.

输出如下:

    Sending build context to Docker daemon  2.048kB
    Step 1/2 : FROM ubuntu:latest
    latest: Pulling from library/ubuntu
    9fb6c798fa41: Pull complete
    3b61febd4aef: Pull complete
    9d99b9777eb0: Pull complete
    d010c8cf75d7: Pull complete
    7fac07fb303e: Pull complete
    Digest: sha256:31371c117d65387be2640b8254464102c36c4e23d2abe1f6f4667e47716483f1
    Status: Downloaded newer image for ubuntu:latest
     ---> 2d696327ab2e
    Step 2/2 : CMD /bin/echo hello world
     ---> Running in 9356a508590c
     ---> e61f88f3a0f7
    Removing intermediate container 9356a508590c
    Successfully built e61f88f3a0f7
    Successfully tagged helloworld:latest

使用`docker image ls`命令展示可用的镜像:
    
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    helloworld          latest              e61f88f3a0f7        3 minutes ago       122MB
    ubuntu              latest              2d696327ab2e        4 days ago          122MB


使用以下命令运行容器:    

    docker container run helloworld

命令输出如下:

    hello world

如果你没有看到期望的输出,检查你的Dockerfille文件确保内容与上文所示一致.再次构建镜像并且运行它.

在`Dockerfile`文件中改变基础镜像，从`ubuntu` 修改为`busybox`.
再次构建镜像:    

    docker image build -t helloworld:2 .

并且通过`docker image ls`命令查看镜像:

    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    helloworld          2                   7fbedda27c66        3 seconds ago       1.13MB
    helloworld          latest              e61f88f3a0f7        5 minutes ago       122MB
    ubuntu              latest              2d696327ab2e        4 days ago          122MB
    busybox             latest              54511612f1c4        9 days ago          1.13MB  
    
`helloworld:2` 是一种定义镜像名称的格式,通过`:`分割标识/版本号.

<h2 id ="1.3">使用Java创建你的首个镜像</h2>

<h3 id ="1.3.1">创建你的首个镜像</h3>

**Note:**

如果你正在运行 OpenJDK 9
If you are running OpenJDK 9, `mvn package` 可能会失败:

    [ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.1:compile (default-compile) on project helloworld: Compilation failure: Compilation failure:
    [ERROR] Source option 1.5 is no longer supported. Use 1.6 or later.
    [ERROR] Target option 1.5 is no longer supported. Use 1.6 or later.
因为一些Java 5的支持在JDK9中被[废弃](http://openjdk.java.net/jeps/182)了.
你可以通过添加以下属性配置:

      <properties>
        <maven.compiler.source>1.6</maven.compiler.source>
        <maven.compiler.target>1.6</maven.compiler.target>
      </properties>

到生成的pom.xml中用1.6来代替.详情查看 [用Java 9构建Docker 镜像章节](https://github.com/docker/labs/blob/master/developer-tools/java/chapters/chapters/ch03-build-image-java-9.adoc).

创建一个java项目:

    mvn archetype:generate -DgroupId=org.examples.java -DartifactId=helloworld -DinteractiveMode=false

构建项目:

    cd helloworld
    mvn package

运行java应用:

    java -cp target/helloworld-1.0-SNAPSHOT.jar org.examples.java.App

输出如下:

    Hello World!

接下来将这个应用打包成docker镜像

**Java Docker 镜像**

以一种交互性的方式运行 OpenJDK 容器:

    docker container run -it openjdk

这将会在容器当中打开一个终端窗口.检查Java的版本:

    root@8d0af9da5258:/# java -version
    openjdk version "1.8.0_141"
    OpenJDK Runtime Environment (build 1.8.0_141-8u141-b15-1~deb9u1-b15)
    OpenJDK 64-Bit Server VM (build 25.141-b15, mixed mode)

通过在容器 shell中输入`exit` 退出容器.

<h3 id ="1.3.2">作为Docker 镜像打包和运行Java应用</h3>    

在`helloworld`目录创建一个新的Dockerfile文件并且使用以下内容:

    FROM openjdk:latest
    
    COPY target/helloworld-1.0-SNAPSHOT.jar /usr/src/helloworld-1.0-SNAPSHOT.jar
    
    CMD java -cp /usr/src/helloworld-1.0-SNAPSHOT.jar org.examples.java.App
    
构建镜像:

    docker image build -t hello-java:latest .
    
运行镜像:

    docker container run hello-java:latest

输出如下:

    Hello World!
    
<h3 id ="1.3.2">使用Docker Maven Plugin打包和运行Java应用</h3>   

[Docker Maven Plugin](https://github.com/fabric8io/docker-maven-plugin) 允许你通过Maven管理Docker 镜像和容器.
allows you to manage Docker images and containers using Maven. 它通过预定义目标来完成相应操作:

|Goal|	Description|
|:----:|:----:|
|docker:build|Build images|
|docker:start|Create and start containers
|docker:stop|Stop and destroy containers
|docker:push|Push images to a registry
|docker:remove|Remove images from local docker host
|docker:logs|Show container logs

完整的目标列表如下: https://github.com/fabric8io/docker-maven-plugin.

示例下载地址: https://github.com/arun-gupta/docker-java-sample/.

创建Docker 镜像:

    mvn -f docker-java-sample/pom.xml package -Pdocker
    
输出如下:    

    [INFO] Copying files to /Users/argu/workspaces/docker-java-sample/target/docker/hellojava/build/maven
    [INFO] Building tar: /Users/argu/workspaces/docker-java-sample/target/docker/hellojava/tmp/docker-build.tar
    [INFO] DOCKER> [hellojava:latest]: Created docker-build.tar in 87 milliseconds
    [INFO] DOCKER> [hellojava:latest]: Built image sha256:6f815
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------

使用`docker image ls | grep hello-java`命令查看镜像列表:    

    hello-java  latest  ea64a9f5011e   5 seconds ago       643 MB

运行Docker容器:

    mvn -f docker-java-sample/pom.xml install -Pdocker
    
输出如下:

    [INFO] DOCKER> [hellojava:latest]: Start container 30a08791eedb
    30a087> Hello World!
    [INFO] DOCKER> [hellojava:latest]: Waited on log out 'Hello World!' 510 ms

使用 `java` CLI 或者使用 Docker 容器使用 `docker container run` 命令来运行这个Java应用，输出都是相似的.
    
容器运行在最顶层的位置.使用 `Ctrl` + `C` 来终止容器并且返回到终端.

在项目中想要允许或禁止 Docker packaging 和 running ,只需要改变一处地方.在`pom.xml`配置文件中添加如下Maven配置:

    <profiles>
        <profile>
            <id>docker</id>
            <build>
                <plugins>
                    <plugin>
                        <groupId>io.fabric8</groupId>
                        <artifactId>docker-maven-plugin</artifactId>
                        <version>0.22.1</version>
                        <configuration>
                            <images>
                                <image>
                                    <name>hello-java</name>
                                    <build>
                                        <from>openjdk:latest</from>
                                        <assembly>
                                            <descriptorRef>artifact</descriptorRef>
                                        </assembly>
                                        <cmd>java -cp maven/${project.name}-${project.version}.jar org.examples.java.App</cmd>
                                    </build>
                                    <run>
                                        <wait>
                                            <log>Hello World!</log>
                                        </wait>
                                    </run>
                                </image>
                            </images>
                        </configuration>
                        <executions>
                            <execution>
                                <id>docker:build</id>
                                <phase>package</phase>
                                <goals>
                                    <goal>build</goal>
                                </goals>
                            </execution>
                            <execution>
                                <id>docker:start</id>
                                <phase>install</phase>
                                <goals>
                                    <goal>start</goal>
                                    <goal>logs</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>

<h2 id="1.4">Dockerfile 命令设计模式</h2>
    
<h3 id="1.4.1">CMD 和 ENTRYPOINT 之间的不同</h3>

`CMD` 在大多数情况下都能正常工作.

容器默认的入口是`/bin/sh`,即默认的shell.

使用命令`docker container run -it ubuntu` 来运行容器并且启动默认shell.输出如下:

    > docker container run -it ubuntu
    root@88976ddee107:/#

`ENTRYPOINT` 允许使用其它命令重写 entry point ，设置进行自定义.例如,一个容器能这样启动:

    > docker container run -it --entrypoint=/bin/cat ubuntu /etc/passwd
    root:x:0:0:root:/root:/bin/bash
    daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
    bin:x:2:2:bin:/bin:/usr/sbin/nologin
    sys:x:3:3:sys:/dev:/usr/sbin/nologin
    . . .

这个容器重写了容器的 entry point 到 `/bin/cat` 目录. 

<h3 id="1.4.2">ADD 和 COPY 之间的不同</h3>

`COPY` 大多数情况都能正常工作.

`ADD` 拥有 `COPY` 命令的所有功能并且拥有以下额外特性:

* 允许tar 文件在镜像中自动解压缩,例如:`ADD app.tar.gz /opt/var/myapp`.

* 允许文件从远程URL下载.然而,下载文件会成为镜像的一部分.这会使得镜像越来越臃肿.所以一般还是建议使用curl 或者wget的方式来明确地下载，解压缩或者移除存档文件.

<h3 id="1.4.3">导入和导出镜像</h3>

Docker 镜像能使用`image save`命令保存为`.tar`文件.

    docker image save helloworld > helloworld.tar

这些tar文件能够使用`load`命令进行导入:

    docker image load -i helloworld.tar





     