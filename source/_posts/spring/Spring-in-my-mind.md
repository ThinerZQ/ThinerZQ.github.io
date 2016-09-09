title: Spring in my mind
date: 2016-04-24 18:41:17
tags: [spring]
categories: [spring]
---

Spring最为最强大的java 企业级开发框架，大约3年前有接触过，但是学的不深，读研的日子没有多少时间去学习这个框架，现在找实习就要用并且之前的学的是Spring 3的内容，现在都Spring 4, Servlt 3.0了变化好大，有必要重新系统的学习一边，从看官方文档开始。[spring framework](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/)
# 模块
## Spring的核心容器

Spring的核心模块包括：<code>spring‐core</code> , <code>spring‐beans</code> , <code>spring‐context</code> , <code>spring‐context‐support</code> , and
<code>spring‐expression</code> modules.

<code>spring‐core</code>，<code>spring‐beans</code> ：Beans和Core提供了最基本的DI功能  

<code>spring‐context</code>: Context提供了在应用程序中访问对象的方式，并且在Beans和Core的基础上提供了，国际化，事件传播，资源加载的功能,同时也支持EJB,JMX等JavaEE特征，其中最重要的<code>ApplicationContext</code>>接口  

<code>spring‐context‐support</code>：提供了整合第三方类库到Spring Context中，例如：(EhCache, Guava, JCache), 邮件 (JavaMail), 调度 (CommonJ, Quartz)，和模板引擎(FreeMarker, JasperReports, Velocity).  

<code>spring‐expression</code>：spring的表达式语言，扩展自 JSP规范里面的EL表达式，提供了在运行时查询或者访问对象属性值的功能，同时还支持投影，选择和聚合功能

<!-- more -->

## AOP模块

<code>spring-aop</code>>: 提供了面向切面编程的功能，实现了功能的解耦  
<code>spring‐aspects</code>: 提供了和AspectJ整合的功能

## Messaging模块
<code>spring-messaging</code> 模块为集成messaging api和消息协议提供支持。

## Data Access/Integration模块
<code>spring‐jdbc</code>: 提供了JDBC的抽象  
<code>spring‐tx</code>: 提供了声明式的实物管理  
<code>spring‐orm</code>: 提供了和其他ORM框架的整合能力，例如：including JPA, JDO, and Hibernate  
<code>pring‐oxm</code>: provides an abstraction layer that supports Object/XML mapping implementations such as JAXB, Castor, 没了解过
XMLBeans, JiBX and XStream. （不是很了解）  
<code>spring‐jms</code>: 提供了产生和消费的功能，从spring 4.1以后，这个模块逐渐和<code>spring-messaging</code>模块进行整合了。

## Web 部分

<code>spring‐web</code>>: 提供了基础的web相关的整合，例如文件上传，使用面向web应用程序的监听器去初始化IOC容器等。   
<code>spring‐webmvc</code>>： 包含了Web MVC模式和REST web service的实现  
<code>spring‐webmvc-portlet</code>>：提供了在Portlet环境下的Web mvc模式的实现  

## Test 部分
<code>spring-test</code>: 支持Junit和TestNG的单元测试和整合测试，他提供了加载Spring应用程序上下文和cache的功能，同时还提供了Mock 对象的功能。  


