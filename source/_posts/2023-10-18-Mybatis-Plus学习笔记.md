---
title: Mybatis-Plus学习笔记
date: 2023-10-18 15:09:24
tags: MybatisPlus
categories: [后端技术,MybatisPlus]
---

***

###### 1、MybatisPlus需要的依赖
``` xml
<!-- mysql连接驱动 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.27</version>
</dependency>
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.3.1</version>
</dependency>
<!-- 代码生成器依赖 -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.4.0</version>
</dependency>
```

<br/>

###### 2、配置日志输出
``` yaml
# 这个默认的日志，输出到控制台，其它的需要导入依赖
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

<br/>

###### 3、实体类中主键配置
``` java
/** type的值：
  * ID_WORKER： 全局唯一id，默认方式
  * AUTO： 自增，设置了这个之后，数据库里也一定要设置自增才会生效，不然会报错
  * NONE：  未设置主键
  * INPUT：  手动输入
  * UUID：  全局唯一id，uuid
  * ID_WORKER_STR：  ID_WORKER字符串标识法
**/
   @TableId(value = "id", type = IdType.INPUT)
    private String id;
```

<br/>

###### 4、自动填充数据
1、第一种：数据库操作  
在数据库表中创建两个字段，分别是create_time,update_time，填上默认值：CURRENT_TIMESTAMP （时间戳），然后将更新勾选上，然后需要在实体类中同步这两个属性，在实体类中创建这两个属性。  
2、第二种：代码级别  
在数据库和实体类都创建这两个字段，然后添加注解TableField，在创建时间属性上为@TableField(fill = FieldFill.INSERT)，在更新时间属性上为@TableField(fill = FieldFill.UPDATE)，然后创建一个配置类来处理，如下：
``` java
@Component
  public class MyMetaObjectHandler implement MetaObjectHandler {
    // 插入时填充
    @Override
    public void insertFill(MetaObject metaObject) {
        this.setFieldValByName("createTime",new Date(),metaObject);
        this.setFieldValByName("updateTime",new Date(),metaObject);
    }

    // 更新时填充
   @Override
    public void updateFill(MetaObject metaObject) {
        this.setFieldValByName("updateTime",new Date(),metaObject);
    }
}
```

<br/>

###### 5、注意版本区别
（1）mysql 5：驱动不同 com.mysql.jdbc.Driver  
（2）mysql 8：驱动不同 com.mysql.cj.jdbc.Driver，需要增加时区的配置 serverTimezone=GMT

<br/>

###### 6、使用方法
**特别说明：**dao要继承mybatis-plus的BaseMapper<T>，"<>"里的是具体的实体类，这里是什么类，增删改查就会针对这个实体类对应的数据库表来进行操作，如果需要查询其它表的信息，就必须自己在xml中写sql语句。
``` java
// 查询全部
List<User> users = userDao.selectList(null);
users.forEach(System.out::println);

// 根据ID更新数据
User user = users.get(1); // 这里要从0开始，意思0对应数据库里的第一条数据
user.setAge(28);
// 它是动态拼接sql更新语句的
int i = userDao.updateById(user);
System.out.println(i);

// 通过ID查询一条数据
 User user1 = userDao.selectById(1);
System.out.println(user1);

// 批量查询
 List<User> users = userDao.selectBatchIds(Arrays.asList(1, 2, 3, 4));
users.forEach(System.out::println);

