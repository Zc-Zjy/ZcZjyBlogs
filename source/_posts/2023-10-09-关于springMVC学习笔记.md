---
title: 关于springMVC学习笔记
date: 2023-10-09 13:54:52
tags: springMVC
categories: [后端技术,Spring]
---

***

# 一、SpringMVC相关知识整理
### 1、springMVC支持ant风格的路径
\?：表示任意单个字符  
\*：表示任意的0个或多个字符  
\*\*：表示任意的一层或多层目录  
注意：在使用 ** 时，只能使用 /**/xxx 的方式
``` java
// 前端发送的请求：localhost:8080/aaa/hello 或者 localhost:8080/a:a/hello
@RequestMapping("/a?a/hello")
public String hello(){}

// 前端发送的请求：localhost:8080/aaa/hello 或者 localhost:8080/aaaaa/hello
@RequestMapping("/a*a/hello")
public String hello(){}

// 前端发送的请求：localhost:8080/a/b/hello 或者 localhost:8080/a/b/c/d/hello
@RequestMapping("/**/hello")
public String hello(){}
```

### 2、SpringMVC常用组件
1、`DispatcherServlet`：  
**前端控制器**，不需要工程师开发，由框架提供，作用：统一处理请求和响应，整个流程控制的中心，由它调用其它组件处理用户的请求。  
2、`HandlerMapping`：  
**处理器映射器**，不需要工程师开发，由框架提供，作用：根据请求的`url`、`method`等信息查找`Handler`，即控制器方法。  
（1）`Handler`：**处理器**，需要工程师开发，作用：在`DispatcherServlet`的控制下，`Handler`对具体的用户请求进行处理。  
（2）`HandlerAdapter`：**处理器适配器**，不需要工程师开发，由框架提供，作用：通过`HandlerAdapter`对处理器（控制器方法）进行执行。  
（3）`ViewResolver`：**视图解析器**，不需要工程师开发，由框架提供，作用：进行视图解析，得到相应的视图，例如：`ThymeleafView`、`InternalResourceView`、`RedirectView`。  
（4）`View`：**视图**，不需要工程师开发，由框架或视图技术提供，作用：将模型数据通过页面展示给用户。

### 3、SpringMVC执行流程
说明：Handler就是Controller。  
1、用户向服务器发送请求，请求被`springMVC`前端控制器`DispatcherServlet`捕获；  
2、`DispatcherServlet`对请求`url`进行解析，得到请求资源标识符`uri`，判断请求`uri`对应的映射；（**特别说明：**`<mvc:default-servler-handler/>`的作用，如果没有配置，那么`springMVC`会把所有请求当作后端控制器`controller`的映射请求，如果请求地址没有映射到`controller`的，都会404，比如静态资源的请求。而配置上后，`springMVC`会解析所有请求，将后端控制器有的请求映射给`controller`，把控制器没有映射的请求交给默认的`servlet`）  
3、根据`uri`，调用`HandlerMapping`获得该`Handler`配置的所有相关对象（包括`Handler`对象以及对应的拦截器），最后以`HandlerExecutionChain`执行链对象的形式返回；  
4、`DispatcherServlet`根据获得的`Handler`，选择一个合适的`HandlerAdapter`；  
5、如果成功获得`HandlerAdapter`，此时将开始执行拦截器的`preHandler`方法；  
6、提取`Request`中的模型数据，填充`Handler`入参，开始执行`Handler`方法，处理请求；在填充`Handler`的入参过程中，根据配置，`spring`将做一些额外的工作：  
（1）`HttpMessageConveter`：将请求消息（如`json`）转换成一个对象，将对象转换为指定的响应信息。  
（2）数据转换：对请求消息进行数据转换，如`string`类型转换成`Integer`。  
（3）数据格式化：对请求消息进行数据格式化，如将字符串转换成格式化数字或格式化日期等。
（4）数据验证：验证数据的有效性（长度、格式等）。  
7、`Handler`执行完成后，向`DispatcherServlet`返回一个`ModelAndView`对象；  
8、此时将开始执行拦截器的`postHandler`方法；  
9、根据返回的`ModelAndView`（此时会判断是否存在异常，如果存在异常，则执行`HandlerExceptionResolver`进行异常处理）选择一个适合的`ViewResolver`进行视图解析，根据`Model`和`View`，来渲染视图页面；  
10、渲染视图完毕执行拦截器的`afterCompletion`方法；  
11、将渲染结果返回给客户端。


