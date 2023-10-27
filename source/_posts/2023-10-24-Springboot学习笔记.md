---
title: Springboot学习笔记
date: 2023-10-24 09:09:19
tags: Springboot
categories: [后端技术,Spring]
---

***

# 一、Springboot基础知识记录
### 1、Springboot IOC和DI注入方式
1、按照类型注入（直接在类上使用`@Autowired`）  
（1）当只有一个名字相同的类存在：  
``` java
// Service层
@Service
public class DemoServiceImpl implements DemoService {
    .........
}

// Controller层
@Controller
public class DemoController {

    @Autowired
    private DemoService demoService;
}
```
（2）当存在多个相同的类名（不在同一个包下）：  
``` java
// Service层
@Service("a")
public class DemoServiceImpl implements DemoService {
    .........
}

@Service("b")
public class DemoServiceImpl implements DemoService {
    .........
}

// Controller层
@Controller
public class DemoController {

    @Autowired
    @Qualifier(value="a") // 使用a声明的service注入
    private DemoService demoService;
}
```

2、按照名称注入  
**说明：**直接在类上使用`@Resource`（`@Resource`相当于`@Autowired + @Qualifier`）
``` java
// Service层
@Service("a")
public class DemoServiceImpl implements DemoService {
    // .........
}

@Service("b")
public class DemoServiceImpl implements DemoService {
    // .........
}

// Controller层
@Controller
public class DemoController {

    @Resource(name="a") // 使用a声明的service注入
    private DemoService demoService;
}
```

3、按照`setter`方法注入
``` java
// Service层
@Service
public class DemoServiceImpl implements DemoService {
    .........
}

// Controller层
@Controller
public class DemoController {

    private DemoService demoService;

    @Autowired
    public void setDemoService(DemoService demoService){
        this.demoService = demoService;
    }
}
```

4、按照构造器方式注入  
**说明：**构造器注入可以避免`Field`注入的 循环依赖 问题，比如在`Demo1`中注入了`Demo2`，在`Demo2`中注入了`Demo1`，使用构造器注入，在项目启动的时候会抛出`BeanCurrentlyCreationException`提醒循环依赖。
``` java
// Service层
@Service
public class DemoServiceImpl implements DemoService {
    .........
}

// Controller层
@Controller
public class DemoController {

    private DemoService demoService;

    @Autowired
    public DemoController(DemoService demoService){
        this.demoService = demoService;
    }
}
```


### 2、Springboot 关于使用 aop
1、导入依赖，在`pom.xml`中导入AOP的依赖
``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```
2、编写AOP程序
``` java
@Component // 将这个类注入到容器中
@Aspect // 使用这个注解标注这个类
public class DemoAspect{

    @Around("execution(* com.xxx.service.*.*(..))")
    public Object recordTime(ProceedingJoinPoint p){
        p.proceed(); // 调用原始方法
    }
}
```
3、通知类型  
（1）`@Around` 环绕通知（需要自己调用`proceed`方法执行原始方法，下面其他的注解不用调用）  
此注解标注的通知方法在目标方法前、后都被执行。  
（2）`@Before` 前置通知  
此注解标注的通知方法在目标方法前被执行。  
（3）`@After` 后置通知  
此注解标注的通知方法在目标方法后被执行，无论是否有异常都会执行。  
（4）`@AfterReturning` 返回后通知  
此注解标注的通知方法在目标方法后被执行，有异常不会执行。  
（5）`@AfterThrowing` 异常后通知  
此注解标注的通知方法发生异常后执行。  

### 3、关于 Springboot 配置文件
1、配置文件加载优先级  
`application.properties`配置文件 > `application.yml`配置文件 > `application.yaml`配置文件。  
2、`properties`配置文件名必须是`application.properties`，如果想更改它的名字，必须在这里配置：
{% asset_img Springboot1.jpg %}
3、不止可以在配置文件中配置，还可以在java属性或者命令行配置  
（1）idea中设置  
- java属性配置  
在idea启动配置里有一项配置：VM options  
例如：可以在VM options里配置端口号：-Dserver.port=8080  
- 命令行配置  
同java属性配置，有一项配置：Program arguments  
例如：在Program arguments里配置端口号：--server.port=8080  

（2）在Dos命令行设置  
- java属性配置
java -Derver.port=8080 -jar demo.jar
- 命令行配置
java -jar demo.jar --server.port=8080

**注意优先级：**命令行 > java属性 > 配置文件  
4、多个.yml文件的使用方法  
SpringBoot默认加载的是`application.yml`文件，所以想要引入其他配置的yml文件，就要在`application.yml`中激活该文件，定义一个`application-student.yml`文件（注意：必须以application-开头）。  
`application.yml`中：（注意必须是在`application.yml`或者`.properties`)
```yml
spring:                    # 这个也是用来启动多环境的
  profiles:               # 如果有开发、测试、生产环境
    active: student      # 就用这个来启动
