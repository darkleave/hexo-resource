---
title: Pom.xml详解
date: 2018-01-11 00:07:42
tags: [Maven]
---

Maven pom.xml详细配置说明

<!--more-->

## 1.概述

pom中节点如下分布

    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
    
        <!-- 基本配置 -->
        <groupId>...</groupId>
        <artifactId>...</artifactId>
        <version>...</version>
        <packaging>...</packaging>
    
    
        <!-- 依赖配置 -->
        <dependencies>...</dependencies>
        <parent>...</parent>
        <dependencyManagement>...</dependencyManagement>
        <modules>...</modules>
        <properties>...</properties>
    
        <!-- 构建配置 -->
        <build>...</build>
        <reporting>...</reporting>
    
        <!-- 项目信息 -->
        <name>...</name>
        <description>...</description>
        <url>...</url>
        <inceptionYear>...</inceptionYear>
        <licenses>...</licenses>
        <organization>...</organization>
        <developers>...</developers>
        <contributors>...</contributors>
    
        <!-- 环境设置 -->
        <issueManagement>...</issueManagement>
        <ciManagement>...</ciManagement>
        <mailingLists>...</mailingLists>
        <scm>...</scm>
        <prerequisites>...</prerequisites>
        <repositories>...</repositories>
        <pluginRepositories>...</pluginRepositories>
        <distributionManagement>...</distributionManagement>
        <profiles>...</profiles>
    </project>

## 2.基本配置

- modelVersion：pom模型版本，maven2和3只能为4.0.0
- groupId：组ID，maven用于定位
- artifactId：在组中的唯一ID用于定位
- version：项目版本
- packaging：项目打包方式，有以下值：pom, jar, maven-plugin, ejb, war, ear, rar, par

## 3.依赖配置

parent

用于确定父项目的坐标。

    <parent>
        <groupId>com.learnPro</groupId>
        <artifactId>SIP-parent</artifactId>
        <relativePath></relativePath>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    
- groupId：父项目的构件标识符
- artifactId：父项目的唯一标识符
- relativePath：Maven首先在当前项目的找父项目的pom，然后在文件系统的这个位置（relativePath），然后在本地仓库，再在远程仓库找。
- version：父项目的版本

modules

有些maven项目会做成多模块的，这个标签用于指定当前项目所包含的所有模块。之后对这个项目进行的maven操作，会让所有子模块也进行相同操作。

    <modules>
       <module>com-a</module>
       <module>com-b</module>
       <module>com-c</module>
    </modules>
    
properties

用于定义pom常量

    <properties>
        <java.version>1.7</java.version>
    </properties>
    
上面这个常量可以在pom文件的任意地方通过${java.version}来引用

dependencies

项目相关依赖配置，如果在父项目写的依赖，会被子项目引用，一般父项目会将子项目公用的依赖引入（将在之后详细讲解）

    <dependencies>
        <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.12</version>
        </dependency>
    </dependencies>
    
这边依赖和中央仓库中的一致，就可以引入对应的jar

dependencyManagement

配置写法同dependencies

    <dependencyManagement>
        <dependencies>
        .....
        </dependencies>
    </dependencyManagement>

在父模块中定义后，子模块不会直接使用对应依赖，但是在使用相同依赖的时候可以不加版本号：

    父项目：
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.12</version>
                <scope>test</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    
    子项目：
    
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
    </dependency>
    
这样的好处是，父项目统一了版本，而且子项目可以在需要的时候才引用对应的依赖    

## 4.构建配置

build

