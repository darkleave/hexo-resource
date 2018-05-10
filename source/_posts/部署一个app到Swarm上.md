---
title: 3.0部署一个app到Swarm上
date: 2018-02-12 11:07:42
tags: [docker]
---


这个教程将会通过创建和定制一个投票app来知道你关于swarm上面的部署.
按步骤完成教程至关重要，并且确保定制的部分是开放了定制功能的.

注意：完成这个章节的教程，你需要先完成Docker和git的安装.

<!--more-->

## Voting app

应用源代码路径[Docker Example Voting App](https://github.com/docker/example-voting-app).这个app包含5个组件:
* Python webapp 组件，实现在两个选项之间投票的功能
* Redis queue 收集新的投票
* .NET worker 消费投票并且保存它们在...
* Postgres database 通过Docker volume 来支持
* Node.js webapp 即时展示投票结果

克隆仓库到你的机器上并且通过cd命令到对应目录下:


    git clone https://github.com/docker/example-voting-app.git
    cd example-voting-app

## 部署app

在这首个阶段，我们将会使用在Docker Store中的首个镜像.

这个app依赖[Docker Swarm模式](https://docs.docker.com/engine/swarm/).Swarm mode是嵌入在Docker engine中的集群管理和编排特性.你可以用一个描述你app期望状态的文件来轻松部署到swarm.Swarm允许你运行你的容器在多台机器上.
在这个教程中，你可以使用一台机器，或者使用一些像[Docker for AWS](https://beta.docker.com/)或者[Docker for Azure](https://beta.docker.com/)来快速的创建多个节点的机器.另外，你还可以使用Docker Machine 在你的部署机器上创建多个本地节点.查看[Swarm Mode lab](https://github.com/docker/labs/blob/master/swarm-mode/beginner-tutorial/README.md#creating-the-nodes-and-swarm)查询更多相关信息

首先,创建一个Swarm.

    docker swarm init

接下来，你将需要一个[Docker Compose](https://docs.docker.com/compose)文件.你并不需要安装Docker Compose,尽管如果你使用的是Docker for Mac 或者 Docker for Windows的话默认是已经安装的了.然而**docker stack deploy**接受一个Docker Compose 格式的文件.你需要的文件就在Docker Example Voting App的根目录.文件名为docker-stack.yml.你也可以直接复制和粘贴以下内容:

    version: "3"
    services:
    
      redis:
        image: redis:alpine
        ports:
          - "6379"
        networks:
          - frontend
        deploy:
          replicas: 2
          update_config:
            parallelism: 2
            delay: 10s
          restart_policy:
            condition: on-failure
      db:
        image: postgres:9.4
        volumes:
          - db-data:/var/lib/postgresql/data
        networks:
          - backend
        deploy:
          placement:
            constraints: [node.role == manager]
      vote:
        image: dockersamples/examplevotingapp_vote:before
        ports:
          - 5000:80
        networks:
          - frontend
        depends_on:
          - redis
        deploy:
          replicas: 2
          update_config:
            parallelism: 2
          restart_policy:
            condition: on-failure
      result:
        image: dockersamples/examplevotingapp_result:before
        ports:
          - 5001:80
        networks:
          - backend
        depends_on:
          - db
        deploy:
          replicas: 1
          update_config:
            parallelism: 2
            delay: 10s
          restart_policy:
            condition: on-failure
    
      worker:
        image: dockersamples/examplevotingapp_worker
        networks:
          - frontend
          - backend
        deploy:
          mode: replicated
          replicas: 1
          labels: [APP=VOTING]
          restart_policy:
            condition: on-failure
            delay: 10s
            max_attempts: 3
            window: 120s
          placement:
            constraints: [node.role == manager]
    
      visualizer:
        image: dockersamples/visualizer
        ports:
          - "8080:8080"
        stop_grace_period: 1m30s
        volumes:
          - /var/run/docker.sock:/var/run/docker.sock
        deploy:
          placement:
            constraints: [node.role == manager]
    
    networks:
      frontend:
      backend:
    
    volumes:
      db-data:

首先部署它，然后我们将会深入了解更多细节:

    docker stack deploy --compose-file docker-stack.yml vote
    Creating network vote_frontend
    Creating network vote_backend
    Creating network vote_default
    Creating service vote_vote
    Creating service vote_result
    Creating service vote_worker
    Creating service vote_redis
    Creating service vote_db

验证已经成功部署的堆栈,使用`docker stack services vote`

    docker stack services vote
    ID            NAME         MODE        REPLICAS  IMAGE
    25wo6p7fltyn  vote_db      replicated  1/1       postgres:9.4
    2ot4sz0cgvw3  vote_worker  replicated  1/1       dockersamples/examplevotingapp_worker:latest
    9faz4wbvxpck  vote_redis   replicated  2/2       redis:alpine
    ocm8x2ijtt88  vote_vote    replicated  2/2       dockersamples/examplevotingapp_vote:before
    p1dcwi0fkcbb  vote_result  replicated  2/2       dockersamples/examplevotingapp_result:before

如果你查看`docker-stack.yml文件`,你将会看到文件定义    

* 基于Python image的vote container
* 基于Node.js image的 result container
* 基于redis image的redis container,用以临时存储数据.
* 基于.NET image的.NET based worker app
* 基于postgres image 的Postgres 容器.

Compose文件同样定义两个networks,front-tier 和back-tier .每个容器都是被放置在一个或者两个networks.一旦在那些networks中,它们就可以在那个network中以使用service名称的方式在代码中访问其它处于这个network的services.Services可以处于任意数量的networks.Services被它们各自的network所隔离.即使Services处于同一个network中,Services也只能通过name名称去发现彼此.想了解更多networking,点击[Networking Lab](https://github.com/docker/labs/tree/master/networking).

再看一下Compose文件，你会发现它是以:

    version: "3"

version 3 这个版本号对compose 文件来说非常重要,因为`docker stack deploy` 命令不支持更早版本的.你将会看到那里同样有个`services`key,在下面的是一个分割键(~~不就是一个yml文件么..这么啰嗦~~)，用来隔离每个services,例如:

     vote:
        image: dockersamples/examplevotingapp_vote:before
        ports:
          - 5000:80
        networks:
          - frontend
        depends_on:
          - redis
        deploy:
          replicas: 2
          update_config:
            parallelism: 2
          restart_policy:
            condition: on-failure

`image`键 指定了你能使用哪个镜像,在这个例子中就是`dockersamples/examplevotingapp_vote:before`，如果你对Compose足够熟悉,你可能知道有一个`build`键,用来基于Dockerfile进行构建.然而,`docker stack deploy`不支持`build`，所以你需要使用预构建镜像.

与`docker run`很相似,你能够定义`ports` 和 `networks`.`depends_on`键用来指定一个service只在另一个service之后才进行部署.在这个例子中`vote`只在`redis`之后进行部署.

`deploy`键是version3 版本新增的.它允许你在部署到Swarm时指定不同的属性.在这个例子中,你指定了需要两个 replicas(复制品),也就是在Swarm上部署两个容器.你可以指定其它属性,像什么时候去重启(restart),使用什么样的[healthcheck](https://docs.docker.com/engine/reference/builder/#healthcheck),配置约束(placement constraints),资源(resources)等等.

## 测试运行

现在app已经开始运行了,你可以通过http://localhost:5000来进行访问.

点击其中一个进行投票.你可以通过http://localhost:5001来查看结果.

NOTE: 如果你是在云环境运行这个教程，像AWS,Azure,Digital Ocean,或者GCE 你将不能直接通过localhost或者127.0.0.1来进行访问.一个变通的手段是利用ssh port forwarding.下面是Mac OS的一个例子.这同样适用于Windows 和Putty 用户.

    $ ssh -L 5000:localhost:5000 <ssh-user>@<CLOUD_INSTANCE_IP_ADDRESS>

## 定做app

在这个步骤，你将会定制app并且重新部署它。我们将会使用相同的图片，但是通过使用`after`标签将投票选项cats和dogs换成Java和.NET.

### 改变使用的图片

重新修改`docker-stack.yml`文件,将`vote`和`result` 图片修改为`after`标签,如下:

     vote:
        image: dockersamples/examplevotingapp_vote:after
        ports:
          - 5000:80
        networks:
          - frontend
        depends_on:
          - redis
        deploy:
          replicas: 2
          update_config:
            parallelism: 2
          restart_policy:
            condition: on-failure
      result:
        image: dockersamples/examplevotingapp_result:after
        ports:
          - 5001:80
        networks:
          - backend
        depends_on:
          - db
        deploy:
          replicas: 2
          update_config:
            parallelism: 2
            delay: 10s
          restart_policy:
            condition: on-failure
            
### 重新部署

重新部署跟之前的操作一样.

    docker stack deploy --compose-file docker-stack.yml vote
    
### 测试运行

再次查看app运行界面结果.

### 移除堆栈

从swarm移除堆栈.

    docker stack rm vote

## 后续

现在你已经构建了一些镜像并且把它们推送到Docker Cloud，并且学习到基础的Swarm mode,你可以通过查看[the documentation](https://docs.docker.com/)文档了解更多信息.并且如果你需要任何帮助的话，可以访问[Docker Forums](https://forums.docker.com/)或者[StackOverflow](https://stackoverflow.com/tags/docker/).
     