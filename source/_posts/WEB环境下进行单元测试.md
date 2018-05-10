---
title: WEB环境下进行单元测试
date: 2017-12-17 00:07:42
tags: [SpringBoot]
---

**WEB环境下进行单元测试**

<!--more-->

 SpringApplication	将尝试为你创建正确类型的	 ApplicationContext	，默认
情况下，根据你开发的是否为web应用决定使
用	 AnnotationConfigApplicationContext	或	 AnnotationConfigEmbeddedWeb
ApplicationContext	。
用于确定是否为web环境的算法相当简单（判断是否存在某些类），你可以使
用	 setWebEnvironment(boolean	webEnvironment)	覆盖默认行为。
通过调用	 setApplicationContextClass(…)	，你可以完全控
制	 ApplicationContext	的类型。
注	在Junit测试中使用	 SpringApplication	，调
用	 **setWebEnvironment(false)**	是很有意义的。




