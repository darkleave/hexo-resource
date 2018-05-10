---
title: Spring WebMVC框架
date: 2017-12-17 00:07:42
tags: [SpringBoot,MVC]
---

**Spring WebMVC框架**

<!--more-->

Spring	Web	MVC框架（通常简称为"SpringMVC"）是一个富“模型，视图，控制器”web框架，	允许用户创建特定的	 @Controller	或	 @RestController		beans来处理传入的HTTP请求，通过@RequestMapping注解可以将控制器中的方法映射到相应的HTTP请求。
示例：

    @RestController
    @RequestMapping(value="/users")
    public	class	MyRestController	{
    				@RequestMapping(value="/{user}",	method=RequestMethod.GET)
    				public	User	getUser(@PathVariable	Long	user)	{
    								//	...
    				}
    				@RequestMapping(value="/{user}/customers",	method=RequestMet
    hod.GET)
    				List<Customer>	getUserCustomers(@PathVariable	Long	user)	{
    								//	...
    				}
    				@RequestMapping(value="/{user}",	method=RequestMethod.DELETE
    )
    				public	User	deleteUser(@PathVariable	Long	user)	{
    								//	...
    				}
    }

Spring	MVC是Spring框架的核心部分，详细信息可以参考reference
documentation，spring.io/guides也有一些可用的指导覆盖Spring	MVC。

##	Spring	MVC自动配置##

Spring	Boot为Spring	MVC提供的auto-configuration适用于大多数应用，并在
Spring默认功能上添加了以下特性：

1.	 引入	 ContentNegotiatingViewResolver	和	 BeanNameViewResolver	
beans。
2.	 对静态资源的支持，包括对WebJars的支持。
3.	 自动注册	 Converter	，	 GenericConverter	，	 Formatter		beans。
4.	 对	 HttpMessageConverters	的支持。
5.	 自动注册	 MessageCodeResolver	。
6.	 对静态	 index.html	的支持。
7.	 对自定义	 Favicon	的支持。
8.	 自动使用	 ConfigurableWebBindingInitializer		bean。

如果保留Spring	Boot	MVC特性，你只需添加其他的MVC配置（拦截器，格式化处理器，视图控制器等）。你可以添加自己的	 WebMvcConfigurerAdapter	类型的@Configuration类，而不需要注解@EnableWebMvc。如果希望使用自定义的	 RequestMappingHandlerMapping	，RequestMappingHandlerAdapter	，或	 ExceptionHandlerExceptionResolver，你可以声明一个WebMvcRegistrationsAdapter	实例提供这些组件。

如果想全面控制Spring	MVC，你可以添加自己的	 @Configuration	，并使用	 @EnableWebMvc	注解。

##HttpMessageConverters##

Spring	MVC使用	 HttpMessageConverter	接口转换HTTP请求和响应，合适的默认配置可以开箱即用，例如对象自动转换为JSON（使用Jackson库）或XML（如果Jackson XML扩展可用，否则使用JAXB），字符串默认使用	 UTF-8	编码。
可以使用Spring	Boot的	 HttpMessageConverters	类添加或自定义转换类：

    import	org.springframework.boot.autoconfigure.web.HttpMessageCon
    verters;
    import	org.springframework.context.annotation.*;
    import	org.springframework.http.converter.*;
    @Configuration
    public	class	MyConfiguration	{
    				@Bean
    				public	HttpMessageConverters	customConverters()	{
    								HttpMessageConverter<?>	additional	=	...
    								HttpMessageConverter<?>	another	=	...
    								return	new	HttpMessageConverters(additional,	another);
    				}
    }
    
上下文中出现的所有	 HttpMessageConverter		bean都将添加到converters列表，你可以通过这种方式覆盖默认的转换器列表（converters）。    

##自定义JSON序列化器和反序列化器##

如果使用Jackson序列化，反序列化JSON数据，你可能想编写自己的	 JsonSerializer	和	 JsonDeserializer	类。自定义序列化器（serializers）通常通过Module注册到Jackson，但Spring	Boot提供了	 @JsonComponent	注解这一替代方式，它能轻松的将序列化器注册为Spring	Beans。

##MessageCodesResolver##

