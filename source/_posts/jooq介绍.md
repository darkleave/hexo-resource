---
title: jooq介绍
date: 2018-01-15 00:07:42
tags: [jooq]
---

## jooq是什么

jOOQ（Java Object Oriented Querying，即面向Java对象查询）是基于Java访问关系型数据库的工具包，轻量，简单，并且足够灵活，可以轻松的使用Java面向对象语法来实现各种复杂的sql;是一个高效地合并了复杂SQL、类型安全、源码生成、ActiveRecord、存储过程以及高级数据类型的Java API的类库.

<!--more-->

## jooq的特点

1. 类型安全(TypeSafe SQL)
   jooq使用内部的DSL(domain specific language领域专用语言)对sql进行模块化，并且使用java编译器去编译你的sql语法，元数据以及数据类型.

2. 映射代码生成
   jooq可以从你的数据库元数据生成对应的java映射类，生成的实体类按照数据库字段以驼峰命名法重新命名，同时用户可以通过继承实体类的方式来添加自定义属性及方法.

3. ActiveRecords
   我们jooq通过代码生成器生成的ActiveReocrds可以直接对POJO(Plain Old Java Object)映射对象进行CRUD(Create Retrieve Update Delete)操作

4. 多架构(多模式Schema)
   jooq允许你在运行时环境动态配置数据库模式和表并且支持行级别的安全性,即通过不同的jooq Configuration配置得到对应的DSLContext上下文再对数据库进行CRUD操作.

5. 标准化
   jooq可以通过配置数据库方言来支持不同的数据库:mysql,oracle等等,比如通过spring配置spring.jooq.sql-dialect = mysql 来支持mysql数据库

6. 查询生命周期
   jooq通过一些接口开放SQL生成的生命周期，包括日志，事务处理，id生成，sql转换等等。

7. 存储过程
   jooq允许你在模块化sql语句中嵌入存储过程调用.

8. 强大的Fluent API和完善文档,使用方便流畅

**参考链接**
https://www.jianshu.com/p/46164f9ba53c
https://www.jooq.org/