// 动态条件查询
Map<String,Object> map = new HashMap<>();
map.put("name","左冲");
map.put("age",24); // 加条件是在sql语句后加 and age=24
List<User> users = userDao.selectByMap(map);
users.forEach(System.out::println);
```

<br/>

###### 7、乐观锁
1、介绍  
A、B两个线程更新一条数据，A取出这条数据获取到的version为1.0，A更新这条数据的同时，B也取出这条数据获取到的version也为1.0，A完成更新，version变为2.0，但是B更新的时候version是1.0，所以B更新失败。  
2、乐观锁实现方式  
（1）取出数据，获取当前version  
（2）更新时，带上这个version  
（3）执行更新时，set version = newVersion where version = oldVersion，如果version不对，就更新失败。  
3、操作  
（1）在数据库中增加version字段，对应的实体类中也要增加一个属性version，并加上注解@Version。
``` java
@Vsersion
private Integer version;
```
（2）添加配置类
``` java
@MapperScan("com.zuo.dao") // 扫描dao层
@EnableTransactionManagement // 开启事务管理
@Configuration // 配置类
public class MybatisPlusConfig {
    // 注册乐观锁插件
    @Bean
    public OptimisticLockerInterceptor optimisticLockerInterceptor() {
        return new OptimisticLockerInterceptor();
    }
}
```

<br/>

###### 8、分页
1、需要在第6点配置类中注入分页（新版本不知道需不需要配置）
``` java
@MapperScan("com.zuo.dao") // 扫描dao层
@EnableTransactionManagement // 开启事务管理
@Configuration // 配置类
public class MybatisPlusConfig {
    // 注册乐观锁插件
    @Bean
    public OptimisticLockerInterceptor optimisticLockerInterceptor() {
        return new OptimisticLockerInterceptor();
    }

    // 注册分页插件
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
}
```
2、操作
``` java
// new Page<>(1,1)：（pageNo, pageSize）
// Page<User>，尖括号里面的实体类是什么就针对对应的表来进行查询分页
Page<User> page = new Page<>(1,1);
// null：条件查询器，下面会有第9点
Page<User> userPage = userDao.selectPage(page, null);
System.out.println(userPage);
// page.getRecords()：是一个数组，它的样子是这样的
[
  User{id=1, name='左冲', sex='男', age=28}, 
  User{id=2, name='李四', sex='男', age=28}, 
  User{id=3, name='小明', sex='男', age=24}
]
System.out.println(page.getRecords());
page.getRecords().forEach(System.out::println);
```

<br/>

###### 9、逻辑删除
1、在实体类中增加字段delFlag字段，并在字段上面使用注解@TableLogic，还需要在数据库中增加一个逻辑删除标识字段del_flag，0：存在，1：删除该数据。  
**说明：**在数据库中添加这个字段之后，执行删除操作的时候，它会自动将删除操作转变为更新操作，将del_flag更改为1，表示删除此数据，再执行查找的时候就查找不到该数据，但是数据库中还是存在该数据。  
2、在配置类中注入逻辑删除组件，在配置文件中配置，新版本不知道需不需要配置：
``` java
// 配置类中配置
@Bean
public ISqlInjector sqlInjector() {
    return new LogicSqlInjector();
}
```
``` xml
# 配置文件中配置
mybatis-plus.global-config.db-config.logic-delete-field=0
mybatis-plus.global-config.db-config.logic-not-delete-value=1
```
3、操作
``` java
@TableLogic
private Integer delFlag;

int i = userDao.deleteById(2);
System.out.println(i);
List<User> users = userDao.selectList(null);
users.forEach(System.out::println);
```

<br/>

###### 10、条件查询器
``` java
QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.eq("name","小黑");
wrapper.or();
wrapper.eq("age",24);
List<User> users = userDao.selectList(wrapper);
users.forEach(System.out::println);
System.out.println("----------------");
Page<User> page = new Page<>(1,1);
Page<User> page1 = userDao.selectPage(page, wrapper);
page1.getRecords().forEach(System.out::println);
System.out.println("----------------");
List<Map<String, Object>> maps = userDao.selectMaps(wrapper);
maps.forEach(System.out::println);
System.out.println("---------------");
Page<User> page2 = userDao.selectPage(page, wrapper);
page2.getRecords().forEach(System.out::println);
```

<br/>

###### 11、代码生成器
**说明：**使用代码生成器之前，一定要配置好数据库。  
[查看官网使用方法](https://baomidou.com/pages/d357af/#%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B)