title: Spring基础学习
date: 2016-07-09 09:08:06
tags: [spring]
categories: [spring]
---

通常说的Spring其实指的是Spring Framework，它是Spring下的一个子项目，Spring围绕Spring Framework这个核心项目开发了大量其他项目，比如Spring Security，Spring Data，Spring Boot等. Spring Framework包括他的核心解决方案IoC容器、 AOP。

<!-- more -->
# 控制反转

所谓控制反转，是指通过使用IoC容器对象依赖关系的管理被反转了，也就是说，对象之间的依赖关系由IoC容器进行管理，并且由Ioc容器通过依赖注入（DI，Dependency Injection）的方式来完成对象的注入（以前一个对象需要new了才能使用，现在你只要声明就行了，spring会帮你new好了，在你需要的时候把这个对象的引用传递给你）。

整个依赖入住的过程中，有一个问题如何解决循环依赖的对象的注入：构造器循环依赖是无法解决的，只有单例模式下的setter注入可以解决（通过将new完了，还未调用其setter方法的bean放入到Map中，从而使其他Bean能引用到该Bean）

# 切面编程

面向切面编程（AOP）通过提供另外一种思考程序结构的途经来弥补面向对象编程（OOP）的不足。在OOP中模块化的关键单元是类（classes），而在AOP中模块化的单元则是切面。切面能对关注点进行模块化，例如横切多个类型和对象的事务管理。Aop中有一些核心概念：

- 横切性关注点：对哪些方法拦截，拦截后怎么处理，这些关注就称之为横切性关注点.
Aspect（切面）：指横切性关注点的抽象即为切面，它与类相似，只是两者的关注点不一样，类是对物体特征的抽象，而切面是横切性关注点的抽象。
- Joinpoint（连接点）：所谓连接点是指那些被拦截到的点。在Spring中，这些点指的是方法，因为Spring只支持方法类型的连接点，实际上joinpoint还可以是field或类构造器。
- Pointcut(切入点)：所谓切入点是指我们要对那些joinpoin进行拦截的定义。
- Advice（通知）：所谓通知是指拦截到joinpoint之后所要做的事情就是通知。通知分为前置通知，后置通知，异常通知，最终通知，环绕通知。
  - 前置通知（Before advice）：在某连接点之前执行的通知，但这个通知不能阻止连接点之前的执行流程（除非它抛出一个异常）。
  - 后置通知（After returning advice）：在某连接点正常完成后执行的通知：例如，一个方法没有抛出任何异常，正常返回。
  - 异常通知（After throwing advice）：在方法抛出异常退出时执行的通知。
  - 最终通知（After (finally) advice）：当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。
  - 环绕通知（Around Advice）：包围一个连接点的通知，如方法调用。这是最强大的一种通知类型。环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它自己的返回值或抛出异常来结束执行。
- Target（目标对象）：代理的目标对象
- Weave(织入)：指将aspects应用到target对象并导致proxy对象创建的过程称为织入。

由于开启了切面编程功能，所以当获取一个被切面类监控管理的bean对象时，它实际上获取的是此对象的一个代理对象，而在spring中对代理对象的处理有如下原则：（1）如果要代理的对象实现了接口，则会按照Proxy的方式来产生代理对象，这即是说产生的代理对象只能是接口类型。如果要代理的对象未实现接口，则按cglib方式来产生代理对象。
要想spring的切面技术起作用，被管理的bean对象只能是通过spring容器获取的对象。


# 事物管理

在 Spring 中，事务是通过 TransactionDefinition 接口来定义的。接口只提供了获取事物隔离级别，传播行为，超时时间的方法

## 事物的隔离级别
隔离级别是指若干个并发的事务之间的隔离程度。TransactionDefinition 接口中定义了五个表示隔离级别的常量：

- TransactionDefinition.ISOLATION_DEFAULT：这是默认值，表示使用底层数据库的默认隔离级别。对大部分数据库而言，通常这值就是TransactionDefinition.ISOLATION_READ_COMMITTED。
- TransactionDefinition.ISOLATION_READ_UNCOMMITTED：该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。该级别不能防止脏读和不可重复读，因此很少使用该隔离级别。
- TransactionDefinition.ISOLATION_READ_COMMITTED：该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，这也是大多数情况下的推荐值。
- TransactionDefinition.ISOLATION_REPEATABLE_READ：该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询，并且每次返回的记录都相同。即使在多次查询之间有新增的数据满足该查询，这些新增的记录也会被忽略。该级别可以防止脏读和不可重复读。
- TransactionDefinition.ISOLATION_SERIALIZABLE：所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

## 事务传播行为
所谓事务的传播行为是指，如果在开始当前事务之前，一个事务上下文已经存在，此时有若干选项可以指定一个事务性方法的执行行为。在TransactionDefinition定义中包括了如下几个表示传播行为的常量：
- TransactionDefinition.PROPAGATION_REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
- TransactionDefinition.PROPAGATION_REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。
- TransactionDefinition.PROPAGATION_MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。

## 事务超时
所谓事务超时，就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。在 TransactionDefinition 中以 int 的值来表示超时时间，其单位是秒。

通常情况下，如果在事务中抛出了未检查异常（继承自 RuntimeException 的异常），则默认将回滚事务。如果没有抛出任何异常，或者抛出了已检查异常，则仍然提交事务

基于 @Transactional 的声明式事务管理，可以作用于接口、接口方法、类以及类方法上。当作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，也可以在方法级别使用该标注来覆盖类级别的定义。， @Transactional 注解应该只被应用到 public 方法上，如果在protected、private 或者默认可见性的方法上使用 @Transactional 注解，将被忽略，也不会抛出任何异常。


# SpringMVC

DispatcherServlet是前置控制器，配置在web.xml文件中的。拦截匹配的请求，Servlet拦截匹配规则要自已定义，把拦截下来的请求，依据相应的规则分发到目标Controller来处理，是配置spring MVC的第一步。

Controller：具体处理请求的控制器。日常功能都在Controller 中处理

handlerMapping：映射处理器。负责映射中央处理器转发给 Controller时的映射策略，即DispatcherServlet按什么规则去寻找需要访问的


Controller。需要在spring-mvc的配置文件中配置。主要有三种类型，分别为为按Controller控制器的名称、类型和简单url隐射。默认为按Controller的名称（可省略）

ModelAndView：服务层返回的数据和视图层的封装类,用来封装 Controller 需要访问的视图(view)和传输的数据(model)，一般在控制器Controller  中自己实现处理。

ViewResolver & View：视图解析器.需要在spring-mvc的配置文件中配置，一般分为前缀与后缀。前缀：webroot到某一指定的文件夹的路径；后缀：视图名称的后缀。然后和 ModelAndView 中的view结合使用，确认具体的访问视图。

Interceptors ：拦截器,负责拦截我们定义的请求然后做处理工作。

运行原理：
1. 客户端请求提交到DispatcherServlet；
2. 由DispatcherServlet控制器查询一个或多个HandlerMapping，找到处理请求的Controller；
3. DispatcherServlet将请求提交到Controller；
4. Controller调用业务逻辑处理后，返回ModelAndView；
5. DispatcherServlet查询一个或多个ViewResoler视图解析器，找到ModelAndView指定的视图；
6. 视图负责将结果显示到客户端。
