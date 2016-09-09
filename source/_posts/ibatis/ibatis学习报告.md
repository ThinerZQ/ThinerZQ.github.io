title: Ibatis学习
date: 2016-07-20 09:08:06
tags: [ibatis]
categories: [ibatis]
---
# iBatis 简介：
iBatis 是apache 的一个开源项目(2010后改名为Mybatis)，一个半自动化的O/R Mapping 解决方案，iBatis 最大的特点就是小巧，上手很快。
适用于以下场景：
1. 系统的部分或全部数据来自现有数据库，出于安全考虑，只对开发团队提供几条Select SQL（或存储过程）以获取所需数据，具体的表结构不予公开。
2. 开发规范中要求，所有牵涉到业务逻辑部分的数据库操作，必须在数据库层由存储过程实现
3. 系统数据处理量巨大，性能要求极为苛刻，这往往意味着必须通过经过高度优化的SQL语句才能达到系统性能设计指标。

ORM框架都是在JDBC层面上的封装，把如下的jdbc代码分解到不同的步骤去执行，以达到流程化，可定制，方便配置管理的目的。
```java
Class.forName("com.mysql.jdbc.Driver");
Connection conn= DriverManager.getConnection(url,user,password);
PreparedStatement  st = conn.prepareStatement(sql);
st.setInt(0,1);
st.execute();
ResultSet rs =  st.getResultSet();
while(rs.next()){
    String result = rs.getString(colname);
}
```

iBatis主要做了两件事:
- 根据 JDBC 规范建立与数据库的连接
- 通过反射打通 Java 对象与数据库参数交互之间相互转化关系。


iBATIS的一个重要组成部分就是其 SqlMap 配置文件，SqlMap 配置文件的核心是 Statement 语句包括CRUD。 iBATIS 通过解析 SqlMap 配置文件得到所有的 sql 执行语句，同时会形成 ParameterMap、ResultMap 两个对象用于处理sql语句中的参数和sql语句执行后的ResultSet-->Object的构造。

<!-- more -->
# 基本配置
## SqlMapConfig.xml
- properties：将数据连接单独配置在db.properties中，只需要在SqlMapConfig.xml中加载db.properties的属性值，在SqlMapConfig.xml中就不需要对数据库连接参数进行硬编码。数据库连接参数只配置在db.properties中，方便对参数进行统一管理，其它xml可以引用该db.properties

- settings：全局配置参数，用于配置和优化SqlMapClient实例的各选项，如缓存，延迟加载，最大线程数等等。
- typeAlias：为一个通常较长的、全限定类名指定一个较短的别名。SQL Map配置文件预定义了几个别名
  1. JDBC com.ibatis.sqlmap.engine.transaction.jdbc.JdbcTransactionConfig
  2. JTA    com.ibatis.sqlmap.engine.transaction.jta.JtaTransactionConfig
  3. EXTERNAL com.ibatis.sqlmap.engine.transaction.external.ExternalTransactionConfig
  4. SIMPLE   com.ibatis.sqlmap.engine.datasource.SimpleDataSourceFactory
  5. DBCP     com.ibatis.sqlmap.engine.datasource.DbcpDataSourceFactory
  6. JNDI     com.ibatis.sqlmap.engine.datasource.JndiDataSourceFactory
- transactionManager ： 为SQL Map配置事务管理服务。属性type指定所使用的事务管理器类型。这个属性值可以是一个类名，也可以是一个别名。包含在框架的三个事务管理器分别是：JDBC，JTA和EXTERNAL。
  - JDBC：通过常用的Connection commit()和rollback()方法，让JDBC管理事务。
  - JTA：本事务管理器使用一个JTA全局事务，使SQL Map的事务包括在更大的事务范围内，这个更大的事务范围可能包括了其他的数据库和事务资源。这个配置需要一个UserTransaction属性，以便 从JNDI获得一个UserTransaction
  - EXTERNAL：这个配置可以让您自己管理事务。您仍然可以配置一个数据源，但事务不再作为框架生命周期的一部分被提交或回退。这意味着SQL Map外部应用的一部分必须自己管理事务。这个配置也可以用于没有事务管理的数据库（例如只读数据库）

- dataSource：是transactionManager的一部分，为SQL Map数据源设置了一系列参数。必需配置的是的是jdbcdriver url,username,password。
- sqlMap ：用于包括SQL Map映射文件和其他的SQL Map配置文件。每个SqlMapClient对象使用的所有SQL Map映射文件都要在此声明。映射文件作为stream resource从类路径或URL读入。