```
（1）赋值（如果是xml的话，它是自动完成的）  
`application-student.yml`配置文件代码：
``` yml
student:
  name: 小明
  age: 27
  sex: 男
```
方案一：使用`@Value`注解
``` java
@Component
public class Student {
    @Value("${student.name}")
    private String name;
    @Value("${student.age}")
    private Integer age;
    @Value("${student.sex}")
    private String sex;
    
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

   @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", sex='" + sex + '\'' +
                ", cat=" + cat +
                '}';
    }
```
**注意：**如果使用`application.properties`，需要使用`@PropertySource(value="application.properties")`来加载并在属性上使用`@Value("")`使用。  
方案二：使用`@ConfigurationProperties`
``` java
@Component
@ConfigurationProperties(prefix = "student")
public class Student {
    
    private String name;
    
    private Integer age;
   
    private String sex;
    
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

   @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", sex='" + sex + '\'' +
                ", cat=" + cat +
                '}';
    }
```

### 4、springboot集成mybatis需要的依赖
``` xml
<!-- mysql驱动 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.20</version>
        </dependency>
        <!-- mybatis --> 
       <!-- 这里这个可以换，比如如果要用druid的就导入druid的 --> 
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.2.6</version>
        </dependency>
        <dependency>
           <groupId>log4j</groupId>
          <artifactId>log4j</artifactId>
          <version>1.2.17</version>
       </dependency>
       <dependency>
          <groupId>org.mybatis.spring.boot</groupId>
          <artifactId>mybatis-spring-boot-starter</artifactId>
          <version>2.1.4</version>
      </dependency>

<!-----------------或者只需要以下依赖就可以了--------------------->

<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.6</version>
         </dependency>
         <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.20</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.4</version>
        </dependency>