Spring	MVC有一个实现策略，用于从绑定的errors产生用来渲染错误信息的错误码：	 MessageCodesResolver	。SpringBoot会自动为你创建该实现，只要设置spring.mvc.message-codes-resolver.format	属性为	 PREFIX_ERROR_CODE	或	 POSTFIX_ERROR_CODE	（具体查看	 DefaultMessageCodesResolver.Format	枚举值）。

##静态内容##

默认情况下，Spring	Boot从classpath下的	 /static	（	 /public	，	 /resources	或	 /META-INF/resources	）文件夹，或从ServletContext	根目录提供静态内容。

这是通过Spring	MVC的	 ResourceHttpRequestHandler	实现的，你可以自定义WebMvcConfigurerAdapter	并覆写addResourceHandlers	方法来改变该行为（加载静态文件）。

在单机web应用中，容器会启动默认的servlet，并用它加载	 ServletContext	根目录下的内容以响应那些Spring不处理的请求。大多数情况下这都不会发生（除非你修改默认的MVC配置），因为Spring总能够通过	 DispatcherServlet	处理这些请求。

你可以设置	 spring.resources.staticLocations	属性自定义静态资源的位置（配置一系列目录位置代替默认的值），如果你这样做，默认的欢迎页面将从自定义位置加载，所以只要这些路径中的任何地方有一个	 index.html	，它都会成为应用的主页。

