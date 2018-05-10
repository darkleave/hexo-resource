---
title: SpringBoot应用启动事件
date: 2017-12-17 00:07:42
tags: [SpringBoot,event]
---


**SpringBoot应用启动事件**

<!--more-->


**监听Spring boot应用的事件只需实现ApplicationListener接口来监听对应事件.**

有些事件实际上是在	ApplicationContext	创建前触发的，所以你不能在那些

事件（处理类）中通过	@Bean	注册监听器，只能通

过	SpringApplication.addListeners(…)	或	SpringApplicationBuilder.lis

teners(…)	方法注册.

应用运行时，事件会以下面的次序发送：

1.	在运行开始，但除了监听器注册和初始化以外的任何处理之前，会发送一

个	ApplicationStartedEvent	。

2.	在Environment将被用于已知的上下文，但在上下文被创建前，会发送一

个	ApplicationEnvironmentPreparedEvent	。

3.	在refresh开始前，但在bean定义已被加载后，会发送一

个	ApplicationPreparedEvent	。

4.	在refresh之后，相关的回调处理完，会发送一个	ApplicationReadyEvent	，

表示应用准备好接收请求了。

5.	启动过程中如果出现异常，会发送一个	ApplicationFailedEvent	。

注	通常不需要使用application事件，但知道它们的存在是有用的（在某些场合可能

会使用到），比如，在Spring	Boot内部会使用事件处理各种任务。