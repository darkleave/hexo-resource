---
title: Ubuntu docker 安装
date: 2018-02-04 00:07:42
tags: [docker]
---

## Docker是什么?
Docker是一种容器技术.
Docker容器技术已在云计算市场上风靡一时了，使docker容器技术如此受欢迎的原因是，容器技术可实现不同云计算之间应用程序的可移植性，以及提供了一个把应用程序拆分为分布式组件的方法，此外，用户还可以管理和扩展这些组件成为集群.

<!--more-->

[官网链接](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

一旦你完成docker的安装后,可以用如下命令对你的docker进行测试:

    $ docker run hello-world
    Unable to find image 'hello-world:latest' locally
    latest: Pulling from library/hello-world
    03f4658f8b78: Pull complete
    a3ed95caeb02: Pull complete
    Digest: sha256:8be990ef2aeb16dbcb9271ddfe2610fa6658d13f6dfb8bc72074cc1ca36966a7
    Status: Downloaded newer image for hello-world:latest
    
    Hello from Docker.
    This message shows that your installation appears to be working correctly.
    ...
    
### 卸载老版本

老版本的Docker一般命名为 docker 或者 docker-engine.在安装新版本前需要对他们进行卸载.

    $ sudo apt-get remove docker docker-engine docker.io

即时apt-get命令提示没有上述的任意包被安装也没有影响.
/var/lib/docker/ 文件夹一般包含images, containers, volumes, and networks等.
 Docker CE包现在被命名为: docker-ce
 
### 安装Docker CE

你可以根据需要选择以下合适的方式来安装Docker CE：

* 通过repository进行安装
  在你第一次在一台新主机上安装docker之前,你需要先设置Docker仓库,之后,你就可以直接通过仓库安装和更新Docker.
  **设置仓库**
  1. 更新apt命令的包索引
 ```
     sudo apt-get update 
 ```
  2. 安装仓库使用所需的https依赖包
    
        ```
        sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        software-properties-common
        ```
  3. 添加Docker的官方GPG key
        ```
        
        $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
        ```
     验证你现在拥有的指纹秘钥:(eg:)
     `9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88`
     搜索最后八个长度的指纹:
     
        ```
    
        $ sudo apt-key fingerprint 0EBFCD88
        
        pub   4096R/0EBFCD88 2017-02-22
              Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
        uid                  Docker Release (CE deb) <docker@docker.com>
        sub   4096R/F273FCD8 2017-02-22
        ```    
  4. 使用以下命令来设置你的stable repository(稳定仓库).你总是需要stable repository,即使你想要从edge或者test仓库安装构建也一样.想要添加edge或者test仓库,只需在以下命令的stable后面添加相应的edge或者test即可.

        ```
    
       提示：lsb_release -cs 子命令返回你Ubuntu发布版本的名称,例如xenial.有时候,一些发布版本,比如Linux Mint,你可能需要把 $(lsb_release -cs)命令换成你对应的父级 Ubuntu 发布版本比如,如果你正在使用Linux Mint Rafaela,你可能要使用 trusty.
       
        ```
**x86_64 / amd64**
    ```
    $ sudo add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
       $(lsb_release -cs) \
       stable"
    ```
     **提示:从17.06版本开始,stable 发布时会同样推送到edge和test仓库.**
     
        [Learn about stable and edge channels.](https://docs.docker.com/install/)
        
  **Docker CE安装步骤**
    
  1. 更新apt包索引...
    
        ```
        $ sudo apt-get update
          
        ```      
        
  2. 安装最新版本的Docker CE或者直接到下一步安装特定版本的Docker CE.
      任何当前的版本会被替换为安装的版本.
      ```
        $ sudo apt-get install docker-ce
      ```
  
      ```
      
      获取多个Docker仓库?
      如果你有多个可用的Docker仓库,安装或者更新操作没有指定特定的版本,apt-get install 或者apt-get update 命令总是会按照最高的可用版本,可能并不满足你稳定性的需要.
     ```  
     
 3. 在生产环境,你需要使用特定版本的Docker CE版本而不是一直使用最新的.
    以下命令会列出所有可用版本(输出只展示了一部分):

     ```  
     
     $ apt-cache madison docker-ce
    docker-ce | 17.12.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
     ```  
    该列表数据依赖于你可用的仓库.选择一个特定的版本去安装.第二列是版本字符串.第三列是仓库名,用来表明该仓库的来源地址和稳定性等级,stable,edge或者test.想要安装特定的版本,只需在包名后面添加(=)后和版本字符串即可.
    
     ```  
     $ sudo apt-get install docker-ce=<VERSION>   
     ``` 
Docker守护进程在安装完成后会自动启动.

4. 通过运行hello-world image 来验证Docker CE是否已经正确安装.
``` 
$ sudo docker run hello-world
  ```    
这个命令会下载一个测试镜像并且在容器中运行它.当容器运行,它会打印提示信息并且退出.

Dcoker CE 已经安装并且运行.docker group 已经创建但是没有用户添加到它上面.你需要通过使用sudo命令去运行Docker命令.继续到[Linux postinstall](https://docs.docker.com/install/linux/linux-postinstall/)去允许未授权的用户能够运行Docker命令和一些可选的配置步骤.

**升级 DOCKER CE**

升级Docker CE,首先同样通过sudo命令更新包索引apt-get update,然后按照按照指令,选择新的版本直接进行安装即可.

**卸载Docker CE**

1. 卸载Docker CE包:
   ```
   
   $ sudo apt-get purge docker-ce
   ```
2. 手动删除镜像，容器等文件(Images, containers, volumes, or customized configuration files)

   ```
   $ sudo rm -rf /var/lib/docker
   ```
   你必须删除所有已编辑的配置文件.
   
   
   