<br/>

***

<br/>


# 二、springMVC项目搭建
1、创建`Maven`工程项目
{% asset_img springMVC1.jpg %}

2、项目结构
{% asset_img springMVC2.jpg %}
如果没有web文件夹，我们就新建一个web文件夹，然后点击菜单栏 `File` -> `Project Structure`，然后根据下图所示创建：
{% asset_img springMVC3.jpg %}
{% asset_img springMVC4.jpg %}

3、添加pom文件需要的依赖，刷新maven
**特别注意：**springMVC项目的依赖和springboot的不同，springMVC依赖还需要手动去web目录下创建lib包，并把所需的依赖手动添加到lib包中才能生效！！！添加方法下面会讲到。  
``` xml
<dependencies>
<!--        mybatis依赖包中有,是test的-->
        <dependency>
            <groupId>commons-dbcp</groupId>
            <artifactId>commons-dbcp</artifactId>
            <version>1.4</version>
        </dependency>

<!--        mybatis-spring依赖包中有是provided，只有在编译和测试的时候使用-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.3.7</version>
        </dependency>

        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>jstl</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.3.7</version>
        </dependency>
		<!--        springMVC依赖包-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.3.7</version>
        </dependency>
    </dependencies>
```

4、在java包下创建`controller`包（xxx.xx.controller），并且新建`MyController`类
``` java
/*
* @Controller:创建控制器（处理器）对象
*   控制器：后端控制器，自定义的类处理请求
* */
@Controller
public class MyController {
    /*
    * @RequestMapping:请求映射
    *   属性：value 请求中的url地址，唯一值，以“/”开头
    *   位置：1、在方法的上面（必须的） 2、在类的上面（可选）
    *   作用：把指定的请求，交给指定的方法处理，等同于url-pattern
    * */
    @RequestMapping(value = "/some.do")
    public ModelAndView doSome(ModelAndView model){
        //存放数据
        model.addObject("msg","1111");
        model.addObject("fun","2222");
        //要返回的前端界面
        model.setViewName("show");
        return model;
    }

    /**
     * 当框架调用完doSome()方法后，得到返回中ModelAndView
     * 框架会在后续的处理逻辑值，处理mv多选里面的数据和视图
     * 对数据执行request.setAttribute("msg","处理了some.do请求")；把数据放入到request作用域
    * */
}
```

5、在resources文件夹下创建`spring-mvc.xml`、`applicationContext.xml`：  
（1）`spring-mvc.xml`
``` xml
<?xml version="1.0" encoding="utf-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/mvc
       https://www.springframework.org/schema/mvc/spring-mvc.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">

    <!--注解驱动，如果要使用阿里巴巴的fastjson，再导入依赖包之后就使用下面被注释的配置-->
    <!--    <mvc:annotation-driven>-->
    <!--        <mvc:message-converters register-defaults="true">-->
    <!--            <bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter" id="fastJsonHttpMessageConverter">-->
    <!--                <property name="supportedMediaTypes">-->
    <!--                    <list>-->
    <!--                        <value>text/html;charset=UTF-8</value>-->
    <!--                        <value>application/json</value>-->
    <!--                    </list>-->
    <!--                </property>-->
    <!--            </bean>-->
    <!--        </mvc:message-converters>-->
    <!--    </mvc:annotation-driven>-->
    <mvc:annotation-driven></mvc:annotation-driven>
    <!--静态资源过滤-->
    <mvc:default-servlet-handler/>
    <!--扫描包：controller-->
    <context:component-scan base-package="controller"></context:component-scan>
    <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<!-- value值里面的jsp要和创建的文件夹名字一致 -->
        <property name="prefix" value="/WEB-INF/jsp/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
</beans>
```
（2）`applicationContext.xml`
``` xml
<?xml version="1.0" encoding="utf-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 将spring导入 -->
<!--    <import resource="classpath:spring-dao.xml"></import>-->
<!--    <import resource="classpath:spring-service.xml"></import>-->
    <import resource="classpath:spring-mvc.xml"></import>
</beans>
```

