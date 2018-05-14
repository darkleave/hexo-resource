---
title: Docker Java教程6：使用Swarm 模式部署应用
date: 2018-05-14 23:09:42
tags: [docker]
---


Docker Engine 包含 swarm 模式 用来管理一个Docker Engines 的集群.Docker CLI 可以用来创建swarm,部署应用服务到swarm上,并且管理swarm的行为.完全的细节地址：https://docs.docker.com/engine/swarm/.在开始这个章节之前需要先了解一些重要的概念: [Swarm mode key concepts](https://docs.docker.com/engine/swarm/key-concepts/) 

<!--more-->

[初始化 Swarm](#1.1)

[多容器应用](#1.2)

[验证在应用中的服务和容器](#1.3)

[访问应用](#1.4)

[停止应用](#1.5)

[完全地移除应用](#1.6)


Swarm 是多个Docker Engines 的集群.但一般情况下我们只会运行一个单独节点的Swarm.

这个章节将会部署一个应用，在[MySQL](https://www.mysql.com/)的数据桶上提供一个 CRUD/REST 接口.这一点通过使用部署在[WildFly](http://wildfly.org/)上的Java EE应用 来访问数据库已经完成》

<h2 id="1.1">初始化 Swarm</h2>

使用如下命令初始化Swarm :

	docker swarm init

这会启动一个 Swarm Manager.默认情况下,一个管理节点同样是一个worker,但是只能配置给一个manager.

使用`docker info`命令可以得到一些关于这个单节点集群的一些信息.

	Containers: 0
	 Running: 0
	 Paused: 0
	 Stopped: 0
	Images: 17
	Server Version: 17.09.0-ce
	Storage Driver: overlay2
	 Backing Filesystem: extfs
	 Supports d_type: true
	 Native Overlay Diff: true
	Logging Driver: json-file
	Cgroup Driver: cgroupfs
	Plugins:
	 Volume: local
	 Network: bridge host ipvlan macvlan null overlay
	 Log: awslogs fluentd gcplogs gelf journald json-file logentries splunk syslog
	Swarm: active
	 NodeID: p9a1tqcjh58ro9ucgtqxa2wgq
	 Is Manager: true
	 ClusterID: r3xdxly8zv82e4kg38krd0vog
	 Managers: 1
	 Nodes: 1
	 Orchestration:
	  Task History Retention Limit: 5
	 Raft:
	  Snapshot Interval: 10000
	  Number of Old Snapshots to Retain: 0
	  Heartbeat Tick: 1
	  Election Tick: 3
	 Dispatcher:
	  Heartbeat Period: 5 seconds
	 CA Configuration:
	  Expiry Duration: 3 months
	  Force Rotate: 0
	 Autolock Managers: false
	 Root Rotation In Progress: false
	 Node Address: 192.168.65.2
	 Manager Addresses:
	  192.168.65.2:2377
	Runtimes: runc
	Default Runtime: runc
	Init Binary: docker-init
	containerd version: 06b9cb35161009dcb7123345749fef02f7cea8e0
	runc version: 3f2f8b84a77f73d38244dd690525642a72156c64
	init version: 949e6fa
	Security Options:
	 seccomp
	  Profile: default
	Kernel Version: 4.9.49-moby
	Operating System: Alpine Linux v3.5
	OSType: linux
	Architecture: x86_64
	CPUs: 4
	Total Memory: 1.952GiB
	Name: moby
	ID: TJSZ:O44Y:PWGZ:ZWZM:SA73:2UHI:VVKV:KLAH:5NPO:AXQZ:XWZD:6IRJ
	Docker Root Dir: /var/lib/docker
	Debug Mode (client): false
	Debug Mode (server): true
	 File Descriptors: 35
	 Goroutines: 142
	 System Time: 2017-10-05T20:57:14.037442426Z
	 EventsListeners: 1
	Registry: https://index.docker.io/v1/
	Experimental: true
	Insecure Registries:
	 127.0.0.0/8
	Live Restore Enabled: false

这个集群有一个节点，同时也是一个管理节点.默认情况下，管理节点也同样是一个worker 节点.这意味着容器能在这个节点上运行.

<h2 id="1.2">多容器应用</h2>

这个章节描述了如何使用 Docker Compose 在 Swarm 模式下使用 Docker CLI 来部署一个多容器应用.

创建一个新目录并且使用`cd`命令移动到此:

	mkdir webapp
	cd webapp

使用[configuration file](https://github.com/docker/labs/blob/master/developer-tools/java/chapters/ch05-compose.adoc#configuration-file)创建一个新的Compose 定义文件 `docker-compose.yml`.

这个应用能够用如下命令启动:

	docker stack deploy --compose-file=docker-compose.yml webapp

这里展示的输出如下:

	Ignoring deprecated options:
	
	container_name: Setting the container name is not supported.
	
	Creating network webapp_default
	Creating service webapp_db
	Creating service webapp_web


WildFly Swarm 和 MySQL services 在这个节点上启动.每个 service 有一个单独的容器. 如果Swarm 模式在多个节点上可用的话，那么所有容器将会通过这些节点进行分布式部署.

一个新的覆盖网络被创建.这可以通过`docker network ls`命令来验证.这个网络允许多个容器在不同的hosts上互相交互联系.


<h2 id="1.3">验证在应用中的服务和容器</h2>

使用`docker service ls`命令验证WildFly 和 MySQL 服务.

	ID                  NAME                MODE                REPLICAS            IMAGE                                   PORTS
	j21lwelj529f        webapp_db           replicated          1/1                 mysql:8                                 *:3306->3306/tcp
	m0m44axt35cg        webapp_web          replicated          1/1                 arungupta/docker-javaee:dockerconeu17   *:8080->8080/tcp,*:9990->9990/tcp


使用`docker service inspect webapp_web` 命令取得更多细节:

	[
	    {
	        "ID": "m0m44axt35cgjetcjwzls7u9r",
	        "Version": {
	            "Index": 22
	        },
	        "CreatedAt": "2017-10-07T00:17:44.038961419Z",
	        "UpdatedAt": "2017-10-07T00:17:44.040746062Z",
	        "Spec": {
	            "Name": "webapp_web",
	            "Labels": {
	                "com.docker.stack.image": "arungupta/docker-javaee:dockerconeu17",
	                "com.docker.stack.namespace": "webapp"
	            },
	            "TaskTemplate": {
	                "ContainerSpec": {
	                    "Image": "arungupta/docker-javaee:dockerconeu17@sha256:6a403c35d2ab4442f029849207068eadd8180c67e2166478bc3294adbf578251",
	                    "Labels": {
	                        "com.docker.stack.namespace": "webapp"
	                    },
	                    "Privileges": {
	                        "CredentialSpec": null,
	                        "SELinuxContext": null
	                    },
	                    "StopGracePeriod": 10000000000,
	                    "DNSConfig": {}
	                },
	                "Resources": {},
	                "RestartPolicy": {
	                    "Condition": "any",
	                    "Delay": 5000000000,
	                    "MaxAttempts": 0
	                },
	                "Placement": {
	                    "Platforms": [
	                        {
	                            "Architecture": "amd64",
	                            "OS": "linux"
	                        }
	                    ]
	                },
	                "Networks": [
	                    {
	                        "Target": "bwnp1nvkkga68dirhp1ue7qey",
	                        "Aliases": [
	                            "web"
	                        ]
	                    }
	                ],
	                "ForceUpdate": 0,
	                "Runtime": "container"
	            },
	            "Mode": {
	                "Replicated": {
	                    "Replicas": 1
	                }
	            },
	            "UpdateConfig": {
	                "Parallelism": 1,
	                "FailureAction": "pause",
	                "Monitor": 5000000000,
	                "MaxFailureRatio": 0,
	                "Order": "stop-first"
	            },
	            "RollbackConfig": {
	                "Parallelism": 1,
	                "FailureAction": "pause",
	                "Monitor": 5000000000,
	                "MaxFailureRatio": 0,
	                "Order": "stop-first"
	            },
	            "EndpointSpec": {
	                "Mode": "vip",
	                "Ports": [
	                    {
	                        "Protocol": "tcp",
	                        "TargetPort": 8080,
	                        "PublishedPort": 8080,
	                        "PublishMode": "ingress"
	                    },
	                    {
	                        "Protocol": "tcp",
	                        "TargetPort": 9990,
	                        "PublishedPort": 9990,
	                        "PublishMode": "ingress"
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
	                        "TargetPort": 8080,
	                        "PublishedPort": 8080,
	                        "PublishMode": "ingress"
	                    },
	                    {
	                        "Protocol": "tcp",
	                        "TargetPort": 9990,
	                        "PublishedPort": 9990,
	                        "PublishMode": "ingress"
	                    }
	                ]
	            },
	            "Ports": [
	                {
	                    "Protocol": "tcp",
	                    "TargetPort": 8080,
	                    "PublishedPort": 8080,
	                    "PublishMode": "ingress"
	                },
	                {
	                    "Protocol": "tcp",
	                    "TargetPort": 9990,
	                    "PublishedPort": 9990,
	                    "PublishMode": "ingress"
	                }
	            ],
	            "VirtualIPs": [
	                {
	                    "NetworkID": "vysfza7wgjepdlutuwuigbws1",
	                    "Addr": "10.255.0.5/16"
	                },
	                {
	                    "NetworkID": "bwnp1nvkkga68dirhp1ue7qey",
	                    "Addr": "10.0.0.4/24"
	                }
	            ]
	        }
	    }
	]

使用`docker service logs -f webapp_web`命令来查看service 的日志.

	webapp_web.1.lf3y5k7pkpt9@moby    | 00:17:47,296 INFO  [org.jboss.msc] (main) JBoss MSC version 1.2.6.Final
	webapp_web.1.lf3y5k7pkpt9@moby    | 00:17:47,404 INFO  [org.jboss.as] (MSC service thread 1-8) WFLYSRV0049: WildFly Core 2.0.10.Final "Kenny" starting
	webapp_web.1.lf3y5k7pkpt9@moby    | 2017-10-07 00:17:48,636 INFO  [org.wildfly.extension.io] (ServerService Thread Pool -- 20) WFLYIO001: Worker 'default' has auto-configured to 8 core threads with 64 task threads based on your 4 available processors
	
	. . .
	
	webapp_web.1.lf3y5k7pkpt9@moby    | 2017-10-07 00:17:56,619 INFO  [org.jboss.resteasy.resteasy_jaxrs.i18n] (ServerService Thread Pool -- 12) RESTEASY002225: Deploying javax.ws.rs.core.Application: class org.javaee.samples.employees.MyApplication
	webapp_web.1.lf3y5k7pkpt9@moby    | 2017-10-07 00:17:56,621 WARN  [org.jboss.as.weld] (ServerService Thread Pool -- 12) WFLYWELD0052: Using deployment classloader to load proxy classes for module com.fasterxml.jackson.jaxrs.jackson-jaxrs-json-provider:main. Package-private access will not work. To fix this the module should declare dependencies on [org.jboss.weld.core, org.jboss.weld.spi]
	webapp_web.1.lf3y5k7pkpt9@moby    | 2017-10-07 00:17:56,682 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 12) WFLYUT0021: Registered web context: /
	webapp_web.1.lf3y5k7pkpt9@moby    | 2017-10-07 00:17:57,094 INFO  [org.jboss.as.server] (main) WFLYSRV0010: Deployed "docker-javaee.war" (runtime-name : "docker-javaee.war")


<h2 id="1.4">访问应用</h2>

现在 WildFly 和 MySQL 服务器已经被配置,接下来根据指定的ip地址进行访问:

	curl -v http://localhost:8080/resources/employees


输出如下:

	*   Trying ::1...
	* TCP_NODELAY set
	* Connected to localhost (::1) port 8080 (#0)
	> GET /resources/employees HTTP/1.1
	> Host: localhost:8080
	> User-Agent: curl/7.51.0
	> Accept: */*
	>
	< HTTP/1.1 200 OK
	< Connection: keep-alive
	< Content-Type: application/xml
	< Content-Length: 478
	< Date: Sat, 07 Oct 2017 00:22:59 GMT
	<
	* Curl_http_done: called premature == 0
	* Connection #0 to host localhost left intact
	<?xml version="1.0" encoding="UTF-8" standalone="yes"?><collection><employee><id>1</id><name>Penny</name></employee><employee><id>2</id><name>Sheldon</name></employee><employee><id>3</id><name>Amy</name></employee><employee><id>4</id><name>Leonard</name></employee><employee><id>5</id><name>Bernadette</name></employee><employee><id>6</id><name>Raj</name></employee><employee><id>7</id><name>Howard</name></employee><employee><id>8</id><name>Priya</name></employee></collection>

<h2 id="1.5">停止应用</h2>

如果你只是想临时性地停止应用，并保持已经被创建的networks，推荐的方式是将service replicas(服务副本)的数量设置为0.


	docker service scale webapp_db=0 webapp_web=0


输出如下:

	webapp_db scaled to 0
	webapp_web scaled to 0
	Since --detach=false was not specified, tasks will be scaled in the background.
	In a future release, --detach=false will become the default.

这特别有用尤其是堆具备一定体积而你又想保存数据的时候.它允许你再次启动堆,通过将replicas 的数设置为大于0的方式.

<h2 id="1.6">完全地移除应用</h2>

使用`docker stack rm webapp`命令关闭应用 :

	Removing service webapp_db
	Removing service webapp_web
	Removing network webapp_default

这会停止在每个service中的容器，并且移除services.这同样会删除这个应用中的所有网络通信.


[官方链接](https://github.com/docker/labs/blob/master/developer-tools/java/chapters/ch06-swarm.adoc)