```
在`application.yml`中配置数据源：
``` yml
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://localhost:3306/数据库名字?serverTimezone=UTC&userUnicode=true&characterEncoding=utf-8
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.mysql.cj.jdbc.MysqlDataSource #这个依赖导入哪个就用哪个
mybatis:
  type-aliases-package: 实体类包路径，比如com.cn.pojo
  mapper-locations: mapper的路径，比如classpath:mapper/*.xml
```

### 5、Springboot bean的管理
1、获取bean：
``` java
@Autowired
public ApplicationContext applicationContext;
```
2、非第三方bean管理  
我们将一个类注入到`spring`容器中成为一个`bean`对象，默认是单例；要想多例只需在该类上加上`@Scope("prototype")`注解，例如：  
``` java
@Scope("prototype")
@Component
public class Demo {}
```
3、第三方bean管理  
第三方bean：比如第三方的jar包，我们无法在第三方jar中某个类上面使用`@Component、@Service`等注解的情况下，就使用`@Bean`注解，举个场景：  
``` java
public void test(){
    // 我们想使用JSONObject，必须new
    JSONObject jSONObject = new JSONObject();
}
```
上面这个场景，我们不能在`JSONObject`类上面使用注解`@Component、@Service`来将它注入到容器中，但是我们可以这样：  
在`springboot`启动类里，把`JSONObject`类注入到`spring`容器中**（不过在启动类中定义是不推荐的，推荐的是在配置类中定义）**
``` java
@SpringBootApplication
public class SpringbootDemo{

    public static void main(String[] args){
        SpringApplication.run(SpringbootDemo.class,args);
    }

    @Bean // 将当前方法的返回值对象交给IOC容器管理，成为IOC容器bean
    public JSONObject jSONObject(){
        return new JSONObject();
    }
}
```
4、多个`bean`使用方法  
概述：  
通过`Spring`管理的类,默认是单例模式,但是如果有的类需要使用独立的属性,则需要配置为多例模式的. 但是多例模式不仅仅只是加一个声明,使用`@Autowired`进行注入,可能并不会是你想要的结果.因为多例模式的类是需要单独调用的。  
不搞清楚原理直接测试：  
需要多例的类上加上注解`@Scope(“prototype”)`
``` java
@Component
@Scope("prototype")
public class ExampleService{

    public void test(){
        System.out.println("test,current bean is" + this);
    }
    
}

引用直接使用@Autowired
@Controller
public class ExampleService{

    @Autowired
    private ExampleService exampleService;
    
    @RequestMapping("test")
    public void test(){
        exampleService.test();
    }
    
}


结果: 每个request过来的时候,exampleService实例均为同一个实例.

解决办法：
第一种：不使用@Autowired

@Controller
public class ExampleService{

    @Autowired
    private org.springframework.beans.factory.BeanFactory beanFactory;
    
    @RequestMapping("test")
    public void test(){
        ExampleService exampleService = beanFactory.getBean(ExampleService.class);
        exampleService.test();
    }
}

第二种：使用bean工厂

@Autowired
private ApplicationContext context;


@Bean
public WebSocketHandler websocketBHandler() {
    PerConnectionWebSocketHandler perConnectionHandler = new PerConnectionWebSocketHandler(WebSocketBHandler.class);
    perConnectionHandler.setBeanFactory(context.getAutowireCapableBeanFactory());
    //设置bean工厂，否则bean工厂WebSocketBHandler将不会自动连接
    return perConnectionHandler;
}

然后使用ApplicationContext进行代理bean工厂

注入
@Autowired
private ApplicationContext context;
使用
this.Bservice = context.getBean(BService.class, this);
```

### 6、Springboot自动配置加载流程
1、说明  
（1）导入 `starter` ，就会导入 `autoconfigure` 包；  
（2）`autoconfigure` 包里面有一个文件： `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` ，
里面指定的是启动要加载的所有自动配置类；  
（3）`@EnableAutoConfiguration` 会自动的把上面文件里面写的所有自动配置类都导入进`spring`容器；  
（4）自动配置类中的 `xxxAutoConfiguration` 是有条件注解，来进行按需加载的；  
（5）`xxxAutoConfiguration` 给容器中导入一堆组件，组件都是从 `xxxProperties` 中提取属性值的；  
（6）`xxxProperties` 又是和配置文件进行了绑定。  
2、`springboot`默认配置说明  
（1）包含了 `ContentNegotiatingViewResolver` 和 `BeanNameViewRseolver` 组件，方便视图解析。  
（2）默认的静态资源处理机制：静态资源在static文件夹下即可直接访问。  
（3）自动注册了 `Converter`，`GenericConverter`，`Formatter` 组件，适配常见数据类型转换和格式化需求。  
（4）支持 `HttpMessageConverters`，可以方便返回`json`等数据类型。  
（5）自动使用 `ConfigurableWebBindingInitializer`，实现消息处理、数据绑定、类型转化、数据校验等功能。  
**重要：**  
（1）如果想保持默认配置，并且自定义更多配置，如：`interceptors（拦截器）、formatters（格式化器）、viewControllers（视图解析器）`等，可以创建一个新类，使用 `@Configuration` 注解和实现一个 `WebMvcConfigurer`接口，并且不要标注 `@EnableWebMvc` 注解。  
（2）如果想保持默认配置，但要自定义核心组件实例，比如：`RequestMappingHandlerMapping，RequestMappingHandlerAdapter，ExceptionHandlerExceptionResolver`等，只需给容器中放一个 `WebMvcRegistrations` 组件即可。  
（3）如果想全面接管`SpringMvc`，`@Configuration`标注一个配置类，并加上`@EnableWebMvc`注解，实现`WebMvcConfigurer`接口。  
3、`WebMvcConfigurer`接口方法说明  
（1）`addArgumentResolvers`：参数解析器，用来解析`Controller`中各方法的参数。  
（2）`addCorsMappings`：跨域  
（3）`addFormatters`：格式化器，用来处理配置文件中格式化的问题。  
（4）`addInterceptors`：拦截器  
（5）`addResourceHandlers`：资源处理器，处理静态资源规则。  
（6）`addReturnValueHandlers`：返回值处理器，当`controller`返回一个字符串或者对象等时，是要跳转页面还是返回数据。  
（7）`addViewControllers`：视图控制器，如果想发一个请求 `/a` 直接跳转到`xxx.html`页面，不用写`controller`，可以在这里设置。  
（8）`configureAsyncSupport`：异步支持  
（9）`configureContentNegotiation`：内容协商  
（10）`configureHandlerExceptionResolvers`：配置异常处理解析器。  
（11）`configureMessageConverters`：配置消息转换器  
（12）`configurePathMatch`：路径匹配  
（13）`configureViewResolvers`：配置视图解析器  

<br/>

***

<br/>

# 二、Springboot使用方法
### 1、要想在tomcat中启动项目
``` java
@SpringBootApplication
public class SpringbootApplication extends SpringBootServletInitializer {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootApplication.class, args);
    }

    //要想在tomcat中使用war包启动springboot项目，首先 先继承 SpringBootServletInitializer
    //然后在使用下面的方法
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(SpringbootApplication.class);
    }
}
```

### 2、在配置文件中配置静态资源
``` yml
spring: 
    mvc: 
        static-path-pattern: /res/**  #修改静态资源访问路径，项目路径+/res/**
    resources: 
        static-locations: classpath:/zuo  #修改静态资源存放路径，修改之后只有将静态资源放到zuo目录下才能被访问到。
```

### 3、springboot各种pom文件需要和学习指南地址
[springboot各种pom文件需要和学习指南地址](https://docs.spring.io/spring-boot/docs/2.0.3.RELEASE/reference/htmlsingle/#using-boot-starter)

### 4、Springboot读配置的4种方式
###### （1）@Value注解
**使用该注解需要注意：**  
1、该类必须注入`spring`容器中，才能使用`@Value`注解获取配置文件中的数据。  
2、配置文件里必须有该属性，不然启动会报异常。
``` java
@Component
public class Demo {

    // @Value("${user.name:默认值}") 为了不报异常，可以加默认值，如果是："${user.name:}"，表示值为""
    @Value("${user.name}")
    private String userName; // 如果属性有static和final关键字的话是无法生效的
}
```

###### （2）@ConfigurationProperties注解
只需要指定配置文件中某一个key的前缀就可以了。例如：  
1、配置文件
``` yml
demo:
    userName: xxx
```
2、JavaDemo
``` java
@ConfigurationProperties(prefix = "demo")
public class Demo {

    private String userName;
}
```

###### （3）通过Environment类动态获取（是spring底层提供的API）
1、第一种实现方式，实现`EnvironmentAware`接口，重写`setEnvironment`方法。
``` java
@SpringBootTest
public class Demo implements EnvironmentAware {

    private Environment env;

    @Override
    public void setEnvironment(Environment e){
        this.env = e;
    }
	
	@Test
	public void test() {
	    String var = env.getProperty("demo.userName");
	    System.out.println("从配置文件获取" + var);
	}
}
```
2、第二种通过自动注入
``` java
@SpringBootTest
public class Demo {

    @Resource
    private Environment env;

    @Test
    public void test() {
        String var = env.getProperty("demo.name");
        System.out.println("从配置文件获取" + var);
    }
}
```
**以上三种是获取默认的配置文件，要想获取自定义的配置文件可以通过下面的方法：**  
（1）默认的配置文件：`application`开头  
（2）自定义：`demo.properties`

###### （4）@PropertySource注解
只能获取`properties`的配置文件。
``` java
@Configuration
@PropertySources(@PropertySource(value = "classpath:demo.properties",encoding = "utf-8"))
public class Demo {

    @Value("${user.name}")
    private String name;
}
```
要想获取yml的需要设置配置类：
``` java
@Configuration
public class MyYmlConfig {

    @Bean
    public static PropertySourcesPlacehokderConfigurer yamlConfigurer() {
        PropertySourcesPlacehokderConfigurer p = new PropertySourcesPlacehokderConfigurer();
        YamlPropertiesFactory y = new YamlPropertiesFactory();
        y.setResources(new ClassPathResource("demo.yml"));
        p.setProperties(Objects.requireNonNull(y.getObject()));
        return p;
    }
}
```
然后就可以通过第一种方式@Value注解获取。  

### 5、Springboot获取bean对象方法
###### （1）启动获取 ApplicationContext
在项目启动时先获取 `ApplicationContext` 对象，然后将其存储在一个地方，以便后续用到时进行使用。  
这里提供两种场景的获取：  
1、基于 `xml` 配置 `bean` 的形式，适用于比较古老的项目，已经很少使用了；  
2、基于 `SpringBoot` 启动时获取 `ApplicationContext` 对象；  
**1、基于 xml 的形式实现：**  
其中`applicationContext.xml` 为配置容器的`xml`，不过现在一般很少使用了。  
``` java
ApplicationContext ac = new FileSystemXmlApplicationContext("applicationContext.xml");
```
这里等于直接初始化容器，并且获得容器的引用。这种方式适用于采用 `Spring` 框架的独立应用程序，需要程序通过配置文件手工初始化 `Spring` 的情况。目前大多数 `Spring` 项目已经不再采用 `xml` 配置，很少使用了。  
**2、基于 SpringBoot 启动实现：**
``` java
@SpringBootApplication
public class ExampleApplication {

    public static void main(String[] args) {
        // 启动时，保存上下文，并保存为静态
        ConfigurableApplicationContext ac = SpringApplication.run(ExampleApplication.class, args);
        SpringContextUtil.setApplicationContext(ac);
    }
}

// 对应的 SpringContextUtil 类如下：
public class SpringContextUtil1 {

    private static ApplicationContext ac;

    public static <T> T getBean(String beanName, Class<T> clazz) {
        T bean = ac.getBean(beanName, clazz);
        return bean;
    }

    public static void setApplicationContext(ApplicationContext applicationContext){
        ac = applicationContext;
    }
}
```
两种方式都是在启动 `Spring` 项目时，直接获取到 `ApplicationContext` 的引用，然后将其存储到工具类当中。在使用时，则从工具类中获取 `ApplicationContext` 容器，进而从中获得 `Bean` 对象。

###### （2）通过继承 ApplicationObjectSupport
此种方式依旧是先获得 `ApplicationContext` 容器，然后从中获取 `Bean` 对象，只不过是基于继承 `ApplicationObjectSupport` 类实现的，具体实现代码：
``` java
@Component
public class SpringContextUtil extends ApplicationObjectSupport {

    public <T> T getBean(Class<T> clazz) {
        ApplicationContext ac = getApplicationContext();
        if(ac == null){
            return null;
        }
        return ac.getBean(clazz);
    }
}
```
**注意：**这里的 `SpringContextUtil` 类需要实例化。  
`ApplicationObjectSupport` 类图，我们看到它实现了 `ApplicationContextAware` 接口，在 `Spring` 容器初始化过程中回调方法 `setApplicationContext` 来完成 `ApplicationContext` 的赋值。

###### （3）通过继承 WebApplicationObjectSupport
`WebApplicationObjectSupport` 是 `ApplicationObjectSupport` 的一个实现类，提供了 `Web` 相关的支持。实现原理与 `ApplicationObjectSupport` 一样，具体实现代码如下：
``` java
@Component
public class SpringContextUtil extends WebApplicationObjectSupport {

    public <T> T getBean(Class<T> clazz) {
        ApplicationContext ac = getApplicationContext();
        if(ac == null){
            return null;
        }
        return ac.getBean(clazz);
    }
}
```
通过类图我们可以看到它是 `ApplicationObjectSupport` 的实现子类，此方式除了继承对象不同外，没有其他区别，都是基于 `getApplicationContext` 方法来获取。

###### （4）通过 WebApplicationContextUtils
Spring 提供了工具类 `WebApplicationContextUtils`，通过该类可获取 `WebApplicationContext` 对象，具体实现代码如下：
``` java
public class SpringContextUtil2 {

    public static <T> T getBean(ServletContext request, String name, Class<T> clazz){
        // 或者 WebApplicationContext webApplicationContext1 = WebApplicationContextUtils.getWebApplicationContext(request);
        WebApplicationContext webApplicationContext = WebApplicationContextUtils.getRequiredWebApplicationContext(request);
        // webApplicationContext1.getBean(name, clazz)
        T bean = webApplicationContext.getBean(name, clazz);
        return bean;
    }
}
```
这个方法很常见于 `SpringMVC` 构建的 Web 项目中，适用于 Web 项目的 B/S 结构。

###### （5）通过 ApplicationContextAware
通过实现 `ApplicationContextAware` 接口，在 Spring 容器启动时将 `ApplicationContext` 注入进去，从而获取 `ApplicationContext` 对象，这种方法也是常见的获取 Bean 的一种方式，推荐使用，具体实现代码如下：
``` java
@Component
public class SpringContextUtil3 implements ApplicationContextAware {

    private static ApplicationContext ac;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        ac = applicationContext;
    }

    public static <T> T getBean(Class<T> clazz) {
        T bean = ac.getBean(clazz);
        return bean;
    }
}
```
这种方式与前面通过 `BeanFactoryAware` 获得 `BeanFactory` 的思路一致。

###### （6）通过 ContextLoader
使用 `ContextLoader` 提供的 `getCurrentWebApplicationContext` 方法，也是常用的获取 `WebApplicationContext` 的一种方法，具体实现代码如下：
``` java
WebApplicationContext wac = ContextLoader.getCurrentWebApplicationContext();
wac.getBean(beanID);
```
该方法常见于 SpringMVC 实现的 Web 项目中。该方式是一种不依赖于 Servlet，不需要注入的方式。但是需要注意一点，在服务器启动时和 Spring 容器初始化时，不能通过该方法获取 Spring 容器。

###### （7）通过 BeanFactoryPostProcessor
Spring 工具类，方便在非 Spring 管理环境中获取 Bean。
``` java
@Component
public final class SpringUtils implements BeanFactoryPostProcessor{

    /** Spring应用上下文环境 */
    private static ConfigurableListableBeanFactory beanFactory;

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException{
        SpringUtilsS.beanFactory = beanFactory;
    }

    /**
    * 获取对象
    * @param name
    * @return Object 一个以所给名字注册的bean的实例
    * @throws BeansException
    */
    @SuppressWarnings("unchecked")
    public static <T> T getBean(String name) throws BeansException{
        return (T) beanFactory.getBean(name);
    }

    /**
    * 获取类型为requiredType的对象
    * @param clz
    * @return
    * @throws BeansException
    */
    public static <T> T getBean(Class<T> clz) throws BeansException{
        T result = (T) beanFactory.getBean(clz);
        return result;
    }

    /**
    * 如果BeanFactory包含一个与所给名称匹配的bean定义，则返回true
    * @param name
    * @return boolean
    */
    public static boolean containsBean(String name){
        return beanFactory.containsBean(name);
    }

    /**
    * 判断以给定名字注册的bean定义是一个singleton还是一个prototype。 如果与给定名字相应的bean定义没有被找到，将会抛出一个异常（NoSuchBeanDefinitionException）
    * @param name
    * @return boolean
    * @throws NoSuchBeanDefinitionException
    */
    public static boolean isSingleton(String name) throws NoSuchBeanDefinitionException{
        return beanFactory.isSingleton(name);
    }

    /**
    * @param name
    * @return Class 注册对象的类型
    * @throws NoSuchBeanDefinitionException
    */
    public static Class<?> getType(String name) throws NoSuchBeanDefinitionException{
        return beanFactory.getType(name);
    }

    /**
    * 如果给定的bean名字在bean定义中有别名，则返回这些别名
    * @param name
    * @return
    * @throws NoSuchBeanDefinitionException
    */
    public static String[] getAliases(String name) throws NoSuchBeanDefinitionException{
        return beanFactory.getAliases(name);
    }

    /**
    * 获取aop代理对象
    * @param invoker
    * @return
    */
    @SuppressWarnings("unchecked")
    public static <T> T getAopProxy(T invoker){
        return (T) AopContext.currentProxy();
    }
}
```
其中 ConfigurableListableBeanFactory 接口，也属于 BeanFactory 的子接口。

### 6、怎么使用好springboot自动配置
1、选场景：`spring-boot-starter-data-redis`  
然后找到这个场景的自动配置类 `xxxAutoConfiguration`  
2、写配置  
在这个自动配置类中，找到 `@EnableConfigurationProperties(xxxProperties.class)` 分析开启了哪些属性绑定关系,修改redis相关的配置  
3、分析组件  
在 `xxxAutoConfiguration` 这个自动配置类中分析，因为有两个方法，表示这个自动配置类给容器中放了两个组件，有一个组件（方法）叫 `stringRedisTemplate` ，然后去业务代码中使用自动装配注解装配这个方法。（使用组件的前提是，知道这个组件是干嘛的）  
4、定制化（如果不满足需求，需要定制化）  
自定义组件，自己写一个 `stringRedisTemplate` 方法，并放到容器中（自动配置类中的 `stringRedisTemplate` 这个方法上面标注了 `@ConditionalOnMissingBean`，这个注解的作用就是如果我们自己写了下面的方法，则这个自动配置类就不加载这个方法，容器中使用我们自己自定义的方法。）
5、例子：自定义starter启动类  
（1）步骤：  
- 创建 aliyun-oss-spring-boot-starter 模块。（只负责依赖管理）  
- 创建 aliyun-oss-spring-boot-autoconfigure 模块，在 starter 模块引入该模块
- 在 aliyun-oss-spring-boot-autoconfigure 模块中定义自动配置功能，并定义自动配置文件 META-INF/spring/xxx.imports （老版本的为 META-INF/spring.factories）

（2）实例（对应上面的步骤）  
1. 创建 aliyun-oss-spring-boot-starter 模块：
{% asset_img Springboot2.jpg %}
{% asset_img Springboot3.jpg %}
{% asset_img Springboot4.jpg %}
2. 创建 aliyun-oss-spring-boot-autoconfigure 模块，在 starter 模块引入该模块，pom文件同上：
{% asset_img Springboot5.jpg %}
注意：没有启动类、配置文件、测试类。
- 在步骤`1.`创建的模块pom文件中加入步骤`2.`创建的模块依赖：
``` xml
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-oss-spring-boot-autoconfigure</artifactId>
    <version>0.0.1-SNAPDHOT</version>