用于配置项目构建相关信息

    <build>    
        <!--该元素设置了项目源码目录，当构建项目的时候，构建系统会编译目录里的源码。该路径是相对于pom.xml的相对路径。-->    
        <sourceDirectory/>    
        <!--该元素设置了项目脚本源码目录，该目录和源码目录不同：绝大多数情况下，该目录下的内容 会被拷贝到输出目录(因为脚本是被解释的，而不是被编译的)。-->    
      <scriptSourceDirectory/>    
      <!--该元素设置了项目单元测试使用的源码目录，当测试项目的时候，构建系统会编译目录里的源码。该路径是相对于pom.xml的相对路径。-->    
      <testSourceDirectory/>    
      <!--被编译过的应用程序class文件存放的目录。-->    
      <outputDirectory/>    
      <!--被编译过的测试class文件存放的目录。-->    
      <testOutputDirectory/>    
      <!--使用来自该项目的一系列构建扩展-->    
      <extensions>    
       <!--描述使用到的构建扩展。-->    
       <extension>    
        <!--构建扩展的groupId-->    
        <groupId/>    
        <!--构建扩展的artifactId-->    
        <artifactId/>    
        <!--构建扩展的版本-->    
        <version/>    
       </extension>    
      </extensions>    
      <!--当项目没有规定目标（Maven2 叫做阶段）时的默认值-->    
      <defaultGoal/>    
      <!--这个元素描述了项目相关的所有资源路径列表，例如和项目相关的属性文件，这些资源被包含在最终的打包文件里。-->    
      <resources>    
       <!--这个元素描述了项目相关或测试相关的所有资源路径-->    
       <resource>    
        <!-- 描述了资源的目标路径。该路径相对target/classes目录（例如${project.build.outputDirectory}）。举个例 子，如果你想资源在特定的包里(org.apache.maven.messages)，你就必须该元素设置为org/apache/maven /messages。然而，如果你只是想把资源放到源码目录结构里，就不需要该配置。-->    
        <targetPath/>    
        <!--是否使用参数值代替参数名。参数值取自properties元素或者文件里配置的属性，文件在filters元素里列出。-->    
        <filtering/>    
        <!--描述存放资源的目录，该路径相对POM路径-->    
        <directory/>    
        <!--包含的模式列表，例如**/*.xml.-->    
        <includes/>    
        <!--排除的模式列表，例如**/*.xml-->    
        <excludes/>    
       </resource>    
      </resources>    
      <!--这个元素描述了单元测试相关的所有资源路径，例如和单元测试相关的属性文件。-->    
      <testResources>    
       <!--这个元素描述了测试相关的所有资源路径，参见build/resources/resource元素的说明-->    
       <testResource>    
        <targetPath/><filtering/><directory/><includes/><excludes/>    
       </testResource>    
      </testResources>    
      <!--构建产生的所有文件存放的目录-->    
      <directory/>    
      <!--产生的构件的文件名，默认值是${artifactId}-${version}。-->    
      <finalName/>    
      <!--当filtering开关打开时，使用到的过滤器属性文件列表-->    
      <filters/>    
      <!--子项目可以引用的默认插件信息。该插件配置项直到被引用时才会被解析或绑定到生命周期。给定插件的任何本地配置都会覆盖这里的配置-->    
      <pluginManagement>    
       <!--使用的插件列表 。-->    
       <plugins>    
        <!--plugin元素包含描述插件所需要的信息。-->    
        <plugin>    
         <!--插件在仓库里的group ID-->    
         <groupId/>    
         <!--插件在仓库里的artifact ID-->    
         <artifactId/>    
         <!--被使用的插件的版本（或版本范围）-->    
         <version/>    
         <!--是否从该插件下载Maven扩展（例如打包和类型处理器），由于性能原因，只有在真需要下载时，该元素才被设置成enabled。-->    
         <extensions/>    
         <!--在构建生命周期中执行一组目标的配置。每个目标可能有不同的配置。-->    
         <executions>    
          <!--execution元素包含了插件执行需要的信息-->    
          <execution>    
           <!--执行目标的标识符，用于标识构建过程中的目标，或者匹配继承过程中需要合并的执行目标-->    
           <id/>    
           <!--绑定了目标的构建生命周期阶段，如果省略，目标会被绑定到源数据里配置的默认阶段-->    
           <phase/>    
           <!--配置的执行目标-->    
           <goals/>    
           <!--配置是否被传播到子POM-->    
           <inherited/>    
           <!--作为DOM对象的配置-->    
           <configuration/>    
          </execution>    
         </executions>    
         <!--项目引入插件所需要的额外依赖-->    
         <dependencies>    
          <!--参见dependencies/dependency元素-->    
          <dependency>    
           ......    
          </dependency>    
         </dependencies>         
         <!--任何配置是否被传播到子项目-->    
         <inherited/>    
         <!--作为DOM对象的配置-->    
         <configuration/>    
        </plugin>    
       </plugins>    
      </pluginManagement>    
      <!--使用的插件列表-->    
      <plugins>    
       <!--参见build/pluginManagement/plugins/plugin元素-->    
       <plugin>    
        <groupId/><artifactId/><version/><extensions/>    
        <executions>    
         <execution>    
          <id/><phase/><goals/><inherited/><configuration/>    
         </execution>    
        </executions>    
        <dependencies>    
         <!--参见dependencies/dependency元素-->    
         <dependency>    
          ......    
         </dependency>    
        </dependencies>    
        <goals/><inherited/><configuration/>    
       </plugin>    
      </plugins>    
     </build>
     