6、在`web/WEB-INF`文件夹下创建jsp文件夹（文件夹名字自定义，要和`spring-mvc.xml`中视图解析器里的一致），然后在jsp文件夹下创建`show.jsp`
``` jsp
<%--
  Created by IntelliJ IDEA.
  User: 86182
  Date: 2023/10/9
  Time: 13:45
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
Hello world!
${msg}
${fun}
</body>
</html>
```
将下面代码复制粘贴进`web.xml`中：
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

   <!--DispatchServlet-->
       <servlet>
           <servlet-name>springMVC</servlet-name>
           <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
           <!-- 
             下面的init-param标签是用来初始化applicationContext.xml的路径，默认配置是可以不用配置init-param标签的，
             如果不配置，它的路径就是 /WEB-INFO 文件夹下，并且配置文件名字后缀还要默认加上 -servlet.xml ，
             完整名字就是 springMVC-servlet.xml 
           -->
           <init-param>
               <param-name>contextConfigLocation</param-name>
               <param-value>classpath:applicationContext.xml</param-value>
           </init-param>
           <!-- 设置的话，这个servlet会提前到服务器启动时启动 -->
           <load-on-startup>1</load-on-startup>
       </servlet>
       <servlet-mapping>
           <servlet-name>springMVC</servlet-name>
          <!-- /:不拦截.jsp，/*:拦截.jsp -->
          <!-- /：拦截路径，不拦截页面。
               /*：拦截所有的文件夹，不包含子文件夹 
              /**： 是拦截所有的文件夹及里面的子文件夹
           -->
           <url-pattern>/</url-pattern>
       </servlet-mapping>
</web-app>

```

7、设置lib依赖包，点击菜单栏 `File` -> `Project Structure`，然后根据下图所示创建：
{% asset_img springMVC5.jpg %}
{% asset_img springMVC6.jpg %}
{% asset_img springMVC7.jpg %}

8、项目启动配置
{% asset_img springMVC8.png %}
{% asset_img springMVC9.jpg %}
{% asset_img springMVC10.jpg %}
{% asset_img springMVC11.jpg %}
启动项目之后，在弹出的页面地址栏url后面加some.do访问成功！  
至此，简单的springMVC项目搭建成功，没有包含数据库连接，下面继续讲解包含数据库连接的搭建。  

<br/>

9、在`mysql`数据库中新建表，这个大家自定义，这里不讲解。  

10、在`pom.xml`中加入数据库连接所需要的依赖，并且刷新maven  
**注意：**因为springMVC项目还需要手动去lib包下添加依赖，所以还需要去项目结构那在lib包下添加项目所需依赖，添加方法上面有讲到。
``` xml
<!-- mybatis -->
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.2.6</version>
</dependency
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.3.1</version>
</dependency>
<!--数据库驱动-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.20</version>
</dependency>
```
如果出现  
`java.lang.AbstractMethodError: org.mybatis.spring.transaction.SpringManagedTransaction.getTimeout()Ljava/lang/Integer`  
异常，表示`mybatis`依赖包和`mybatis-spring`依赖包版本不兼容，可以查询版本兼容问题。  

11、在java包下创建`dao、service、pojo`包（xxx.xx.dao、xxx.xx.service、xxx.xx.pojo），并且新建`MyServiceImpl、MyPojo`类，`MyDao`接口：  
（1）`MyPojo`（对应数据库中的表，属性对应数据库中的字段）
``` java
package pojo;

/**
 * @创建人 zjy
 * @文件名 MyPojo
 * @创建时间 2023/10/9
 * @描述 TODO
 */
public class MyPojo {
    private Integer id;
    private String name;
    private String sex;
    private Integer age;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", sex='" + sex + '\'' +
                ", age=" + age +
                '}';
    }
}

```
（2）`MyServiceImpl`（用来处理业务的）
``` java
package service;