</dependency>
```
- 在步骤`2.`创建的模块pom文件中引入oss依赖：
``` xml
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-sdk-oss</artifactId>
    <version>3.15.1</version>
</dependency>
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <version>2.3.1</version>
</dependency>
<dependency>
    <groupId>javax.activation</groupId>
    <artifactId>activation</artifactId>
    <version>1.1.1</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jaxb</groupId>
    <artifactId>jaxb-runtime</artifactId>
    <version>2.3.3</version>
</dependency>
```
- 在`2.`模块中创建 `AliOSSProperties` 类 和 `AliOSSUtils` 类：
``` java
@ConfigurationProperties(prefix = "aliyun.oss")
public class AliOSSProperties {
    private String endpoint;
    private String accessKeyId;
    private String accessKeySecret;
    private String bucketName;
    
    // get 和 set 方法.....
}
```
``` java
public class AliOSSUtils {
    
    private AliOSSProperties aliOSSProperties;

    // aliOSSProperties 的get方法
    public AliOSSProperties getAliOSSProperties() {
        return aliOSSProperties;
    }

    // aliOSSProperties 的set方法
    public AliOSSProperties setAliOSSProperties(AliOSSProperties aliOSSProperties) {
        this.aliOSSProperties = aliOSSProperties;
    }