reporting

该元素描述使用报表插件产生报表的规范。当用户执行“mvn site”，这些报表就会运行。 在页面导航栏能看到所有报表的链接。

    <reporting>    
      <!--true，则，网站不包括默认的报表。这包括“项目信息”菜单中的报表。-->    
      <excludeDefaults/>    
      <!--所有产生的报表存放到哪里。默认值是${project.build.directory}/site。-->    
      <outputDirectory/>    
      <!--使用的报表插件和他们的配置。-->    
      <plugins>    
       <!--plugin元素包含描述报表插件需要的信息-->    
       <plugin>    
        <!--报表插件在仓库里的group ID-->    
        <groupId/>    
        <!--报表插件在仓库里的artifact ID-->    
        <artifactId/>    
        <!--被使用的报表插件的版本（或版本范围）-->    
        <version/>    
        <!--任何配置是否被传播到子项目-->    
        <inherited/>    
        <!--报表插件的配置-->    
        <configuration/>    
        <!--一组报表的多重规范，每个规范可能有不同的配置。一个规范（报表集）对应一个执行目标 。例如，有1，2，3，4，5，6，7，8，9个报表。1，2，5构成A报表集，对应一个执行目标。2，5，8构成B报表集，对应另一个执行目标-->    
        <reportSets>    
         <!--表示报表的一个集合，以及产生该集合的配置-->    
         <reportSet>    
          <!--报表集合的唯一标识符，POM继承时用到-->    
          <id/>    
          <!--产生报表集合时，被使用的报表的配置-->    
          <configuration/>    
          <!--配置是否被继承到子POMs-->    
          <inherited/>    
          <!--这个集合里使用到哪些报表-->    
          <reports/>    
         </reportSet>    
        </reportSets>    
       </plugin>    
      </plugins>    
     </reporting>
     
## 5.项目信息

- name：给用户提供更为友好的项目名
- description：项目描述，maven文档中保存
- url：主页的URL，maven文档中保存
- inceptionYear：项目创建年份，4位数字。当产生版权信息时需要使用这个值
- licenses：该元素描述了项目所有License列表。 应该只列出该项目的license列表，不要列出依赖项目的 license列表。如果列出多个license，用户可以选择它们中的一个而不是接受所有license。（如下）


        <license>  
    
            <!--license用于法律上的名称-->    
            <name>...</name>     
            <!--官方的license正文页面的URL-->    
            <url>....</url>
            <!--项目分发的主要方式：repo，可以从Maven库下载 manual， 用户必须手动下载和安装依赖-->    
            <distribution>repo</distribution>     
            <!--关于license的补充信息-->    
            <comments>....</comments>     
        </license> 
    