## sqlMap.xml
sqlMap.xml常用的配置文件的通常主要包括一些具体的sql操作映射,具体说来主要包括如下的元素：
- sqlMap: sqlMap作为根节点，其中的namespace属性用来防止不同的sqlMap文件中同名方法的冲突，在调用的时候使用namespace.id调用就行了。
- typeAlias
- resultMap：结果映射，主要用于指定数据库列名和返回对象属性名的对应关系
- parameterMap：参数映射，主要用于指定传入参数对象的属性名以及类型与数据库列名与类型的对应关系。
- insert，select,update,delete：用于写CRUD 语句的标签，它的parameterMap和resultMap属性指定了程序传递进来的参数如何映射构成sql语句，已经如果将ResultSet结果返回成需要的对象。select中有一个selectKey用于自动生成数据库主键；select语句中可以使用cacheModel来指定缓存。
- procedure:用于定义存储过程
- sql：定义一段公用的sql语句，在insert,select，update,delete中引用，消除重复
### 映射关系分析
ibateis中，parameterClass的类型大都是：String,Integer/Object/Hashmap
resultclass/resultMap的类型大都是：Object/Hashmap

1. 当parameterClass为string,int时，可用#value#表示或直接用传入的值名表示。
2. 当parameterClass/resultMap的类型是对象时，用#属性#表示。程序会调用JAVABEAN的getter方法，进行获取属性值。
3. 当parameterClass/resultMap的类型是hashmap时，那程序会直接通过key来分析取参数。
4. #是把传入的数据当作字符串，\#方式一般用于传入插入/更新的值或查询/删除的where条件。通过PreparedStatement.set(1,value)的形式插入到sql中
5. $传入的数据直接生成在sql里。$方式一般用于传入数据库对象．例如传入表名.

### 动态映射
ibatis还要一个功能点就是动态映射，提高了SQL语句的重用性和灵活性。主要做法就是提供了一系列的动态SQL标签，能够根据传入的参数值的数量，大小，null等做判断生成对应的sql。主要的标签有dynamic,isNull,isNotNull,iterator等。

# 配置解析
一切都要从SqlMapClientBuilder开始说起。首先构造InputStream或者Reader读取配置文件，然后通过SqlMapClientBuilder的静态方法实例化SqlMapConfigParser，其中使用org.w3c.dom来解析文档，当读到xx标签的时候，就实例化xx标签对应的对象）。等到文件解析完,数据源，事物等配置好，所有的CRUD语句加载完返回SqlMapClient实例，SqlMapClient提供了程序和ibatis的交互接口


# 重要类的说明

- SqlMapClient：提供给编程者的接口，使用这个接口可以执行SqlMap文件中定义的半形式化的sql语句
- SqlMapClientImpl：SqlMapClientImpl类使用了ThreadLocal类型的成员变量来存储关联的SqlMapSessionImpl对象。这就是限制了每个线程都只能操作自己记录的SqlMapSessionImpl数据，从而保证了该属性的线程安全。
- SqlMapSessionImpl：主要是为了持有本次session的SessionScope对象，这个勒种的绝大部分数据操作都转交给了SqlMapExecutorDelegate去完成。
- SqlMapExecutorDelegate：一个工具类，所有的SqlMapClient提供给外部的操作都是通过这个类来完成的，主要为各个操作开启，关闭事物的。
- MappedStatement：为各个CRUD方法的构造RowHandlerCallback对象，验证参数类型，取出参数。
- SqlMapExecutor：实例化PreparedStatement, 取得TypeHandler，获取ParameterMap为PreparedStatement设置参数，并执行最终的sql语句得到ResultSet,并调用 MappedStatement中的RowHandlerCallback处理返回结果，RowHandlerCallback获取到当前的ResultMap判断是哪一种DataExchange(object,hashmap,list)并使用对应的DataExchange做类型映射，最后通过ResultObjectFactoryUtil通过反射实例化需要返回的对象。


# 总结
iBatis真的是一个简单易学的持久层框架，将Sql语言与java程序分离，便于维护和开发，大大简化了jdbc操作。与Hibernate相比，虽然没有Hibernate的功能强大，sql语言的效率更高，更灵活。Hibernate提供了从object<-->jdbc完全的封装，用多了sql语句会生疏，iBatis更加的简单易懂，配置灵活。
