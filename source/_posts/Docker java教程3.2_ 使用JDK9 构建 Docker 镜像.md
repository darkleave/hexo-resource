---
title: "Docker java教程3.2: 使用JDK9 构建 Docker 镜像"
date: 2018-05-10 23:09:42
tags: [docker]
---


目的: 这个章节解释了如何使用JDK9 创建Docker 镜像.

[之前的章节](https://github.com/docker/labs/blob/master/developer-tools/java/chapters/ch03-build-image.adoc)中讲解了通常情况下Docker镜像的创建方法，这一章节在之前的基础上进行了扩展，主要关注JDK9 的特性.

<!--more-->

[使用JDK9 创建Docker镜像](#1.1)
[使用JDK9 和 Alpine Linux 创建Docker镜像](#1.2)
[使用JDK9  创建Docker镜像 和 Java应用](#1.3)
[使用JDK9  为Docker镜像 和 Java应用 瘦身](#1.4)

<h2 id="1.1">使用JDK9 创建Docker镜像</h2>

创建一个目录，比如:`docker-jdk9`.

在那个目录,创建一个新的text文件`jdk-9-debian-slim.Dockerfile`. 内容如下:

    # A JDK 9 with Debian slim
    FROM debian:stable-slim
    # Download from http://jdk.java.net/9/
    # ADD http://download.java.net/java/GA/jdk9/9/binaries/openjdk-9_linux-x64_bin.tar.gz /opt
    ADD openjdk-9_linux-x64_bin.tar.gz /opt
    # Set up env variables
    ENV JAVA_HOME=/opt/jdk-9
    ENV PATH=$PATH:$JAVA_HOME/bin
    CMD ["jshell", "-J-XX:+UnlockExperimentalVMOptions", \
                   "-J-XX:+UseCGroupMemoryLimitForHeap", \
                   "-R-XX:+UnlockExperimentalVMOptions", \
                   "-R-XX:+UseCGroupMemoryLimitForHeap"]

这个镜像使用debian slim 作为基础镜像 并且安装对应 linxu x64 版本的OpenJDK(查看[设置章节](https://github.com/docker/labs/blob/master/developer-tools/java/chapters/ch01-setup.adoc)了解如何下载到当前目录.

镜像默认配置为运行 `jshell`，即 Java REPL(REPL — 交互式解释器环境。
R(read)、E(evaluate)、P(print)、L(loop)).了解更多[关于JShell的介绍](https://docs.oracle.com/javase/9/jshell/introduction-jshell.htm).实验性的标志参数`-XX:+UseCGroupMemoryLimitForHeap` 传递到REPL进程(前端管理用户输入的进程以及后端管理编译的Java进程).这个选项会确保容器内存约束是有效的.

使用如下命令构建镜像:

    docker image build -t jdk-9-debian-slim -f jdk-9-debian-slim.Dockerfile .

使用`docker image ls`命令展示可用镜像:

    REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
    jdk-9-debian-slim       latest              023f6999d94a        4 hours ago         400MB
    debian                  stable-slim         d30525fb4ed2        4 days ago          55.3MB

JDK9 和 JDK8之间最大的不同被认为是大小的不同,因为JDK 9提供 Java modules,这一点我们将在之后的章节看到.

使用如下命令运行容器:

    docker container run -m=200M -it --rm jdk-9-debian-slim

输出如下:

    INFO: Created user preferences directory.
    |  Welcome to JShell -- Version 9
    |  For an introduction type: /help intro
    
    jshell>
    
在Java REPL 中输入如下表达式来查询Java进程的可用内存:

    Runtime.getRuntime().maxMemory() / (1 << 20)  
    
输出:

    jshell> Runtime.getRuntime().maxMemory() / (1 << 20)
    $1 ==> 100

需要注意的是Java进程受限于内存约束(`--memory` of `docker container run`)并且在指定范围之外将不会为容器分配内存.

在未来的JDK版本将不再需要指定一个特殊标识(`-XX:+UnlockExperimentalVMOptions`)，一旦检测到哪个内存约束是稳定有效的，将会自动应用.

JDK9支持 set CUPs 约束(`--cpuset-cpus` of `docker container run`)但是当前不支持其它CPU约束，比如CPU shares.这是 OpenJDK project 正在进行的工作[跟进](http://openjdk.java.net/jeps/8182070).

注意: CPU sets 和 内存约束同样被移植到 JDK 8 release 8u131 及其以上的版本.

输入`Ctrl` + `D`来退出`jshell`.

展示所有的JDK 9 Java 模块 分布 运行如下命令：

    docker container run -m=200M -it --rm jdk-9-debian-slim java --list-modules

输出如下:

    java.activation@9
    java.base@9
    java.compiler@9
    java.corba@9
    java.datatransfer@9
    java.desktop@9
    java.instrument@9
    java.logging@9
    java.management@9
    java.management.rmi@9
    java.naming@9
    java.prefs@9
    java.rmi@9
    java.scripting@9
    java.se@9
    java.se.ee@9
    java.security.jgss@9
    java.security.sasl@9
    java.smartcardio@9
    java.sql@9
    java.sql.rowset@9
    java.transaction@9
    java.xml@9
    java.xml.bind@9
    java.xml.crypto@9
    java.xml.ws@9
    java.xml.ws.annotation@9
    jdk.accessibility@9
    jdk.aot@9
    jdk.attach@9
    jdk.charsets@9
    jdk.compiler@9
    jdk.crypto.cryptoki@9
    jdk.crypto.ec@9
    jdk.dynalink@9
    jdk.editpad@9
    jdk.hotspot.agent@9
    jdk.httpserver@9
    jdk.incubator.httpclient@9
    jdk.internal.ed@9
    jdk.internal.jvmstat@9
    jdk.internal.le@9
    jdk.internal.opt@9
    jdk.internal.vm.ci@9
    jdk.internal.vm.compiler@9
    jdk.jartool@9
    jdk.javadoc@9
    jdk.jcmd@9
    jdk.jconsole@9
    jdk.jdeps@9
    jdk.jdi@9
    jdk.jdwp.agent@9
    jdk.jlink@9
    jdk.jshell@9
    jdk.jsobject@9
    jdk.jstatd@9
    jdk.localedata@9
    jdk.management@9
    jdk.management.agent@9
    jdk.naming.dns@9
    jdk.naming.rmi@9
    jdk.net@9
    jdk.pack@9
    jdk.policytool@9
    jdk.rmic@9
    jdk.scripting.nashorn@9
    jdk.scripting.nashorn.shell@9
    jdk.sctp@9
    jdk.security.auth@9
    jdk.security.jgss@9
    jdk.unsupported@9
    jdk.xml.bind@9
    jdk.xml.dom@9
    jdk.xml.ws@9
    jdk.zipfs@9  
    
总计75个模块：

    $ docker container run -m=200M -it --rm jdk-9-debian-slim java --list-modules | wc -l
      75

<h2 id="1.2">使用JDK9 和 Alpine Linux 创建Docker镜像</h2>      

与使用debian 作为基础镜像相比,使用Alpine Linux JDK 9 的先行版本能够与muslc 库兼容.

创建一个新的text文件 jdk-9-alpine.Dockerfile.使用如下内容:

    # A JDK 9 with Alpine Linux
    FROM alpine:3.6
    # Add the musl-based JDK 9 distribution
    RUN mkdir /opt
    # Download from http://jdk.java.net/9/
    # ADD http://download.java.net/java/jdk9-alpine/archive/181/binaries/jdk-9-ea+181_linux-x64-musl_bin.tar.gz
    ADD jdk-9-ea+181_linux-x64-musl_bin.tar.gz /opt
    # Set up env variables
    ENV JAVA_HOME=/opt/jdk-9
    ENV PATH=$PATH:$JAVA_HOME/bin
    CMD ["jshell", "-J-XX:+UnlockExperimentalVMOptions", \
                   "-J-XX:+UseCGroupMemoryLimitForHeap", \
                   "-R-XX:+UnlockExperimentalVMOptions", \
                   "-R-XX:+UseCGroupMemoryLimitForHeap"]

这个镜像将alpine 3.6 作为基础镜像并且安装相应版本的OpenJDK.(查看[环境变量设置章节](https://github.com/docker/labs/blob/master/developer-tools/java/chapters/ch01-setup.adoc)了解如何下载到当前目录.

这个镜像配置的方式与debian 基础镜像相同.

使用如下命令构建镜像:

    docker image build -t jdk-9-alpine -f jdk-9-alpine.Dockerfile .

使用`docker image ls`展示可用镜像:

    REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
    jdk-9-debian-slim       latest              023f6999d94a        4 hours ago         400MB
    jdk-9-alpine            latest              f5a57382f240        4 hours ago         356MB
    debian                  stable-slim         d30525fb4ed2        4 days ago          55.3MB
    alpine                  3.6                 7328f6f8b418        3 months ago        3.97MB

注意各个镜像大小的不同.Alpine Linux 被小心地设计为一个迷你型的可运行OS镜像.这个设计的的一部分开销在于可选择的标准库[musl libc](https://www.musl-libc.org/),这个库与C标准库(libc)不兼容,因此JDK需要进行一些改造来运行Alpine Linux.这些改动已经被OpenJDK [Portola Project](http://openjdk.java.net/projects/portola/)提出.

<h2 id="1.3">使用JDK9  创建Docker镜像 和 Java应用</h2>   

Clone  GitHib 项目 https://github.com/PaulSandoz/helloworld-java-9 ,这个项目包含了一个Java基础项目示例:

进入 helloworld-java-9的目录并且在一个正在运行的Docker 容器 使用JDK9 构建这个项目:

    docker container run --volume $PWD:/helloworld-java-9 --workdir /helloworld-java-9 \
    -it --rm openjdk:9-jdk-slim \
    ./mvnw package

(如果你在主机系统本地已经安装了JDK 9,你可以直接通过 `./mvnw package` 命令进行构建.)

在这个示例我们直接使用Docker hub 的 `openjdk:9-jdk-slim`,它已经配置了SSL证书,这样我们能直接通过maven wrapper tool 成功下载 maven 工具. 这个镜像是非官方的并且不以任何形式被 OpenJDK 项目支持(不像 JDK 9 distributions 在最近已经被依赖).相信在未来OpenJDK project发布的JDK版本，将会有root CA 证书(查看[issue JDK-8189131](https://bugs.openjdk.java.net/browse/JDK-8189131))

使用文件 helloworld-jdk-9.Dockerfile 来将这个应用构建为Docker 镜像.
文件内容如下:

    # Hello world application with JDK 9 and Debian slim
    FROM jdk-9-debian-slim
    COPY target/helloworld-1.0-SNAPSHOT.jar /opt/helloworld/helloworld-1.0-SNAPSHOT.jar
    # Set up env variables
    CMD java -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap \
      -cp /opt/helloworld/helloworld-1.0-SNAPSHOT.jar org.examples.java.App

基于`jdk-9-debian-slim`构建一个包含示例Java应用的Docker 镜像:

    docker image build -t helloworld-jdk-9 -f helloworld-jdk-9.Dockerfile .

使用`docker image ls`命令展示可用镜像:

    REPOSITORY              TAG                 IMAGE ID            CREATED              SIZE
    helloworld-jdk-9        latest              eb0539e9529a        19 seconds ago       400MB
    jdk-9-debian-slim       latest              023f6999d94a        5 hours ago          400MB
    jdk-9-alpine            latest              f5a57382f240        5 hours ago          356MB
    openjdk                 9-jdk-slim          6dca67f4790e        3 days ago           372MB
    debian                  stable-slim         d30525fb4ed2        4 days ago           55.3MB
    alpine                  3.6                 7328f6f8b418        3 months ago         3.97MB

注意下 应用镜像 `helloworld-jdk-9`究竟有多大.

运行`jdeps`工具来查看应用所依赖的模块:

    docker container run -it --rm helloworld-jdk-9 jdeps --list-deps /opt/helloworld/helloworld-1.0-SNAPSHOT.jar

可以观察到应用只依赖`java.base`模块.

<h2 id="1.4">使用JDK9  为Docker镜像 和 Java应用 瘦身</h2>   

这个Java应用相当简单并且只使用了JDK 9 发行版本的少量函数,特别是这个应用使用函数只依赖`java.base`模块.我们可以创建一个只包含`java.base` 模块的自定义Java runtime 并且包含到 应用的Docker 镜像当中.

创建一个只包含`java.base`模块的自定义Java runtime:

    docker container run --rm \
      --volume $PWD:/out \
      jdk-9-debian-slim \
      jlink --module-path /opt/jdk-9/jmods \
        --verbose \
        --add-modules java.base \
        --compress 2 \
        --no-header-files \
        --output /out/target/openjdk-9-base_linux-x64

[helloworld-java-9](https://github.com/PaulSandoz/helloworld-java-9)项目中已存在对应的脚本.

JDK 9 工具 `jlink` 一般用来创建自定义 Java 运行时.了解更多关于jlink点击[Tools Reference](https://docs.oracle.com/javase/9/tools/jlink.htm).
这个工具在模块所属的包含JDK 9 及其目录的容器当中执行,/opt/jdk-9/jmods,定义的是模块的路径.
This command exists as create-minimal-java-runtime.sh script in the repo earlier checked out from helloworld-java-9.只有`java.base`模块被选中.

自定义runtime 被输出到目标路径:

    $ du -k target/openjdk-9-base_linux-x64/
    24      target/openjdk-9-base_linux-x64//bin
    12      target/openjdk-9-base_linux-x64//conf/security/policy/limited
    8       target/openjdk-9-base_linux-x64//conf/security/policy/unlimited
    24      target/openjdk-9-base_linux-x64//conf/security/policy
    68      target/openjdk-9-base_linux-x64//conf/security
    76      target/openjdk-9-base_linux-x64//conf
    44      target/openjdk-9-base_linux-x64//legal/java.base
    44      target/openjdk-9-base_linux-x64//legal
    72      target/openjdk-9-base_linux-x64//lib/jli
    16      target/openjdk-9-base_linux-x64//lib/security
    19824   target/openjdk-9-base_linux-x64//lib/server
    31656   target/openjdk-9-base_linux-x64//lib
    31804   target/openjdk-9-base_linux-x64/  

使用`helloworld-jdk-9-base.Dockerfile`文件构建Docker 镜像.文件内容如下:

    # Hello world application with custom Java runtime with just the base module and Debian slim
    FROM debian:stable-slim
    COPY target/openjdk-9-base_linux-x64 /opt/jdk-9
    COPY target/helloworld-1.0-SNAPSHOT.jar /opt/helloworld/helloworld-1.0-SNAPSHOT.jar
    # Set up env variables
    ENV JAVA_HOME=/opt/jdk-9
    ENV PATH=$PATH:$JAVA_HOME/bin
    CMD java -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap \
      -cp /opt/helloworld/helloworld-1.0-SNAPSHOT.jar org.examples.java.App

以 debian:stable-slim作为基础镜像，创建示例Java应用的Docker镜像:

    docker image build -t helloworld-jdk-9-base -f helloworld-jdk-9-base.Dockerfile .  

再度展示可用镜像L`docker image ls`:    

    REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
    helloworld-jdk-9-base   latest              7052483fdb77        24 seconds ago      87.7MB
    helloworld-jdk9         latest              eb0539e9529a        17 minutes ago      400MB
    jdk-9-debian-slim       latest              023f6999d94a        5 hours ago         400MB
    jdk-9-alpine            latest              f5a57382f240        5 hours ago         356MB
    openjdk                 9-jdk-slim          6dca67f4790e        3 days ago          372MB
    debian                  stable-slim         d30525fb4ed2        4 days ago          55.3MB
    alpine                  3.6                 7328f6f8b418        3 months ago        3.97MB
    [source, text]

`helloworld-jdk-9-base` 体积要小得多，并且如果Alpine Linux 使用Debian Slim,体积还能进一步缩小.

一个真实的应用会依赖更多的JDK 模块，但是仍然有可能通过只依赖必须模块的方式来减小Java runtime的体积(比如很多应用就并不需要Corba 或者RMI的编译工具模块).