- organization：1.name 组织名 2.url 组织主页url
- developers：项目开发人员列表（如下）
- contributors：项目其他贡献者列表，同developers    


        <developers>  
    
        <!--某个开发者信息-->
        <developer>  
            <!--开发者的唯一标识符-->
            <id>....</id>  
            <!--开发者的全名-->
            <name>...</name>  
            <!--开发者的email-->
            <email>...</email>  
            <!--开发者的主页-->
            <url>...<url/>
            <!--开发者在项目中的角色-->
            <roles>  
                <role>Java Dev</role>  
                <role>Web UI</role>  
            </roles> 
            <!--开发者所属组织--> 
            <organization>sun</organization>  
            <!--开发者所属组织的URL-->
            <organizationUrl>...</organizationUrl>  
            <!--开发者属性，如即时消息如何处理等-->
            <properties>
                <!-- 和主标签中的properties一样，可以随意定义子标签 -->
            </properties> 
            <!--开发者所在时区， -11到12范围内的整数。--> 
            <timezone>-5</timezone>  
        </developer>  
        </developers>  
        

## 6.环境设置

issueManagement

目的问题管理系统(Bugzilla, Jira, Scarab)的名称和URL

    <issueManagement>
        <system>Bugzilla</system>
        <url>http://127.0.0.1/bugzilla/</url>
    </issueManagement>
    
- system：系统类型
- url：路径    

ciManagement

项目的持续集成信息

    <ciManagement>
        <system>continuum</system>
        <url>http://127.0.0.1:8080/continuum</url>
        <notifiers>
          <notifier>
            <type>mail</type>
            <sendOnError>true</sendOnError>
            <sendOnFailure>true</sendOnFailure>
            <sendOnSuccess>false</sendOnSuccess>
            <sendOnWarning>false</sendOnWarning>
            <address>continuum@127.0.0.1</address>
            <configuration></configuration>
          </notifier>
        </notifiers>
      </ciManagement>
      
- system：持续集成系统的名字
- url：持续集成系统的URL
- notifiers：构建完成时，需要通知的开发者/用户的配置项。包括被通知者信息和通知条件（错误，失败，成功，警告） 
    - type：通知方式
    - sendOnError：错误时是否通知
    - sendOnFailure：失败时是否通知
    - sendOnSuccess：成功时是否通知
    - sendOnWarning：警告时是否通知
    - address：通知发送到的地址
    - configuration：扩展项      
    

mailingLists

项目相关邮件列表信息

    <mailingLists>
        <mailingList>
          <name>User List</name>
          <subscribe>user-subscribe@127.0.0.1</subscribe>
          <unsubscribe>user-unsubscribe@127.0.0.1</unsubscribe>
          <post>user@127.0.0.1</post>
          <archive>http://127.0.0.1/user/</archive>
          <otherArchives>
            <otherArchive>http://base.google.com/base/1/127.0.0.1</otherArchive>
          </otherArchives>
        </mailingList>
        .....
      </mailingLists>
      
- subscribe, unsubscribe: 订阅邮件（取消订阅）的地址或链接，如果是邮件地址，创建文档时，mailto: 链接会被自动创建
- archive：浏览邮件信息的URL
- post：接收邮件的地址      

scm

允许你配置你的代码库，供Maven web站点和其它插件使用


    <scm>
        <connection>scm:svn:http://127.0.0.1/svn/my-project</connection>
        <developerConnection>scm:svn:https://127.0.0.1/svn/my-project</developerConnection>
        <tag>HEAD</tag>
        <url>http://127.0.0.1/websvn/my-project</url>
    </scm>
    
- connection, developerConnection：这两个表示我们如何连接到maven的版本库。connection只提供读，developerConnection将提供写的请求 
    - 写法如：scm:[provider]:[provider_specific]
    - 如果连接到CVS仓库，可以配置如下：scm:cvs:pserver:127.0.0.1:/cvs/root:my-project
- tag：项目标签，默认HEAD
- url：共有仓库路径    

prerequisites

项目构建的前提

    <prerequisites>
        <maven>2.0.6</maven>
    </prerequisites>
    
repositories,pluginRepositories