    public String upload(MultipartFile file) throws IOException {
        String endpoint = aliOSSProperties.getEndpoint();
        String accessKeyId= aliOSSProperties.getAccessKeyId();
        String accessKeySecret= aliOSSProperties.getAccessKeySecret();
        String bucketName= aliOSSProperties.getBcketName();

        // 获取上传的文件输入流
        InputStrem inputStream = file.getInputStream();
        
        // 避免文件名重复导致被覆盖
        String originalFileName = file.getOriginalFilename();
        String fileName = UUID.randomUUID().toString() + originalFileName.substring(originalFileName.lastIndexOf("."));

        // 上传文件到OSS
        OSS ossClient = new OSSClientBuilder().build(endpoint,accessKeyId,accessKeySecret);
        ossClient.putObject(bucketName,fileName,inputStream);

        // 拼接文件访问路径
        String url = endpoint.split("//")]0] + "//" + bucketName + "." + endpoint.split("//")[1] + "/";

        // 关闭ossClient
        ossClient.shutdown();
        return url;
    }
}
```
- 在`2.`模块中创建 AliOSSAutoConfiguration 配置类：
``` java
@Configuration
@EnableConfigurationProperties(AliOSSProperties.class)
public class AliOSSAutoConfiguration {

    @Bean
    public AliOSSUtils aliOSSUtils(AliOSSProperties aliOSSProperties) {
        AliOSSUtils aliOSSUtils = new AliOSSUtils();
        aliOSSUtils.setAliOSSProperties(aliOSSProperties);
        return aliOSSUtils;
    }
}
```

3. 在 `aliyun-oss-spring-boot-autoconfigure` 模块中定义自动配置功能，并定义自动配置文件 `META-INF/spring/xxx.imports` （老版本的为 `META-INF/spring.factories`）
- 在`2.`模块的`resource`目录下创建文件夹 `META-INF`，继续在`META-INF`文件夹下创建`spring`文件夹，继续在`spring`文件夹下创建：`org.springframework.boot.autoconfigure.AutoConfiguration.imports`文件
- 将 `AliOSSAutoConfiguration` 类的全类名填入`org.springframework.boot.autoconfigure.AutoConfiguration.imports`文件中


<br/>

***

<br/>

# 三、Springboot其他组件
### 1、全局异常处理器
说明：  
当出现异常的时候，会把异常抛到前端，抛出数据格式不符合规范，所以我们必须在每个controller中去处理异常，这样的话每个controller类中的方法都要去写这个处理的过程，太繁琐了，所以我们就要弄一个全局异常处理器。
``` java
@RestControllerAdvice
public class GlobalExceptionHandler{
    // 这个注解的作用是指定 要捕获的异常
    @ExceptionHandler(Exception.class)
    public Result ex(Exception ex){
        ex.printStackTrace();
        return Result.error("对不起，操作失败，请联系管理员");
    }
 }
