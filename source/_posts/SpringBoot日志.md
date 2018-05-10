---
title: SpringBoot日志
date: 2017-12-17 00:07:42
tags: [SpringBoot,log]
---


**SpringBoot日志**

<!--more-->

##日志格式##

Spring	Boot默认的日志输出格式如下：

    2014-03-05	10:57:51.112		INFO	45469	---	[main]org.apache.catalina.core.StandardEngine		:	Starting	Servlet	Engine:	Apache	Tomcat/7.0.522014-03-05	10:57:51.253		INFO	45469	---	[ost-startStop-1]	o.a.c.
    c.C.[Tomcat].[localhost].[/]							:	Initializing	Spring	embedde
    d	WebApplicationContext
    2014-03-05	10:57:51.253		INFO	45469	---	[ost-startStop-1]	o.s.we
    b.context.ContextLoader												:	Root	WebApplicationContext:
    	initialization	completed	in	1358	ms
    2014-03-05	10:57:51.698		INFO	45469	---	[ost-startStop-1]	o.s.b.
    c.e.ServletRegistrationBean								:	Mapping	servlet:	'dispatche
    rServlet'	to	[/]
    2014-03-05	10:57:51.702		INFO	45469	---	[ost-startStop-1]	o.s.b.
    c.embedded.FilterRegistrationBean		:	Mapping	filter:	'hiddenHttp
    MethodFilter'	to:	[/*]


输出的节点（items）如下：

 1.  日期和时间	-	精确到毫秒，且易于排序。
 2.  日志级别	-	 	 ERROR	,	 	 WARN	,	 	 INFO	,	 	 DEBUG		或	 	 TRACE	。
 3.  Process	ID。
 4.  	 ---	分隔符，用于区分实际日志信息开头。
 5.  线程名	-	包括在方括号中（控制台输出可能会被截断）。
 6.  日志名	-	通常是源class的类名（缩写）。
 7.  日志信息。
注	Logback没有	 FATAL	级别，它会映射到	 ERROR	。

##控制台输出##

默认的日志配置会在写日志消息时将它们回显到控制台，级别为	 ERROR	,WARN	和	 INFO	的消息会被记录。你可以在启动应用时，通过	 --debug	标识开启控制台的DEBUG级别日志记录，也可以在application.properties	中指定	 debug=true	。

    $	java	-jar	myapp.jar	--debug
    
当debug模式启用时，一系列核心loggers（内嵌容器，Hibernate，Spring	Boot
等）记录的日志会变多，但不会输出所有的信息。
相应地，你可以在启动应用时，通过	 --trace	（或在	 application.properties	设置	 trace=true	）启用"trace"模式，该模式能够追踪核心loggers（内嵌容器，Hibernate生成的schema，Spring全部的portfolio）的所有日志信息。    

##文件输出##

默认情况下，SpringBoot只会将日志记录到控制台，而不写进日志文件，如果需要，你可以设置	 logging.file	或	 logging.path	属性（例如	 application.properties	）。
下表展示如何组合使用	 logging.*	：

|logging.file |logging.path| 示例| 描述|
|:----:|:----:|:----:|:----:|
|(none) |(none) |  |只记录到控制台|
|Specific	file |(none)| my.log| 写到特定的日志文件，名称可以是精确的位置或相对于当前目录|
|(none)|Specific directory|/var/log|写到特定目录下的	 spring.log	里，名称可以是精确的位置或相对于当前目录|

日志文件每达到10M就会被分割，跟控制台一样，默认记录	 ERROR	,WARN	和	 INFO	级别的信息。

##日志级别##

所有Spring	Boot支持的日志系统都可以在Spring	 	 Environment	中设置级别（	 application.properties	里也一样），设置格式为'logging.level.*=LEVEL'，其中	 LEVEL	是	 TRACE	,	 	 DEBUG	,	 	 INFO	,	 	 WARN	,	 	 ERROR	,	 	 FATAL	,	 	 OFF	之一：
以下是	 application.properties	示例：

    logging.level.root=WARN
    logging.level.org.springframework.web=DEBUG
    logging.level.org.hibernate=ERROR
    
注	默认情况，Spring	Boot会重新映射Thymeleaf的	 INFO	信息到	 DEBUG	级别，这
能减少标准日志输出的噪声。查看LevelRemappingAppender可以按自己的配置设
置映射。