依赖和扩展的远程仓库列表，同上篇文章，setting.xml配置中介绍的。

    <repositories>
        <repository>
          <releases>
            <enabled>false</enabled>
            <updatePolicy>always</updatePolicy>
            <checksumPolicy>warn</checksumPolicy>
          </releases>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>never</updatePolicy>
            <checksumPolicy>fail</checksumPolicy>
          </snapshots>
          <id>codehausSnapshots</id>
          <name>Codehaus Snapshots</name>
          <url>http://snapshots.maven.codehaus.org/maven2</url>
          <layout>default</layout>
        </repository>
      </repositories>
      <pluginRepositories>
        ...
      </pluginRepositories>
      
- releases, snapshots:这是各种构件的策略，release或者snapshot。这两个集合，POM就可以根据独立仓库任意类型的依赖改变策略。如：一个人可能只激活下载snapshot用来开发。
- enable：true或者false，决定仓库是否对于各自的类型激活(release 或者 snapshot)。
- updatePolicy: 这个元素决定更新频率。maven将比较本地pom的时间戳（存储在仓库的maven数据文件中）和远程的. 有以下选择: always, daily (默认), interval:X (x是代表分钟的整型) ， never.
- checksumPolicy：当Maven向仓库部署文件的时候，它也部署了相应的校验和文件。可选的为：ignore，fail，warn，或者不正确的校验和。
- layout：在上面描述仓库的时候，提到他们有统一的布局。Maven 2有它仓库默认布局。然而，Maven 1.x有不同布局。使用这个元素来表明它是default还是legacy。      

distributionManagement

它管理的分布在整个构建过程生成的工件和支持文件

    <distributionManagement>
        ...
        <downloadUrl>http://mojo.codehaus.org/my-project</downloadUrl>
        <status>deployed</status>
    </distributionManagement>
    
- downloadUrl: 其他pom可以通过此url的仓库抓取组件
- status：给出该构件在远程仓库的状态 
    - none: 默认
    - converted: 将被早期Maven 2 POM转换过来
    - partner: 这个项目会从合作者仓库同步过来
    - deployed: 从Maven 2或3实例部署
    - verified: 被核实时正确的和最终的    
    
Repository

指定Maven pom从远程下载控件到当前项目的位置和方式，如果snapshotRepository没有被定义则使用repository相关的配置


     <distributionManagement>
        <repository>
          <uniqueVersion>false</uniqueVersion>
          <id>corp1</id>
          <name>Corporate Repository</name>
          <url>scp://repo/maven2</url>
          <layout>default</layout>
        </repository>
        <snapshotRepository>
          <uniqueVersion>true</uniqueVersion>
          <id>propSnap</id>
          <name>Propellors Snapshots</name>
          <url>sftp://propellers.net/maven</url>
          <layout>legacy</layout>
        </snapshotRepository>
        ...
      </distributionManagement>

- id, name：仓库的唯一标识
- uniqueVersion：true或false，指明控件部署的时候是否获取独立的版本号。
- url：repository元素的核心。指定位置和部署协议发布控件到仓库。
- layout：布局，default或legacy

Site Distribution

多分布存储库,distributionManagement负责定义如何部署项目的网站和文档。

     <distributionManagement>
        ...
        <site>
          <id>mojo.website</id>
          <name>Mojo Website</name>
          <url>scp://beaver.codehaus.org/home/projects/mojo/public_html/</url>
        </site>
        ...
      </distributionManagement>

- id, name, url: 这些元素与distributionManagement repository中的相同

Relocation

重新部署-项目不是静态的，是活的。他们需要被搬到更合适的地方。如：当你的下个成功的开源项目移到Apache下，重命名为org.apache:my-project:1.0 对你项目更有好处。

    <distributionManagement>
        ...
        <relocation>
          <groupId>org.apache</groupId>
          <artifactId>my-project</artifactId>
          <version>1.0</version>
          <message>We have moved the Project under Apache</message>
        </relocation>
        ...
     </distributionManagement>
     
     
**原文链接: http://blog.csdn.net/oDeviloo/article/details/52050277**     
**参考官方文档： http://maven.apache.org/pom.html**   