```

### 2、过滤器和拦截器
###### 1、过滤器
`Filter`快速入门（是`javax.servlet`下的）：  
1、定义`Filter`  
定义一个类，实现`Filter`接口，并重写其所有方法。  
2、配置`Filter`  
`Filter`类上加`@WebFilter`注解，配置拦截资源的路径，启动类上加`@ServletComponentScan`开启`Servlet`组件支持。
``` java
// urlPatterns ：要拦截什么样的请求，"/*"表示所有
@WebFilter(urlPatterns = "/*")
public class DemoFilter implements Filter {
    // 初始化方法，Web服务器启动，创建Filter时调用，只调用一次
    public void init(FilterConfig filterConfig){
        Filter.super.init(filterConfig);
    }
    // 拦截到请求时，调用该方法，可调用多次
    public void doFilter(ServletRequest request,ServletResponse response,FilterChain chain){
        // 放行
        chain.doFilter(request,response);
    }
    // 销毁方法，服务器关闭时调用，只调用一次
    public void destroy(){
        Filter.super.destroy();
    }
}
```

###### 2、拦截器
`Interceptor`快速入门：  
1、定义拦截器  
实现`HandlerInterceptor`接口，并重写其所有方法。  
2、注册拦截器  
``` java
@Component
public class LoginCheckInterceptor implements HandlerInterceptor{
    // 目标资源方法执行前执行，返回true：放行；返回false：不放行
    @Override
    public boolean preHandle(HttpServletRequest req,HttpServletResponse resp,Object handler) throws Exception{
        return true;
    }
    // 目标资源方法执行后执行
    @Override
    public void postHandle(HttpServletRequest req,HttpServletResponse resp,Object handler,ModelAndView modelAnd View){
        
    }
    //  视图渲染完毕后执行，最后执行
    @Override
    public void afterCompletion(HttpServletRequest req,HttpServletResponse resp,Object handler,Exception ex){
        
    }
}
```
3、添加拦截器配置  
``` java
@Configuration
public class WebConfig implements WebWvcConfigure{

