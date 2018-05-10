---
title: 使用YAML代替Properties
date: 2017-12-17 00:07:42
tags: [SpringBoot,YAML]
---

**使用YAML代替Properties**

<!--more-->

##SpringBoot支持YAML##
YAML是JSON的一个超集，也是一种方便的定义层次配置数据的格式。只要你
将SnakeYAML	库放到classpath下，	 SpringApplication	就会自动支持YAML，
以作为properties的替换。
注	如果你使用'Starters'，添加	 spring-boot-starter	依赖会自动加载
SnakeYAML。

##加载YAML##

Spring框架提供两个便利的类用于加载YAML文档，YamlPropertiesFactoryBean	会将YAML加载为	 Properties	，	 YamlMapFactoryBean	会将YAML加载为	 Map	。
例如，下面的YAML文档：

    environments:
    				dev:
    								url:	http://dev.bar.com
    								name:	Developer	Setup
    				prod:
    								url:	http://foo.bar.com
    								name:	My	Cool	App

会被转化到这些属性：

    environments.dev.url=http://dev.bar.com
    environments.dev.name=Developer	Setup
    environments.prod.url=http://foo.bar.com
    environments.prod.name=My	Cool	App
    
    
YAML列表被表示成使用[index]间接引用作为属性keys的形式，例如下面的
YAML：

    my:
    			servers:
    							-	dev.bar.com
    							-	foo.bar.com
    
将会转化到这些属性:

    my.servers[0]=dev.bar.com
    my.servers[1]=foo.bar.com

使用Spring	 	 DataBinder	工具集绑定这些属性（这是@ConfigurationProperties	做的事）时，你需要确保目标bean有个	 java.util.List	或	 Set	类型的属性，并且需要提供一个setter或使用可变的值初始化它，比如，下面的代码将绑定上面的属性：

    @ConfigurationProperties(prefix="my")
    public	class	Config	{
    		private	List<String>	servers	=	new	ArrayList<String>();
    		public	List<String>	getServers(){
    				return	this.servers;
    		}
    }
    
##在Spring环境中使用YAML暴露属性##

YamlPropertySourceLoader类能够将YAML作为PropertySource导出到SprigEnvironment	，这允许你使用常用的	@Value注解配合占位符语法访问YAML属
性。

##Multi-profile	YAML文档##

你可以在单个文件中定义多个特定配置（profile-specific）的YAML文档，并通过	 spring.profiles	标示生效的文档，例如：

    server:
    				address:	192.168.1.100
    ---
    spring:
    				profiles:	development
    server:
    				address:	127.0.0.1
    ---
    spring:
    				profiles:	production
    server:
    				address:	192.168.1.120
    				
在以上例子中，如果	 development		profile被激活，	 server.address	属性将是	 127.0.0.1	；如果	 development	和	 production		profiles没有启用，则该属性的值将是	 192.168.1.100	。
在应用上下文启动时，如果没有明确指定激活的profiles，则默认的profiles将生效。所以，在下面的文档中我们为	 security.user.password	设置了一个值，该
值只在"default"	profile中有效：    				


    server:
    		port:	8000
    ---
    spring:
    		profiles:	default
    security:
    		user:
    				password:	weak

然而，在这个示例中，由于没有关联任何profile，密码总是会设置，并且如果有必要的话可以在其他profiles中显式重置：    				
    				
    server:
    		port:	8000
    security:
    		user:
    				password:	weak    				

通过	 !	可以对	 spring.profiles指定的profiles进行取反（negated，跟java中的	 !	作用一样），如果negated和non-negated	profiles都指定一个单一文件，至少需要匹配一个non-negated	profile，可能不会匹配任何negated	profiles。    				
    				
##YAML缺点##    				

YAML文件不能通过	 @PropertySource	注解加载，如果需要使用该方式，那就必须使用properties文件。

##合并YAML列表##

正如上面看到的，所有YAML最终都转换为properties，在通过一个profile覆
盖"list"属性时这个过程可能不够直观（counter	intuitive）。例如，假设有一
个	 MyPojo	对象，默认它的	 name	和	 description	属性都为	 null	，下面我们
将从	 FooProperties	暴露一个	 MyPojo	对象列表（list）：

    @ConfigurationProperties("foo")
    public	class	FooProperties	{
    		private	final	List<MyPojo>	list	=	new	ArrayList<>();
    		public	List<MyPojo>	getList()	{
    				return	this.list;
    		}
    }
    
考虑如下配置：

    foo:
    		list:
    				-	name:	my	name
    						description:	my	description
    ---
    spring:
    		profiles:	dev
    foo:
    		list:
    				-	name:	my	another	name
    				
 如果	 dev		profile没有激活，	 FooProperties.list	将包括一个如上述定义的	 MyPojo	实体，即使	 dev	生效，该	 list	仍旧只包含一个实体（	 name	值为	 my	another	name	，description	值为	 null	）。
此配置不会向该列表添加第二个	 MyPojo	实例，也不会对该项进行合并。
当一个集合定义在多个profiles时，只使用优先级最高的：   	

    foo:
    		list:
    				-	name:	my	name
    						description:	my	description
    				-	name:	another	name
    						description:	another	description
    ---
    spring:
    		profiles:	dev
    foo:
    		list:
    					-	name:	my	another	name
    				
 
 在以上示例中，如果	 dev		profile激活，	 FooProperties.list将包含一个	 MyPojo	实体（	 name	值为	 my	another	name	，	 description	值为	 null	）。   
 
 