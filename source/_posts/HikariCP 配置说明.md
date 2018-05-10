---
title: HikariCP 配置说明
date: 2018-01-30 00:07:42
tags: [pool]
---

Fast, simple, reliable. HikariCP is a "zero-overhead" production ready JDBC connection pool. At roughly 130Kb, the library is very light.

<!--more-->

[官网连接](https://github.com/darkleave/HikariCP)
## 必要配置
1. dataSourceClassName 数据源驱动名
2. jdbcUrl jdbc数据库连接
3. username 用户名
4. password 密码

其中dataSourceClassName和jdbcUrl二选一，当使用比较老版本的驱动时，需要同时设置jdbcUrl和driverClassName，dataSourceClassName则不用进行设置

## 可选配置

1. 常用属性
   * connectionTimeout 连接超时时间,最小时间为250 ms，默认为30000ms(30秒)
   * idleTimeout 空闲超时时间，这个属性用来控制空闲连接允许保留在连接池中的最大时间，这个属性只有在minimumIdle（最小空闲连接数）小于maximumPoolSize(最大连接数)时才会生效,空闲连接断开会有15s-30s的延迟变动时间.在这个超时时间之前空闲连接永远不会断开,当连接池达到minimumIdle,连接将永远不会断开，即使处于闲置状态.值为0表示空闲连接将永远不会从连接池中移除，最小值为10000ms （10s),默认值为600000(10min)

   * maxLifetime 最大生命周期.这个属性用来控制连接池中连接的最大生命周期,一个使用中的连接永远不会被断开,只有当它处于关闭状态然后才会被移除.推荐设置比任何数据库或基础设施规定的连接时间限制少至少30秒。 值为0表示没有最大寿命（无限寿命）， 默认：1800000（30分钟）,由于HikariCP的housekeeper默认每30s运行一次,以维护minimumIdle最小空闲连接数，它可能添加新连接或者断开空闲连接，所以你必须设置maxLifetime属性比（mysql)wait_timeout时间少一些来避免 broken connection / exceptions.意思就是说比如mysql wait_timeout为10min,此时有一个连接由于达到超时时间，mysql主动断开了连接，而HakariCP仍然持有此连接，如果再使用此连接去请求数据库则会发生异常,设置maxLifetime最大生命周期比wait_timeout少30s后,就能确保再housekeeper运行期间提前断开此连接，避免发生异常.
   * connectionTestQuery 连接测试查询,如果你的驱动支持jdbc4，则不需要设置此属性，这个属性是为那些不支持Connection.isValid() API的古董级驱动准备的，这是一个查询，用来确保所有请求得到的连接都是alive有效的，尝试不设置这个属性运行连接池，如果你的驱动不支持jdbc4，HikariCP会有错误日志提示.Default:None
   * minimumIdle 最小空闲连接，当空闲连接小于这个值并且总连接数小于maximumPoolSize（最大连接数)HikariCP会尽可能快速有效率地创建额外的连接，然而为了最大限度地提高性能和响应能力，不建议设置这个值，而是用固定大小的连接池取代.Default:与maximumPoolSize相同
   * maximumPoolSize 最大连接池数量，包括使用中和空闲的连接，当达到最大连接池数量时，再尝试获取连接，只能得到connectionTimeout 超时信息.Default:10.
   * metricRegistry 度量注册, Default: none [参考链接](https://www.jianshu.com/p/070f615dfb57)
   * healthCheckRegistry 健康检查注册  Default: none
   * poolName 连接池名字,一般用于日志输出 Default: auto-generated
2. 不常使用
   * initializationFailTimeout 初始化失败超时时间 Default: 1
   * isolateInternalQueries 是否隔离默认查询 Default: 1
   * allowPoolSuspension 是否允许连接池暂停 Default: false
   * readOnly 连接是否只读 Default: false
   * registerMbeans 是否注册JMX Management Beans Default: false
   * catalog 目录服务
   * connectionInitSql 连接初始化sql Default: none
   * driverClassName 驱动名称  Default: none
   * transactionIsolation 事务隔离 Default: driver default
   * validationTimeout 验证超时时间 Default: 5000 
   * leakDetectionThreshold 最低发现阈值 Default: 0
   * dataSource 数据源 Default: none
   * schema 架构  Default: driver default
   * threadFactory 线程工厂 Default: none 
   * scheduledExecutor 计划执行器 Default: none

## Statement Cache

Many connection pools, including Apache DBCP, Vibur, c3p0 and others offer PreparedStatement caching. HikariCP does not. Why?
许多连接池，包括Aache DBCP,Vibur,c3p0 等都是提供PreparedStatement caching.HikariCP并不这样做，为什么？

At the connection pool layer PreparedStatements can only be cached per connection. If your application has 250 commonly executed queries and a pool of 20 connections you are asking your database to hold on to 5000 query execution plans -- and similarly the pool must cache this many PreparedStatements and their related graph of objects.

在连接池中每个连接只能缓存各自的PreparedStatements对象.如果你的应用有250个要执行的普通查询和一个20个连接的连接池，然后你需要不间断地请求你的数据库去完成一个5000查询的执行计划，显然你的连接池必须缓存这所有的PreparedStatements对象和它们相关联的表对象.

Most major database JDBC drivers already have a Statement cache that can be configured, including PostgreSQL, Oracle, Derby, MySQL, DB2, and many others. JDBC drivers are in a unique position to exploit database specific features, and nearly all of the caching implementations are capable of sharing execution plans across connections. This means that instead of 5000 statements in memory and associated execution plans, your 250 commonly executed queries result in exactly 250 execution plans in the database. Clever implementations do not even retain PreparedStatement objects in memory at the driver-level but instead merely attach new instances to existing plan IDs.

许多主流的数据库，它们的jdbc驱动已经有了一个可配置的Statement缓存，包括PostgreSQL, Oracle, Derby, MySQL, DB2等等.JDBC驱动是唯一能利用数据库特定属性的方式，并且近乎所有的缓存实现都可以通过连接共享执行计划.这意味着你的250个普通查询结果在数据库中就是250个执行计划，而不是存储在内存中的5000个statements及其相关联的执行计划.聪明的缓存实现在驱动这一级别并不保持PreparedStatement对象在内存当中，而是为已存在的计划创建新的PreparedStatement实例.

Using a statement cache at the pooling layer is an anti-pattern, and will negatively impact your application performance compared to driver-provided caches.

在连接池层使用statement缓存是一个反面教材,并且相较驱动提供的缓存会对你的应用性能造成更大的消极影响.

## Log Statement Text / Slow Query Logging

Like Statement caching, most major database vendors support statement logging through properties of their own driver. This includes Oracle, MySQL, Derby, MSSQL, and others. Some even support slow query logging. For those few databases that do not support it, several options are available. We have received a report that p6spy works well, and also note the availability of log4jdbc and jdbcdslog-exp.

就像Statement缓存，许多主流数据库供应商支持通过它们驱动的属性配置来添加statement logging的日志功能.这些数据库包括racle, MySQL, Derby, MSSQL等等.一些甚至支持[慢查询](https://baike.baidu.com/item/%E6%85%A2%E6%9F%A5%E8%AF%A2/9200910?fr=aladdin)日志记录功能.对于那些少数不支持的数据库,还有许多可用的其它方式.比如,p6spy,log4jdbc以及jdbcdslog-exp等等.

[mysql推荐配置](https://github.com/brettwooldridge/HikariCP/wiki/MySQL-Configuration)
[Statement和PreparedStatement对象的区别](http://blog.csdn.net/suwu150/article/details/52745055)