    @Autowired
    private LoginCheckInterceptor loginCheckInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry){
        registry.addInterceptor(loginCheckInterceptor).addPathPatterns("/**");
    }
}
```
4、拦截路径  
（1）`/*` ：能匹配 `/dept、/emps、/login`，不能匹配 `/dept/list`。  
（2）`/**` ：能匹配 `/dept、/dept/list、/dept/list/find`。  
（3）`/dept/*` ：能匹配 `/dept/list`，不能匹配 `/dept/list/find、/dept`。  
（4）`/dept/**` ：能匹配 `/dept、/dept/list、/dept/list/find`。

###### 3、过滤器和拦截器的区别
1、执行流程  
（1）浏览器发送请求  
（2）`Filter`过滤器  
（3）`DispatcherServlet`  
（4）`Interceptor`拦截器  
（5）`Controller`  
2、区别  
（1）接口规范不同  
`Filter`过滤器需要实现`Filter`接口，而拦截器需要实现`HandlerInterceptor`接口。  
（2）拦截范围不同  
过滤器`Filter`会拦截所有的资源，而`Interceptor`只会拦截spring环境中的资源。  
（3）拦截器是`spring`中的，只能作用于`DispatcherServlet`，过滤器是`servlet`。  
（4）拦截器基于java反射机制，过滤器基于函数回调。  
（5）拦截器只对`action`请求起作用，过滤器几乎所有请求都起作用。  
（6）拦截器可以多次被调用，过滤器只能在初始化的时候被调用一次。

### 3、热部署
###### 1、使用 jrebel 插件
编写玩代码后，使用 `jrebel` 这个插件自带的启动按钮启动，然后每次编写玩都按 `ctrl + f9` 使用热部署。

###### 2、设置IDEA热部署
设置后，我们就不用频繁的去手动重启，它会自动刷新。就是写好代码后，在IDEA中按`ctrl + F9`，再去浏览器刷新。如果不想按`ctrl + F9`，也可以这样设置，先设置，`Settings ---> Build ---> Compiler`，设置自动编译（`Builed project automatically`勾选），然后`ctrl + shift + alt + /`，点击`Registry`，勾选`compiler.automake.allow.app.running`。




<br/>

***

<br/>

# 四、Springboot使用过程中遇到的问题合集
### 1、springboot自动注入问题
1、使用了`@Component`注解还是无法获取到`bean`对象  
原因：是因为`springboot`扫描器没有扫描到，默认扫描`springboot`启动类同级的包。  
（1）解决方法一：需要到`springboot`启动类上加上包扫描注解`@ComponentScan`，而且不止要扫描需要扫描的包，还要扫描原来`springboot`启动类同级的包：`@ComponentScan({"xxx.xxx.xxx","yyy.yyy.yyy})`。  
（2）解决方法二：在`springboot`启动类上使用`@Import`注解，例子：`@Import({Demo1.class,Demo2.class})`，大括号里的`class`是需要被扫描到的类（不需要使用任何注解的普通类）。  
（3）解决方法三：新建一个配置类（使用`@Configuration`注解标注的类），把需要被扫描的类在这个配置类中使用`@Bean`注解标注，在启动类上使用`@Import`注解，将这个配置类导入。例子：  
``` java
// 配置类
public class DemoConfig{
    
    @Bean
    public Demo1 demo1(){
        return new Demo1();
    }

    @Bean
    public Demo2 demo2(){
        return new Demo2();
    }
}

// 启动类
@Import({DemoConfig.class})
@SpringBootApplication
public class Demo(){
    
    public static void main(String[] args){.....}
}
```
（4）解决方法四：新建一个配置类（使用`@Configuration`注解标注的类），把需要被扫描的类在这个配置类中使用`@Bean`注解标注，在新建一个选择器类实现`ImportSelector`接口，实现这个接口的方法，在这个方法中，将配置类返回，然后将这个选择器类在启动类上使用`@Import`注解导入，例子：  
``` java
// 配置类
public class DemoConfig{
    
    @Bean
    public Demo1 demo1(){ // 需要扫描的类
        return new Demo1();
    }

    @Bean
    public Demo2 demo2(){ // 需要扫描的类
        return new Demo2();
    }
}

// 选择器类
public class DemoSelect implements ImportSelector{

    public String[] selectImports(AnnotationMetadata i){
        return new String[]{"xxx.xxx.DemoConfig"}; // 上面的配置类包路径
    }
}

// 启动类
@Import({DemoSelect.class})  // 将上面选择器类导入
@SpringBootApplication
public class Demo(){
    
    public static void main(String[] args){.....}
}
```
**总结：**  
上面的方法太繁琐了，一般需要扫描的包中会提供给我们一个注解`@EnableXXXX`注解，这个注解封装了上面使用到`@Import`和`@ImportSelector`注解，我们直接在启动类上使用`@EnableXXXX`注解就可以了。