import dao.MyDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import pojo.User;

import java.util.List;

/**
 * @创建人 zjy
 * @文件名 MyServiceImpl
 * @创建时间 2023/10/9
 * @描述 TODO
 */
@Service
public class MyServiceImpl {

    @Autowired
    private MyDao myDao;

// 如果spring-service.xml中实现set注入的方式，就使用这个，并把上面@Service和@Autowired注解注释
//    public void setMyDao(MyDao loginDao) {
//        this.loginDao = loginDao;
//    }

    public List<MyPojo> findUsers() {
        return myDao.findUsers();
    }

}

```
（3）`MyDao`（用来处理数据库查询的接口）
``` java
package dao;

import pojo.User;

import java.util.List;

/**
 * @创建人 zjy
 * @文件名 MyDao
 * @创建时间 2023/10/9
 * @描述 TODO
 */
public interface MyDao {

    List<MyPojo> findUsers();
}

```

12、在`resources`目录下新建`mapper`文件夹、`jdbc.properties、mybatis-config.xml、spring-dao.xml、spring-service.xml`，在`mapper`目录下新建`MyMapper.xml`：  
（1）`MyMapper.xml`（用于写sql增删改查的）
``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace：要对应你的dao接口的包路径 -->
<mapper namespace="dao.MyDao">
	<!-- id：对应MyDao接口中的方法名；resultType：对应方法返回值，这里因为返回MyPojo，所以是MyPojo类的包路径 -->
    <select id="findUsers" resultType="pojo.MyPojo">
        select * from user
    </select>

</mapper>

```
（2）`spring-service.xml`
``` xml
<?xml version="1.0" encoding="utf-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">

    <!--扫描service包-->
    <context:component-scan base-package="service"></context:component-scan>

    <!--将我们所有业务注入到spring中，上面配置了扫描包，在serviceImpl加了@service注解，就不用配置了-->
	<!-- 因为我们使用的是注解@Autowired，所以我们在bean标签中使用自动注入的方式注入 -->
    <!-- <bean id="myServiceImpl" class="service.MyServiceImpl" autowire="byType"></bean> -->
	
	<!-- 这种方式是通过set注入，需要在MyServiceImpl类添加set方法 -->
	<!--    <bean id="myServiceImpl" class="service.MyServiceImpl">-->
    <!--        <property name="myDao" ref="myDao"></property>-->
    <!--    </bean>-->

    <!--声明式事务配置,需要导入 spring-jdbc依赖包 -->
    <bean id="transactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

</beans>

```
（3）`jdbc.properties`
``` properties
#这里mysql数据库用的是5的版本就用这个连接
#driverClassName=com.mysql.jdbc.Driver

#mysql数据库用的是8的版本用这个连接
driverClassName=com.mysql.cj.jdbc.Driver

#注意改下 3306/ 后面数据库的名字
url=jdbc:mysql://root@localhost:3306/test?useUnicode=true&characterEncoding=UTF-8&useSSL=true&serverTimezone=UTC
username=root
password=root
```
（4）`mybatis-config.xml`（用来配置mybatis的各种配置）
``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 将mapper注册到mybatis配置中 -->
    <mappers>
        <mapper resource="/mapper/UserMapper.xml"></mapper>
    </mappers>   
</configuration>
```
（5）`spring-dao.xml`
``` xml
<?xml version="1.0" encoding="utf-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">

    <!--关联数据库配置文件jdbc.properties 方法一-->
    <context:property-placeholder location="classpath:jdbc.properties"></context:property-placeholder>
	
	<!-- 关联数据库配置文件jdbc.properties 方法二 -->
