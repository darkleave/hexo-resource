---
title: 使用CommandLineRunner或ApplicationRunner
date: 2017-12-17 00:07:42
tags: [SpringBoot]
---

**使用CommandLineRunner或ApplicationRunner**

<!--more-->


如果需要在	 SpringApplication	启动后执行一些特殊的代码，你可以实
现	 ApplicationRunner	或	 CommandLineRunner	接口，这两个接口工作方式相
同，都只提供单一的	 run	方法，该方法仅在	 SpringApplication.run(…)	完成
之前调用。
	 CommandLineRunner	接口能够访问string数组类型的应用参数，
而	 ApplicationRunner	使用的是上面描述过的	 ApplicationArguments	接口：

    import	org.springframework.boot.*
    import	org.springframework.stereotype.*
    @Component
    public	class	MyBean	implements	CommandLineRunner	{
    				public	void	run(String...	args)	{
    								//	Do	something...
    				}
    }
如果某些定义的	 CommandLineRunner	或	 ApplicationRunner		beans需要以特定
的顺序调用，你可以实现	 **org.springframework.core.Ordered**	接口或使
用	 **org.springframework.core.annotation.Order**	注解。