## Spring 框架Artifacts
| Group id | Artifact id | Description |
| ------------- |:-------------:|:--------------:|
|org.springframework | spring-aop | Proxybased AOP support |
|org.springframework |spring-aspects|AspectJ based aspects |
|org.springframework |spring-beans|Beans support, including Groovy|
|org.springframework |spring-context|Application context runtime, including scheduling and remoting abstractions |
|org.springframework |spring-context-support|Support classes for integrating common thirdparty libraries into a Spring application context|
|org.springframework| spring-core |Core utilities, used by many other Spring modules |
|org.springframework |spring-expression|Spring Expression Language (SpEL)|
|org.springframework |spring-instrument|Instrumentation agent for JVM bootstrapping |
|org.springframework |spring-instrument-tomcat|Instrumentation agent for Tomcat|
|org.springframework |spring-jdbc|JDBC support package, including DataSource setup and JDBC access support |
|org.springframework |spring-jms|JMS support package, including helper classes to send and receive JMS messages |
|org.springframework |spring-messaging|Support for messaging architectures and protocols |
|org.springframework |spring-orm|Object/Relational Mapping, including JPA and Hibernate support|
|org.springframework| spring-oxm| Object/XML Mapping|
|org.springframework |spring-test|Support for unit testing and integration testing Spring components|
|org.springframework |spring-tx|Transaction infrastructure, including DAO support and JCA integration|
|org.springframework| spring-web|Web support packages, including client and web remoting|
|org.springframework |spring-webmvc|REST Web Services and modelviewcontroller
implementation for web applications|
|org.springframework |spring-webmvc-portlet|MVC implementation to be used in a Portlet environment|
|org.springframework |spring-websocket|WebSocket and SockJS implementations, including STOMP support|

## 使用Maven管理依赖
### Spring远程仓库
for release
```xml
<repositories>
<repository>
<id>io.spring.repo.maven.release</id>
<url>http://repo.spring.io/release/</url>
<snapshots><enabled>false</enabled></snapshots>
</repository>
</repositories>
```

for milestones
```xml
<repositories>
<repository>
<id>io.spring.repo.maven.milestone</id>
<url>http://repo.spring.io/milestone/</url>
<snapshots><enabled>false</enabled></snapshots>
</repository>
</repositories>
```

for snapshot  
```xml
<repositories>
<repository>
<id>io.spring.repo.maven.snapshot</id>
<url>http://repo.spring.io/snapshot/</url>
<snapshots><enabled>true</enabled></snapshots>
</repository>
</repositories>
```
### BOM 依赖
使用这个依赖可以不用指定spring模块的版本号  
```xml
<dependency>
<groupId>org.springframework</groupId>
<artifactId>spring‐framework‐bom</artifactId>
<version>4.2.5.RELEASE</version>
<type>pom</type>
<scope>import</scope>
</dependency>
```
## 如何选择Spring中的日志框架
首先Spring里面默认使用了 <code>commons-logging</code>作为内部的日志框架。
### 1、commons-logging  + log4j 搭配
依赖配置如下
```xml
<dependencies>
<dependency>
<groupId>org.springframework</groupId>
<artifactId>spring‐core</artifactId>
<version>4.2.5.RELEASE</version>
</dependency>
<dependency>
<groupId>log4j</groupId>
<artifactId>log4j</artifactId>
<version>1.2.14</version>
</dependency>
</dependencies>
```
commons-logging 会自动发现类路径下的log4j类库和log4j.propertites文件，使用log4j来记录日志

### 2、不使用commons-logging ，改为 SLF4J +log4j记录日志
依赖配置如下
```xml
<dependencies>
<dependency>
<groupId>org.springframework</groupId>
<artifactId>spring‐core</artifactId>
<version>4.2.5.RELEASE</version>
<exclusions>
<exclusion>
<groupId>commons‐logging</groupId>
<artifactId>commons‐logging</artifactId>
</exclusion>
</exclusions>
</dependency>
<dependency>
<groupId>org.slf4j</groupId>
<artifactId>jcl‐over‐slf4j</artifactId>
<version>1.5.8</version>
</dependency>
<dependency>
<groupId>org.slf4j</groupId>
<artifactId>slf4j‐api</artifactId>
<version>1.5.8</version>
</dependency>
<dependency>
<groupId>org.slf4j</groupId>
<artifactId>slf4j‐log4j12</artifactId>
<version>1.5.8</version>
</dependency>
<dependency>
<groupId>log4j</groupId>
<artifactId>log4j</artifactId>
<version>1.2.14</version>
</dependency>
</dependencies>
```
首先将在<code>Spring-core</code>包里面使用的commons-logging 框架去掉，然后加上slf4j和 log4j的包，spring会自动调用slf4j的接口，由slf4j区发现真正的日志实现框架log4j。