此外，除了上述标准的静态资源位置，有个例外情况是Webjars内容。任何在 /webjars/**	路径下的资源都将从jar文件中提供，只要它们以Webjars的格式
打包。

注	如果你的应用将被打包成jar，那就不要使用	 src/main/webapp文件夹。尽管该文件夹是通常的标准格式，但它仅在打包成war的情况下起作用，在打包成jar时，多数构建工具都会默认忽略它。

Spring	Boot也支持SpringMVC提供的高级资源处理特性，可用于清除缓存的静态资源或对WebJar使用版本无感知的URLs。

如果想使用针对WebJars版本无感知的URLs（version	agnostic），只需要添
加	 webjars-locator	依赖，然后声明你的Webjar。以jQuery为例，	 "/webjars/jquery/dist/jquery.min.js"	实际
为	 "/webjars/jquery/x.y.z/dist/jquery.min.js"	，	 x.y.z为Webjar的版
本。

注	如果使用JBoss，你需要声明	 webjars-locator-jboss-vfs	依赖而不是	 webjars-locator	，否则所有的Webjars将解析为	 404	。

以下的配置为所有的静态资源提供一种缓存清除（cache	busting）方案，实际上是
将内容hash添加到URLs中，比如	 <link	href="/css/spring-
2a2d595e6ed9a0b24f027f2b63b134d6.css"/>	：

    spring.resources.chain.strategy.content.enabled=true
    spring.resources.chain.strategy.content.paths=/**
    
注	实现该功能的是	 ResourceUrlEncodingFilter	，它在模板运行期会重写资源链接，Thymeleaf，Velocity和FreeMarker会自动配置该filter，JSP需要手动配置。
其他模板引擎还没自动支持，不过你可以使用ResourceUrlProvider自定义模块宏或
帮助类。
当使用比如JavaScript模块加载器动态加载资源时，重命名文件是不行的，这也是提供其他策略并能结合使用的原因。下面是一个"fixed"策略，在URL中添加一个静态version字符串而不需要改变文件名：    


    spring.resources.chain.strategy.content.enabled=true
    spring.resources.chain.strategy.content.paths=/**
    spring.resources.chain.strategy.fixed.enabled=true
    spring.resources.chain.strategy.fixed.paths=/js/lib/
    spring.resources.chain.strategy.fixed.version=v12
    
使用以上策略，JavaScript模块加载器加载	 "/js/lib/"下的文件时会使用一个固定的版本策略	 "/v12/js/lib/mymodule.js"	，其他资源仍旧使用内容hash的方式	 <link	href="/css/spring-
2a2d595e6ed9a0b24f027f2b63b134d6.css"/>	。

查看ResourceProperties获取更多支持的选项。
注	该特性在一个专门的博文和Spring框架参考文档中有透彻描述。    

##ConfigurableWebBindingInitializer##

Spring	MVC使用	 WebBindingInitializer	为每个特殊的请求初始化相应的WebDataBinder	，如果你创建自己的	 ConfigurableWebBindingInitializer @Bean	，Spring	Boot会自动配置Spring	MVC使用它。

##模板引擎##

正如REST	web服务，你也可以使用Spring	MVC提供动态HTML内容。Spring	MVC
支持各种各样的模板技术，包括Velocity,	FreeMarker和JSPs，很多其他的模板引擎也提供它们自己的Spring	MVC集成。

Spring	Boot为以下的模板引擎提供自动配置支持：
1.	 FreeMarker
2.	 Groovy
3.	 Thymeleaf
4.	 Velocity（1.4已不再支持）
5.	 Mustache
6.	 
注：由于在内嵌servlet容器中使用JSPs存在一些已知的限制，所以建议尽量不使用它们。

使用以上引擎中的任何一种，并采用默认配置，则模块会从src/main/resources/templates	自动加载。

注：IntelliJ	IDEA根据你运行应用的方式会对classpath进行不同的排序。
在IDE里通过main方法运行应用，跟从Maven，或Gradle，或打包好的jar中运行相比会导致不同的顺序，这可能导致SpringBoot不能从classpath下成功地找到模板。
如果遇到这个问题，你可以在IDE里重新对classpath进行排序，将模块的类和资源放到第一位。
或者，你可以配置模块的前缀为	 classpath*:/templates/	，这样会查找
classpath下的所有模板目录。

##错误处理##

Spring	Boot默认提供一个	 /error	映射用来以合适的方式处理所有的错误，并将它注册为servlet容器中全局的	错误页面。
对于机器客户端（相对于浏览器而言，浏览器偏重于人的行为），它会产生一个具有详细错误，HTTP状态，异常信息的JSON响应。对于浏览器客户端，它会产生一个白色标签样式（whitelabel）的错误视图，该视图将以HTML格式显示同样的数据（可以添加一个解析为'error'的View来自定义它）。
为了完全替换默认的行为，你可以实现	 ErrorController	，并注册一个该类型的bean定义，或简单地添加一个	 ErrorAttributes	类型的bean以使用现存的机制，只是替换显示的内容。

注	 	 BasicErrorController	可以作为自定义	 ErrorController	的基类，如果你想添加对新context	type的处理（默认处理	 text/html	），这会很有帮助。
你只需要继承	 BasicErrorController，添加一个public方法，并注解带有	 produces	属性的	 @RequestMapping，然后创建该新类型的bean。
你也可以定义一个@ControllerAdvice去自定义某个特殊controller或exception类型的JSON文档：

    @ControllerAdvice(basePackageClasses	=	FooController.class)
    public	class	FooControllerAdvice	extends	ResponseEntityException
    Handler	{
    				@ExceptionHandler(YourException.class)
    				@ResponseBody
    				ResponseEntity<?>	handleControllerException(HttpServletReque
    st	request,	Throwable	ex)	{
    								HttpStatus	status	=	getStatus(request);
    								return	new	ResponseEntity<>(new	CustomErrorType(status.v
    alue(),	ex.getMessage()),	status);
    				}
    				private	HttpStatus	getStatus(HttpServletRequest	request)	{
    								Integer	statusCode	=	(Integer)	request.getAttribute("jav
    ax.servlet.error.status_code");
    								if	(statusCode	==	null)	{
    												return	HttpStatus.INTERNAL_SERVER_ERROR;
    								}
    								return	HttpStatus.valueOf(statusCode);
    				}
    }
    
在以上示例中，如果跟	 FooController相同package的某个controller抛出	 YourException	，一个	 CustomerErrorType	类型的POJO的json展示将代替	 ErrorAttributes	展示。

自定义错误页面

如果想为某个给定的状态码展示一个自定义的HTML错误页面，你需要将文件添加到	 /error	文件夹下。错误页面既可以是静态HTML（比如，任何静态资源文件夹下添加的），也可以是使用模板构建的，文件名必须是明确的状态码或一系列标签。

例如，映射	 404	到一个静态HTML文件，你的目录结构可能如下：    

    src/
    	+-	main/
    			+-	java/
    			|			+	<source	code>
    			+-	resources/
    						+-	public/
    							+-	error/
    								|	+-	404.html
    							+-	<other	public	assets>    
    							
    							
使用FreeMarker模板映射所有	 5xx	错误，你需要如下的目录结构：

    src/
    	+-	main/
    			+-	java/
    			|			+	<source	code>
    			+-	resources/
    					+-	templates/
    							+-	error/
    						    |			+-	5xx.ftl
    							+-	<other	templates>
    							
对于更复杂的映射，你可以添加实现	 ErrorViewResolver接口的beans：


    public	class	MyErrorViewResolver	implements	ErrorViewResolver	{
    				@Override
    	public	ModelAndViewresolveErrorView(HttpServletRequest	requ
    est,HttpStatus	status,Map<String,	Object>	model)	{
    	//	Use	the	request	or	status	to	optionally	return	a	Mode
    lAndView
    					return	...
    	}
    }
    
你也可以使用Spring	MVC特性，比如@ExceptionHandler方法和@ControllerAdvice，ErrorController	将处理所有未处理的异常。

映射Spring	MVC以外的错误页面

对于不使用Spring	MVC的应用，你可以通过	 ErrorPageRegistrar	接口直接注册	 ErrorPages。
该抽象直接工作于底层内嵌servlet容器，即使你没有SpringMVC的DispatcherServlet	，它们仍旧可以工作。    

    @Bean
    public	ErrorPageRegistrar	errorPageRegistrar(){
    				return	new	MyErrorPageRegistrar();
    }
    //	...
    private	static	class	MyErrorPageRegistrar	implements	ErrorPageRegistrar	{
    				@Override
    				public	void	registerErrorPages(ErrorPageRegistry	registry)	{
    								registry.addErrorPages(new	ErrorPage(HttpStatus.BAD_REQU
    EST,	"/400"));
    				}
    }
    

注.如果你注册一个	 ErrorPage	，该页面需要被一个	 Filter	处理（在一些非Spring	web框架中很常见，比如Jersey，Wicket），那么该	 Filter	需要明确注册为一个	 ERROR	分发器（dispatcher），例如：    

    @Bean
    public	FilterRegistrationBean	myFilter()	{
    				FilterRegistrationBean	registration	=	new	FilterRegistrationBean();
    				registration.setFilter(new	MyFilter());
    				...
    				registration.setDispatcherTypes(EnumSet.allOf(DispatcherType
    .class));
    				return	registration;
    }

（默认的	 FilterRegistrationBean	不包含	 ERROR		dispatcher类型）。

WebSphere应用服务器的错误处理当部署到一个servlet容器时，Spring	Boot通过它的错误页面过滤器将带有错误状态的请求转发到恰当的错误页面。

request只有在response还没提交时才能转发（forwarded）到正确的错误页面，而WebSphere应用服务器8.0及后续版本默认情况会在servlet方法成功执行后提交response，你需要设置	 com.ibm.ws.webcontainer.invokeFlushAfterService	属性为	 false	来关闭该行为。    


##Spring HATEOAS##

如果正在开发基于超媒体的RESTful	API，你可能需要Spring	HATEOAS，而Spring
Boot会为其提供自动配置，这在大多数应用中都运作良好。

自动配置取代了	 @EnableHypermediaSupport	，只需注册一定数量的beans就能轻松构建基于超媒体的应用，这些beans包括	 LinkDiscoverers	（客户端支持），	 ObjectMapper	（用于将响应编排为想要的形式）。

ObjectMapper	可以根据	 spring.jackson.*	属性或	 Jackson2ObjectMapperBuilder		bean进行自定义。

通过注解	 @EnableHypermediaSupport	，你可以控制Spring	HATEOAS的配置，
但这会禁用上述	 ObjectMapper	的自定义功能。

##CORS支持##

跨域资源共享（CORS）是一个大多数浏览器都实现了的W3C标准，它允许你以灵活的方式指定跨域请求如何被授权，而不是采用那些不安全，性能低的方式，比如IFRAME或JSONP。

从4.2版本开始，Spring MVC对CORS提供开箱即用的支持。不用添加任何特殊配置，只需要在Spring	Boot应用的controller方法上注解	 @CrossOrigin	，并添加CORS配置。
通过注册一个自定义	 addCorsMappings(CorsRegistry)	方法的WebMvcConfigurer		bean可以指定全局CORS配置：

    @Configuration
    public	class	MyConfiguration	{
    			@Bean
    			public	WebMvcConfigurer	corsConfigurer()	{
    								return	new	WebMvcConfigurerAdapter()	{
    												@Override
    			public	void	addCorsMappings(CorsRegistry	registry)	{
    																registry.addMapping("/api/**");
    												}
    								};
    			}
    }
    
    
    