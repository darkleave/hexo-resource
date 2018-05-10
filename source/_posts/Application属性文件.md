﻿---
title: Application属性文件
date: 2017-12-17 00:07:42
tags: [SpringBoot]
---

**Application属性文件**

<!--more-->


SpringApplication	将从以下位置加载	 application.properties	文件，并把
它们添加到Spring	 	 Environment	中：

1.	 当前目录下的	 /config	子目录。

2.	 当前目录。

3.	 classpath下的	 /config	包。

4.	 classpath根路径（root）。

该列表是按优先级排序的（列表中位置高的路径下定义的属性将覆盖位置低的）。
注	你可以使用YAML（'.yml'）文件替代'.properties'。

如果不喜欢将	 application.properties	作为配置文件名，你可以通过指
定	 spring.config.name	环境属性来切换其他的名称，也可以使
用	 spring.config.location	环境属性引用一个明确的路径（目录位置或文件路
径列表以逗号分割）。

    $	java	-jar	myproject.jar	--spring.config.name=myproject
    或
    $	java	-jar	myproject.jar	--spring.config.location=classpath:/de
    fault.properties,classpath:/override.properties

注	在初期需要根据	 spring.config.name	和	 spring.config.location	决定加
载哪个文件，所以它们必须定义为environment属性（通常为OS	env，系统属性或
命令行参数）。

如果	 spring.config.location	包含目录（相对于文件），那它们应该以	 /	结尾
（在被加载前，	 spring.config.name	关联的名称将被追加到后面，包括profile-
specific的文件名）。	 spring.config.location	下定义的文件使用方法跟往常一
样，没有profile-specific变量支持的属性，将被profile-specific的属性覆盖。

不管	 spring.config.location	配置什么值，默认总会按
照	 classpath:,classpath:/config,file:,file:config/	的顺序进行搜索，优
先级由低到高，也就是	 file:config/	获胜。
如果你指定自己的位置，它们会优先于所有的默认位置（locations），并使用相同的由低到高的优先级顺序。
那样，你就可以在	 application.properties	为应用设置默认值，然后在运行的时候使
用不同的文件覆盖它，同时保留默认配置。

注	如果使用环境变量而不是系统属性，需要注意多数操作系统的key名称不允许以
句号分割（period-separated），但你可以使用下划线（underscores）代替（比
如，使用	 SPRING_CONFIG_NAME	代替	 spring.config.name	）。
注	如果应用运行在容器中，那么JNDI属性（java:comp/env）或servlet上下文初始
化参数可以用来代替环境变量或系统属性，当然也可以使用环境变量或系统属性。




