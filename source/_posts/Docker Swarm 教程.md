---
title: Docker Swarm 教程
date: 2018-03-15 22:08:42
tags: [docker]
---


Note:这个教程使用Docker Machine 来模拟多台机器运行的情况.有一种更简单的方式来学习swarm mode,那就是通过[Play with Docker](http://training.play-with-docker.com/swarm-mode-intro/).这个教程因为历史原因保留下来，并且同样因为当你真的想要使用自己的机器来学习swarm时.

<!--more-->

Docker 的swarm 模式是用来管理Docker Engines的集群,所以称之为swarm.你能使用Docker CLI 来创建swarm,部署应用服务到一个swarm,并且管理swarm的行为.这个教程使用[Docker Machine](https://docs.docker.com/machine/)来在你的桌面电脑上创建多个节点.你也可以选择在云上或者多台机器上创建多个节点.

重要提示: 你并不需要使用Docker CLI来执行这些操作.只需要使用`docker stack deploy --compose-file STACKNAME.yml STACKNAME` 命令即可.关于以compose 文件格式的stack文件来部署一个app,参考[Deploying an app to a Swarm](https://github.com/docker/labs/blob/master/beginner/chapters/votingapp.md)

## 准备

你需要在你的系统上安装Docker 和Docker Machine.[下载Docker](https://docker.com/getdocker)并且安装.

提示:

* 如果你是使用Docker for Mac 或者Docker for Windows,Docker Machine已经默认安装.查看[Download Docker for Mac](https://docs.docker.com/docker-for-mac/#/download-docker-for-mac)和[Download Docker for Windows](https://docs.docker.com/docker-for-windows/#/download-docker-for-windows)来查询详细安装细节.

* 如果你是使用Docker for Windows 你还需要使用Hyper-V driver 来正常驱动Docker Machine.这可能需要更多一点设置.查看[Microsoft Hyper-V driver documentation](https://docs.docker.com/machine/drivers/hyper-v/)来获取设置指引.

* 如果你是直接在linux系统上使用Docker,你将需要安装Docker Machine(在安装Docker Engine之后).

## 创建nodes和Swarm

[Docker Machine](https://docs.docker.com/machine/overview/)可以用来:

* 在Mac或者Windows上安装和运行Docker
* 提供和管理多个远程Docker 主机
* 提供Swarm 集群

但它同样能用来在你本地的机器上创建多个节点.在这个仓库有一个[bash脚本](https://github.com/docker/labs/blob/master/swarm-mode/beginner-tutorial/swarm-node-vbox-setup.sh)实现了这一点并且创建swarm.同样有一个[powershell Hyper-V version](https://github.com/docker/labs/blob/master/swarm-mode/beginner-tutorial/swarm-node-hyperv-setup.ps1).在这篇教程我们将贯穿这个脚本,步步深入，除了设置以外,与Hyper-V version基本相同.

首先创建3个machines,并且命名为manger1,manger2和manger3.

    #!/bin/bash
    
    # Swarm mode using Docker Machine
    
    #This configures the number of workers and managers in the swarm
    managers=3
    workers=3
    
    # This creates the manager machines
    echo "======> Creating $managers manager machines ...";
    for node in $(seq 1 $managers);
    do
    	echo "======> Creating manager$node machine ...";
    	docker-machine create -d virtualbox manager$node;
    done
    
第二个步骤是创建三个及以上的machines,并且命名为worker1,worker2,和worker3

    # This create worker machines
    echo "======> Creating $workers worker machines ...";
    for node in $(seq 1 $workers);
    do
    	echo "======> Creating worker$node machine ...";
    	docker-machine create -d virtualbox worker$node;
    done
    
    # This lists all machines created
    docker-machine ls

接下来你创建一个swarm通过在首个manger上初始化它.通过`docker-machine ssh`来运行`docker-machine ssh`命令.

    # initialize swarm mode and create a manager
    echo "======> Initializing first swarm manager ..."
    docker-machine ssh manager1 "docker swarm init --listen-addr $(docker-machine ip manager1) --advertise-addr $(docker-machine ip manager1)"

接下来为mangers和workers获取tokens.

    # get manager and worker tokens
    export manager_token=`docker-machine ssh manager1 "docker swarm join-token manager -q"`
    export worker_token=`docker-machine ssh manager1 "docker swarm join-token worker -q"`

然后把其他managers加入到Swarm

    for node in $(seq 2 $managers);
    do
    	echo "======> manager$node joining swarm as manager ..."
    	docker-machine ssh manager$node \
    		"docker swarm join \
    		--token $manager_token \
    		--listen-addr $(docker-machine ip manager$node) \
    		--advertise-addr $(docker-machine ip manager$node) \
    		$(docker-machine ip manager1)"
    done

最后，添加worker machines并且加入swarm


    # workers join swarm
    for node in $(seq 1 $workers);
    do
    	echo "======> worker$node joining swarm as worker ..."
    	docker-machine ssh worker$node \
    	"docker swarm join \
    	--token $worker_token \
    	--listen-addr $(docker-machine ip worker$node) \
    	--advertise-addr $(docker-machine ip worker$node) \
    	$(docker-machine ip manager1):2377"
    done
    
    # show members of swarm
    docker-machine ssh manager1 "docker node ls"

最后一行将向你展示所有的节点,就像这样:

    ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
    3cq6idpysa53n6a21nqe0924h    manager3  Ready   Active        Reachable
    64swze471iu5silg83ls0bdip *  manager1  Ready   Active        Leader
    7eljvvg0icxlw20od5f51oq8t    manager2  Ready   Active        Reachable
    8awcmkj3sd9nv1pi77i6mdb1i    worker1   Ready   Active        
    avu80ol573rzepx8ov80ygzxz    worker2   Ready   Active        
    bxn1iivy8w7faeugpep76w50j    worker3   Ready   Active   
    

也可以运行以下命令:

    $ docker-machine ls
    NAME       ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER      ERRORS
    manager1   -        virtualbox   Running   tcp://192.168.99.100:2376           v17.03.0-ce   
    manager2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.03.0-ce 
    manager3   -        virtualbox   Running   tcp://192.168.99.102:2376           v17.03.0-ce
    worker1    -        virtualbox   Running   tcp://192.168.99.103:2376           v17.03.0-ce
    worker2    -        virtualbox   Running   tcp://192.168.99.104:2376           v17.03.0-ce
    worker3    -        virtualbox   Running   tcp://192.168.99.105:2376           v17.03.0-ce

下一步是创建一个service并且进行展示.这里创建单个命名为web的service并且运行最新的nginx:

    $ docker-machine ssh manager1 "docker service create -p 80:80 --name web nginx:latest"
    $ docker-machine ssh manager1 "docker service ls"
    ID            NAME  REPLICAS  IMAGE         COMMAND
    2x4jsk6313az  web   1/1       nginx:latest    

现在在你的浏览器上通过对应ip地址进行访问.在这个实例中manger1有一个ip地址为192.168.99.100.

你可以使用任意其它节点的ip地址进行访问都将得到同样的结果,因为[Swarm Mode's Routing Mesh](https://docs.docker.com/engine/swarm/ingress/).

现在检查service:

    $ docker-machine ssh manager1 "docker service inspect web"
    [
        {
            "ID": "2x4jsk6313azr6g1dwoi47z8u",
            "Version": {
                "Index": 104
            },
            "CreatedAt": "2016-08-23T22:43:23.573253682Z",
            "UpdatedAt": "2016-08-23T22:43:23.576157266Z",
            "Spec": {
                "Name": "web",
                "TaskTemplate": {
                    "ContainerSpec": {
                        "Image": "nginx:latest"
                    },
                    "Resources": {
                        "Limits": {},
                        "Reservations": {}
                    },
                    "RestartPolicy": {
                        "Condition": "any",
                        "MaxAttempts": 0
                    },
                    "Placement": {}
                },
                "Mode": {
                    "Replicated": {
                        "Replicas": 1
                    }
                },
                "UpdateConfig": {
                    "Parallelism": 1,
                    "FailureAction": "pause"
                },
                "EndpointSpec": {
                    "Mode": "vip",
                    "Ports": [
                        {
                            "Protocol": "tcp",
                            "TargetPort": 80,
                            "PublishedPort": 80
                        }
                    ]
                }
            },
            "Endpoint": {
                "Spec": {
                    "Mode": "vip",
                    "Ports": [
                        {
                            "Protocol": "tcp",
                            "TargetPort": 80,
                            "PublishedPort": 80
                        }
                    ]
                },
                "Ports": [
                    {
                        "Protocol": "tcp",
                        "TargetPort": 80,
                        "PublishedPort": 80
                    }
                ],
                "VirtualIPs": [
                    {
                        "NetworkID": "24r1loluvdohuzltspkwbhsc8",
                        "Addr": "10.255.0.9/16"
                    }
                ]
            },
            "UpdateStatus": {
                "StartedAt": "0001-01-01T00:00:00Z",
                "CompletedAt": "0001-01-01T00:00:00Z"
            }
        }
    ]
    
规模化service(运行多个相同service):    

    $ docker-machine ssh manager1 "docker service scale web=15"
    web scaled to 15
    $ docker-machine ssh manager1 "docker service ls"
    ID            NAME  REPLICAS  IMAGE         COMMAND
    2x4jsk6313az  web   15/15     nginx:latest  

Docker会将15个服务均匀地分配到所有节点上:

    $ docker-machine ssh manager1 "docker service ps web"
    ID                         NAME    IMAGE         NODE      DESIRED STATE  CURRENT STATE           ERROR
    61wjx0zaovwtzywwbomnvjo4q  web.1   nginx:latest  worker3   Running        Running 13 minutes ago  
    bkkujhpbtqab8fyhah06apvca  web.2   nginx:latest  manager1  Running        Running 2 minutes ago   
    09zkslrkgrvbscv0vfqn2j5dw  web.3   nginx:latest  manager1  Running        Running 2 minutes ago   
    4dlmy8k72eoza9t4yp9c9pq0w  web.4   nginx:latest  manager2  Running        Running 2 minutes ago   
    6yqabr8kajx5em2auvfzvi8wi  web.5   nginx:latest  manager3  Running        Running 2 minutes ago   
    21x7sn82883e7oymz57j75q4q  web.6   nginx:latest  manager2  Running        Running 2 minutes ago   
    14555mvu3zee6aek4dwonxz3f  web.7   nginx:latest  worker1   Running        Running 2 minutes ago   
    1q8imt07i564bm90at3r2w198  web.8   nginx:latest  manager1  Running        Running 2 minutes ago   
    encwziari9h78ue32v5pjq9jv  web.9   nginx:latest  worker3   Running        Running 2 minutes ago   
    aivwszsjhhpky43t3x7o8ezz9  web.10  nginx:latest  worker2   Running        Running 2 minutes ago   
    457fsqomatl1lgd9qbz2dcqsb  web.11  nginx:latest  worker1   Running        Running 2 minutes ago   
    7chhofuj4shhqdkwu67512h1b  web.12  nginx:latest  worker2   Running        Running 2 minutes ago   
    7dynic159wyouch05fyiskrd0  web.13  nginx:latest  worker1   Running        Running 2 minutes ago   
    7zg9eki4610maigr1xwrx7zqk  web.14  nginx:latest  manager3  Running        Running 2 minutes ago   
    4z2c9j20gwsasosvj7mkzlyhc  web.15  nginx:latest  manager2  Running        Running 2 minutes ago   
 
你同样能排除一个特定的节点，通过从那个节点移除所有的服务.这些服务会自动重新安排到其它节点上.

    $ docker-machine ssh manager1 "docker node update --availability drain worker1"
    worker1
    $ docker-machine ssh manager1 "docker service ps web"
    ID                         NAME        IMAGE         NODE      DESIRED STATE  CURRENT STATE           ERROR
    61wjx0zaovwtzywwbomnvjo4q  web.1       nginx:latest  worker3   Running        Running 15 minutes ago  
    bkkujhpbtqab8fyhah06apvca  web.2       nginx:latest  manager1  Running        Running 4 minutes ago   
    09zkslrkgrvbscv0vfqn2j5dw  web.3       nginx:latest  manager1  Running        Running 4 minutes ago   
    4dlmy8k72eoza9t4yp9c9pq0w  web.4       nginx:latest  manager2  Running        Running 4 minutes ago   
    6yqabr8kajx5em2auvfzvi8wi  web.5       nginx:latest  manager3  Running        Running 4 minutes ago   
    21x7sn82883e7oymz57j75q4q  web.6       nginx:latest  manager2  Running        Running 4 minutes ago   
    8so0xi55kqimch2jojfdr13qk  web.7       nginx:latest  worker3   Running        Running 3 seconds ago   
    14555mvu3zee6aek4dwonxz3f   \_ web.7   nginx:latest  worker1   Shutdown       Shutdown 4 seconds ago  
    1q8imt07i564bm90at3r2w198  web.8       nginx:latest  manager1  Running        Running 4 minutes ago   
    encwziari9h78ue32v5pjq9jv  web.9       nginx:latest  worker3   Running        Running 4 minutes ago   
    aivwszsjhhpky43t3x7o8ezz9  web.10      nginx:latest  worker2   Running        Running 4 minutes ago   
    738jlmoo6tvrkxxar4gbdogzf  web.11      nginx:latest  worker2   Running        Running 3 seconds ago   
    457fsqomatl1lgd9qbz2dcqsb   \_ web.11  nginx:latest  worker1   Shutdown       Shutdown 3 seconds ago  
    7chhofuj4shhqdkwu67512h1b  web.12      nginx:latest  worker2   Running        Running 4 minutes ago   
    4h7zcsktbku7peh4o32mw4948  web.13      nginx:latest  manager3  Running        Running 3 seconds ago   
    7dynic159wyouch05fyiskrd0   \_ web.13  nginx:latest  worker1   Shutdown       Shutdown 4 seconds ago  
    7zg9eki4610maigr1xwrx7zqk  web.14      nginx:latest  manager3  Running        Running 4 minutes ago   
    4z2c9j20gwsasosvj7mkzlyhc  web.15      nginx:latest  manager2  Running        Running 4 minutes ago   

worker1节点仍然存活但是被标记为drain 状态.

    $ docker-machine ssh manager1 "docker node ls"
    ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
    3cq6idpysa53n6a21nqe0924h    manager3  Ready   Active        Reachable
    64swze471iu5silg83ls0bdip *  manager1  Ready   Active        Leader
    7eljvvg0icxlw20od5f51oq8t    manager2  Ready   Active        Reachable
    8awcmkj3sd9nv1pi77i6mdb1i    worker1   Ready   Drain         
    avu80ol573rzepx8ov80ygzxz    worker2   Ready   Active        
    bxn1iivy8w7faeugpep76w50j    worker3   Ready   Active
    
降低服务的规模:

    $ docker-machine ssh manager1 "docker service scale web=10"
    web scaled to 10
    $ docker-machine ssh manager1 "docker service ps web"
    ID                         NAME        IMAGE         NODE      DESIRED STATE  CURRENT STATE            ERROR
    61wjx0zaovwtzywwbomnvjo4q  web.1       nginx:latest  worker3   Running        Running 22 minutes ago   
    bkkujhpbtqab8fyhah06apvca  web.2       nginx:latest  manager1  Shutdown       Shutdown 54 seconds ago  
    09zkslrkgrvbscv0vfqn2j5dw  web.3       nginx:latest  manager1  Running        Running 11 minutes ago   
    4dlmy8k72eoza9t4yp9c9pq0w  web.4       nginx:latest  manager2  Running        Running 11 minutes ago   
    6yqabr8kajx5em2auvfzvi8wi  web.5       nginx:latest  manager3  Running        Running 11 minutes ago   
    21x7sn82883e7oymz57j75q4q  web.6       nginx:latest  manager2  Running        Running 11 minutes ago   
    8so0xi55kqimch2jojfdr13qk  web.7       nginx:latest  worker3   Running        Running 7 minutes ago    
    14555mvu3zee6aek4dwonxz3f   \_ web.7   nginx:latest  worker1   Shutdown       Shutdown 7 minutes ago   
    1q8imt07i564bm90at3r2w198  web.8       nginx:latest  manager1  Running        Running 11 minutes ago   
    encwziari9h78ue32v5pjq9jv  web.9       nginx:latest  worker3   Shutdown       Shutdown 54 seconds ago  
    aivwszsjhhpky43t3x7o8ezz9  web.10      nginx:latest  worker2   Shutdown       Shutdown 54 seconds ago  
    738jlmoo6tvrkxxar4gbdogzf  web.11      nginx:latest  worker2   Running        Running 7 minutes ago    
    457fsqomatl1lgd9qbz2dcqsb   \_ web.11  nginx:latest  worker1   Shutdown       Shutdown 7 minutes ago   
    7chhofuj4shhqdkwu67512h1b  web.12      nginx:latest  worker2   Running        Running 11 minutes ago   
    4h7zcsktbku7peh4o32mw4948  web.13      nginx:latest  manager3  Running        Running 7 minutes ago    
    7dynic159wyouch05fyiskrd0   \_ web.13  nginx:latest  worker1   Shutdown       Shutdown 7 minutes ago   
    7zg9eki4610maigr1xwrx7zqk  web.14      nginx:latest  manager3  Shutdown       Shutdown 54 seconds ago  
    4z2c9j20gwsasosvj7mkzlyhc  web.15      nginx:latest  manager2  Shutdown       Shutdown 54 seconds ago  
    
现在重新把worker1 恢复为在线状态并且显示它的状态:

    $ docker-machine ssh manager1 "docker node update --availability active worker1"
    worker1
    $ docker-machine ssh manager1 "docker node inspect worker1 --pretty"
    ID:			8awcmkj3sd9nv1pi77i6mdb1i
    Hostname:		worker1
    Joined at:		2016-08-23 22:30:15.556517377 +0000 utc
    Status:
     State:			Ready
     Availability:		Active
    Platform:
     Operating System:	linux
     Architecture:		x86_64
    Resources:
     CPUs:			1
     Memory:		995.9 MiB
    Plugins:
      Network:		bridge, host, null, overlay
      Volume:		local
    Engine Version:		17.03.0-ce
    Engine Labels:
     - provider = virtualbox

现在把manager1 leader节点,从Swarm中移除:

    $ docker-machine ssh manager1 "docker swarm leave --force"
    Node left the swarm. 

等待30秒来确定生效.Swarm仍然可用,但是必须重新选举一个新的leader.这会自动发生.

    $ docker-machine ssh manager2 "docker node ls"
    ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
    3cq6idpysa53n6a21nqe0924h    manager3  Ready   Active        Reachable
    64swze471iu5silg83ls0bdip    manager1  Down    Active        Unreachable
    7eljvvg0icxlw20od5f51oq8t *  manager2  Ready   Active        Leader
    8awcmkj3sd9nv1pi77i6mdb1i    worker1   Ready   Active        
    avu80ol573rzepx8ov80ygzxz    worker2   Ready   Active        
    bxn1iivy8w7faeugpep76w50j    worker3   Ready   Active

可以看到manager1变为Down 并且 Unreachable ,并且manager2已经被选举为leader.同样简单的方式移除一个service:

    $ docker-machine ssh manager2 "docker service rm web"
    web

## Cleanup 

    $ ./swarm-node-vbox-teardown.sh
    Stopping "manager3"...
    Stopping "manager2"...
    Stopping "worker1"...
    Stopping "manager1"...
    Stopping "worker3"...
    Stopping "worker2"...
    Machine "manager3" was stopped.
    Machine "manager1" was stopped.
    Machine "manager2" was stopped.
    Machine "worker2" was stopped.
    Machine "worker1" was stopped.
    Machine "worker3" was stopped.
    About to remove worker1, worker2, worker3, manager1, manager2, manager3
    Are you sure? (y/n): y
    Successfully removed worker1
    Successfully removed worker2
    Successfully removed worker3
    Successfully removed manager1
    Successfully removed manager2
    Successfully removed manager3
    
## Next Steps

更多详细信息查看[Docker Swarm Mode 文档](https://docs.docker.com/engine/swarm/)