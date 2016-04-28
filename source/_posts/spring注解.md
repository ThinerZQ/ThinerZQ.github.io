title: spring注解
date: 2016-04-26 15:47:52
tags: 
 spring
 annotation
categories: 
 spring
---

|注解名 |参数| 放置位置| 作用 | 近义词 / 来源| 
| ------------- |:------:|:--------------:|:--------------:|:--------------:| 
| @Required | |setter方法 | 强调这个bean的该属性必须被注入| |
|@Autowired | required| 方法，构造器，属性| 强调该属性ByType自动注入 | @Resource,@Inject |
|@Qualifier | |属性，构造器参数，方法参数 |和@Autowire配合使用，当ByType冲突时候，使用ByName |  |
|@Primary || 方法，构造器，属性 | 当@Autowired ByType冲突的时候，自动选择有该注解的对象| |
|@Resource |name| setter方法，属性|同@Autowired |@Autowired,@Inject |
|@PostConstruct | |方法| 强调该方法, 会在实例化之后自动调用 | |
|@PreDestroy | |方法| 强调该方法, 会在销毁之前自动调用 | |
|@Component | | |更加一般会的组件, 不确定该组件作用的时候，使用这个组件| |
|@Service | bane |类 | 表示服务层| |
|@Repository | |类 | 表示数据处理层| | 
|@ComponentScan |basePackages, includeFilters, excludeFilters | 类| 自动发现组件| |
|@Bean |name, initMethod, destroyMethod | 特定方法 | 生成一个bean定义| | 
|@Description | | 方法| 和@Bean一起配合使用，指定bean的描述 | |
|@Repository |name | 类 |   |    |
|@Controller| name | 类 |  |  |
|@Scope |name, proxyMode | 类 | 表示bean的scope域| |
|@Inject | |方法，构造器，属性 | 强调该属性ByType自动注入 | @Autowire / JSR-330|
|@Named | | | 更加一般会的组件, 不确定该组件作用的时候，使用这个组件| @Component / JSR-330|
|@Configuration |  | 类| 和</beans>标签很像，最好和@Bean搭配使用，可以解决内部依赖问题 |  |
|@Import | | 类 | 和@Configuration搭配使用 | | 
|@PropertySource | | 类 | 注入属性资源 | | 



# 在web应用程序中启动Spring注解配置
```xml
        <web‐app>
        <!‐‐ Configure ContextLoaderListener to use AnnotationConfigWebApplicationContext
        instead of the default XmlWebApplicationContext ‐‐>
            <context‐param>
                <param‐name>contextClass</param‐name>
                <param‐value>
                org.springframework.web.context.support.AnnotationConfigWebApplicationContext
                </param‐value>
            </context‐param>
        <!‐‐ Configuration locations must consist of one or more comma‐ or space‐delimited
        fully‐qualified @Configuration classes. Fully‐qualified packages may also be
        specified for component‐scanning ‐‐>
            <context‐param>
                <param‐name>contextConfigLocation</param‐name>
                <param‐value>com.acme.AppConfig</param‐value>
            </context‐param>
        <!‐‐ Bootstrap the root application context as usual using ContextLoaderListener ‐‐>
        <listener>
            <listener‐class>org.springframework.web.context.ContextLoaderListener</listener‐class>
        </listener>
        <!‐‐ Declare a Spring MVC DispatcherServlet as usual ‐‐>
        <servlet>
        <servlet‐name>dispatcher</servlet‐name>
        <servlet‐class>org.springframework.web.servlet.DispatcherServlet</servlet‐class>
        <!‐‐ Configure DispatcherServlet to use AnnotationConfigWebApplicationContext
        instead of the default XmlWebApplicationContext ‐‐>
            <init‐param>
                <param‐name>contextClass</param‐name>
                <param‐value>
                org.springframework.web.context.support.AnnotationConfigWebApplicationContext
                </param‐value>
            </init‐param>
        <!‐‐ Again, config locations must consist of one or more comma‐ or space‐delimited
        and fully‐qualified @Configuration classes ‐‐>
            <init‐param>
                <param‐name>contextConfigLocation</param‐name>
                <param‐value>com.acme.web.MvcConfig</param‐value>
            </init‐param>
        </servlet>
        <!‐‐ map all requests for /app/* to the dispatcher servlet ‐‐>
        <servlet‐mapping>
        <servlet‐name>dispatcher</servlet‐name>
             <url‐pattern>/app/*</url‐pattern>
        </servlet‐mapping>
        </web‐app>
```
 