<!--    <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer" id="propertyPlaceholderConfigurer">-->
<!--        <property name="location" value="classpath:jdbc.properties"></property>-->
<!--    </bean>-->

    <!-- org.apache.commons.dbcp.BasicDataSource需要 commons-dbcp依赖包-->
    <!--  destroy-method=”close”的作用是当数据库连接不使用的时候,就把该连接重新放到数据池中,方便下次使用调用. -->
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
		<!-- 下面四个value要对应jdbc.properties中的四个值，如果jdbc.properties中是jdbc.url，下面就要改成jdbc.url -->
        <property name="driverClassName" value="${driverClassName}" />
        <property name="url" value="${url}" />
        <property name="username" value="${username}" />
        <property name="password" value="${password}" />
    </bean>

    <bean class="org.mybatis.spring.SqlSessionFactoryBean" id="sqlSessionFactory">
        <property name="dataSource" ref="dataSource" />
        <property name="configLocation" value="classpath:mybatis-config.xml" />
		<!-- 如果没有在mybatis-config.xml配置mappers就使用这个 -->
		<!-- <property name="mapperLocations" value="classpath:mapper/*.xml"></property> -->
    </bean>

    <!--配置dao接口扫描包，动态的实现dao接口注入spring中-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="dao"></property>
		<!-- value对应上面SqlSessionFactoryBean的id -->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
    </bean>
</beans>

```

13、将`applicationContext.xml`中的注释打开，在`MyController`类中增加下面的属性：
``` java
@Autowired
public MyServiceImpl myService;
```
并且在`doSome`方法中增加下面的代码：
``` java
List<MyPojo> users = myService.findUsers();
model.addObject("users",users.toString());
```

14、在`show.jsp`增加`${users}`，启动项目，成功！

<br/>

***

<br/>

# 三、SpringMVC下载和上传
### 1、下载
``` java
@RequestMapping("/testDown")
public ResponseEntity<byte[]> testResponseEntity(HttpSession session) throws IOException{
    // 获取ServletContext对象
    ServletContext servletContext = session.getServletContext();
    // 获取服务器中文件的真实路径
    String realPath = servletContext.getRealPath("/static/img/1.jpg");
    System.out.println(realPath);
    // 创建输入流
    InputStream is = new FileInputStream(realPath);
    // 创建字节数组，is.available()方法是流的所有大小
    byte[] bytes = new byte[is.available()];
    // 将流读到字节数组中
    is.read(bytes);
    // 创建HttpHeaders对象设置响应头信息
    MultiValueMap<String,String> headers = new HttpHeaders();
    // 设置要下载方式以及下载文件的名字
    headers.add("Content-Disposition","attachment;filename=1.jpg");
    // 设置响应状态码
    HttpStatus statusCode = HttpStatus.ok;
    // 创建ResponseEntity对象，这个对象是返回包括了响应头和响应体
    ResponseEntity<byte[]> responseEntity = new ResponseEntity<>(bytes,headers,statusCode);
    // 关闭输入流
    is.close();
    return responseEntity;
}
```

### 2、上传
（需要导入上传依赖）
``` xml
<dependency>
    <groupId>commons-fileupload</froupId>
    <artifactId>commons-fileupload</artifaciId>
    <version>1.3.1</version>
</dependency>
```
1、需要在前端`form`表单`form`标签设置`enctype`属性为：`multipart/form-data`，这个值表示上传功能，默认值是：`application/x-www-form-urlencoded`，表示form表单值以：`username=xxx&password=xxxx`形式传给后端。  
2、在springMVC配置文件中配置：  
``` xml
<!-- 配置文件上传解析器，将上传的文件封装为MultipartFile -->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"></bean>
```
3、controller
``` java
@RequestMapping("/upload") // 上传是使用post方式
// MultipartFile 是springMVC封装好的上传对象
public String upload(MultipartFile file,HttpSession session){
    // 获取上传文件的文件名
    String fileName = file.getOriginalFilename();
    // 获取文件名的后缀
    String suffixName = fileName.substring(fileName.lastIndexOf(".");
    // 将uuid和后缀名拼接后的结果作为最终的文件名，以防止文件上传重复，然后导致文件被覆盖
    fileName = UUID.randomUUID().toString.replace("-") + suffixName;
    ServletContext servletContext = session.getServletContext();
    String filePath = servletContext.getRealPath("file");
    File file = new File(filePath);
    // 判断filePath所对应路径是否存在
    if(!file.exists()){
        file.mkdir();
    }
    String finalPath = filePath + File.separator + fileName;
    file.transferTo(new File(finalPath));
    return "success";
}
```