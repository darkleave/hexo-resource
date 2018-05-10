---
title: spring boot简介
date: 2017-12-02 22:22:54
tags: [spring,框架]
---

**spring boot**

spring boot 为所有spring框架开发者提供一种更加易于理解，更加便捷高效的开发方式；
通过提供更为直观的spring平台和第三方依赖库，只需要极其少量的spring配置，便能部署运行spring boot应用。

<!--more-->


1.环境要求

Spring Boot 2.0.0.BUILD-SNAPSHOT 需要 Java 8 以及 Spring Framework 5.0.2.RELEASE 或者以上版本. 
当使用maven或者gradle构建spring boot时需要 Maven 3.2+ 或者 Gradle 4及其以上版本.


2.spring boot安装

2.1使用maven 构建spring boot应用

pom.xml配置:

	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
		<modelVersion>4.0.0</modelVersion>

		<groupId>com.example</groupId>
		<artifactId>myproject</artifactId>
		<version>0.0.1-SNAPSHOT</version>

		<!-- Inherit defaults from Spring Boot -->
		<parent>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-parent</artifactId>
			<version>2.0.0.BUILD-SNAPSHOT</version>
		</parent>

		<!-- Add typical dependencies for a web application -->
		<dependencies>
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-web</artifactId>
			</dependency>
		</dependencies>

		<!-- Package as an executable jar -->
		<build>
			<plugins>
				<plugin>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-maven-plugin</artifactId>
				</plugin>
			</plugins>
		</build>

		<!-- Add Spring repositories -->
		<!-- (you don't need this if you are using a .RELEASE version) -->
		<repositories>
			<repository>
				<id>spring-snapshots</id>
				<url>http://repo.spring.io/snapshot</url>
				<snapshots><enabled>true</enabled></snapshots>
			</repository>
			<repository>
				<id>spring-milestones</id>
				<url>http://repo.spring.io/milestone</url>
			</repository>
		</repositories>
		<pluginRepositories>
			<pluginRepository>
				<id>spring-snapshots</id>
				<url>http://repo.spring.io/snapshot</url>
			</pluginRepository>
			<pluginRepository>
				<id>spring-milestones</id>
				<url>http://repo.spring.io/milestone</url>
			</pluginRepository>
		</pluginRepositories>
	</project>
	
2.2安装Spring Boot CLI

The Spring Boot CLI (Command Line Interface)是一个用于快速建立spring原型的命令行工具.
通过它你能运行Groovy scripts,使用熟悉的类java语法。

使用CLI来运行spring boot不是必须的,但它是使spring应用运行起来的最快方法.

2.2.1 手动安装

你可以在spring 软件仓库下载Spring CLI

[spring-boot-cli-2.0.0.BUILD-SNAPSHOT-bin.zip](https://repo.spring.io/snapshot/org/springframework/boot/spring-boot-cli/2.0.0.BUILD-SNAPSHOT/spring-boot-cli-2.0.0.BUILD-SNAPSHOT-bin.zip)


[spring-boot-cli-2.0.0.BUILD-SNAPSHOT-bin.tar.gz](https://repo.spring.io/snapshot/org/springframework/boot/spring-boot-cli/2.0.0.BUILD-SNAPSHOT/spring-boot-cli-2.0.0.BUILD-SNAPSHOT-bin.tar.gz)

2.2.2 通过SDKMAN安装

SDKMAN!(The Software Development Kit Manager)可以用来管理不同版本的二进制sdks,包括 Groovy和Spring Boot CLI.
从[sdkman.io](http://sdkman.io/)获取SDKMAN!并且通过以下命令来安装Spring Boot

	$ sdk install springboot
	$ spring --version
	Spring Boot v2.0.0.BUILD-SNAPSHOT
	
上述的安装方式都会在本地建立一个名为dev的Spring实例,它指向你的安装路径,所以你每次重建Spring Boot时,
spring 都会更新到最新.

你能通过以下命令看到它是如何进行的:

$ sdk ls springboot

	================================================================================
	Available Springboot Versions
	================================================================================
	> + dev
	* 2.0.0.BUILD-SNAPSHOT

	================================================================================
	+ - local version
	* - installed
	> - currently in use
	================================================================================	

3.开发你首个Spring Boot 应用

这个章节描述了如何区开发一个\"五脏俱全\"的Spring Boot \"Hello World!\"应用.
我们使用Maven来构建这个项目,因为大部分IDES都支持它.

[spring.io](https://spring.io/)web网站包含很多"Getting Started"的spring boot教程,
如果你需要解决一些特定的问题,首先查看这里.

开始之前首先检查jdk版本和maven版本是否满足要求

java -version

mvn -v

pom.xml 配置如下:


	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
		<modelVersion>4.0.0</modelVersion>

		<groupId>com.example</groupId>
		<artifactId>myproject</artifactId>
		<version>0.0.1-SNAPSHOT</version>

		<parent>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-parent</artifactId>
			<version>2.0.0.BUILD-SNAPSHOT</version>
		</parent>

		<!-- Additional lines to be added here... -->

		<!-- (you don't need this if you are using a .RELEASE version) -->
		<repositories>
			<repository>
				<id>spring-snapshots</id>
				<url>http://repo.spring.io/snapshot</url>
				<snapshots><enabled>true</enabled></snapshots>
			</repository>
			<repository>
				<id>spring-milestones</id>
				<url>http://repo.spring.io/milestone</url>
			</repository>
		</repositories>
		<pluginRepositories>
			<pluginRepository>
				<id>spring-snapshots</id>
				<url>http://repo.spring.io/snapshot</url>
			</pluginRepository>
			<pluginRepository>
				<id>spring-milestones</id>
				<url>http://repo.spring.io/milestone</url>
			</pluginRepository>
		</pluginRepositories>
	</project>


3.2 Spring Boot提供了大量的"Starters"开始器使你能在classpath路径添加jars.
我们普通的例子应用已经在POM的父节点使用 spring-boot-starter-parent .

spring-boot-starter-parent是一个提供了众多有用Maven默认配置的特殊启动器.
它同样提供了dependency-management配置使你能够为一些dependencies依赖省略 version 配置.

但你开发其它特殊类型的应用时可能需要使用到其它启动器(Starters),例如,当我们开发一个web应用时,
我们添加spring-boot-starter-web 依赖.在那之前,我们可以通过以下命令查看依赖树:

mvn dependency:tree

通过mvn dependency:tree命令我们可以查看项目的依赖树结构,
你可以看到  spring-boot-starter-parent 本身没有提供其它依赖.
通过编辑pom.xml添加必要的依赖

<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
</dependencies>

如果你再次运行 mvn dependency:tree 命令,你就能看到一系列新添加的依赖,
包括tomcat 服务器以及spring boot本身.

3.3 代码编写

为了完成应用，我们首先需要创建一个java文件.
在src/main/java下创建Example.java文件并添加以下代码:


	import org.springframework.boot.*;
	import org.springframework.boot.autoconfigure.*;
	import org.springframework.web.bind.annotation.*;

	@RestController
	@EnableAutoConfiguration
	public class Example {

		@RequestMapping("/")
		String home() {
			return "Hello World!";
		}

		public static void main(String[] args) throws Exception {
			SpringApplication.run(Example.class, args);
		}

	}


3.3.1 @RestController 和 @RequestMapping 注解





	
	
	






	
	
	