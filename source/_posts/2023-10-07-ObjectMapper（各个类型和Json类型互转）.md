---
title: ObjectMapper（各个类型和Json类型互转）
date: 2023-10-07 17:48:56
tags: Java开发中工具和方法
categories: Java开发中工具和方法
---

<br/>

---

<br/>

# 一、ObjectMapper（各个类型和Json类型互转）
**说明：**`ObjectMapper`类`(com.fasterxml.jackson.databind.ObjectMapper)`是`Jackson`的主要类，它可以帮助我们快速的进行各个类型和Json类型的相互转换。  

### 1、使用
（1）引入Jackson的依赖
``` xml
<!-- 根据自己需要引入相关版本依赖。 -->
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-core</artifactId>
  <version>2.9.10</version>
</dependency>
 
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
  <version>2.9.10</version>
</dependency>
 
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-annotations</artifactId>
  <version>2.9.10</version>
</dependency>
```

### 2、`ObjectMapper`的常用配置
``` java
private static final ObjectMapper mapper;
 
public static ObjectMapper getObjectMapper(){
    return this.mapper;
}
 
static{
    //创建ObjectMapper对象
    mapper = new ObjectMapper()
 
    //configure方法 配置一些需要的参数
    // 转换为格式化的json 显示出来的格式美化
    mapper.enable(SerializationFeature.INDENT_OUTPUT);
 
   //序列化的时候序列对象的那些属性  
   //JsonInclude.Include.NON_DEFAULT 属性为默认值不序列化 
   //JsonInclude.Include.ALWAYS      所有属性
   //JsonInclude.Include.NON_EMPTY   属性为 空（“”） 或者为 NULL 都不序列化 
   //JsonInclude.Include.NON_NULL    属性为NULL 不序列化
   mapper.setSerializationInclusion(JsonInclude.Include.ALWAYS);  
 
   
    //反序列化时,遇到未知属性会不会报错 
    //true - 遇到没有的属性就报错 false - 没有的属性不会管，不会报错
    mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
 
    //如果是空对象的时候,不抛异常  
    mapper.configure(SerializationFeature.FAIL_ON_EMPTY_BEANS, false);  
 
    // 忽略 transient 修饰的属性
    mapper.configure(MapperFeature.PROPAGATE_TRANSIENT_MARKER, true);
 
    //修改序列化后日期格式
    mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);  
    mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
 
   //处理不同的时区偏移格式
   mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
   mapper.registerModule(new JavaTimeModule());
 
}
```

### 3、`ObjectMapper`的常用方法
###### （1）json字符串转对象
``` java
ObjectMapper mapper = new ObjectMapper();
String jsonString = "{\"name\":\"Hyl\", \"age\":20}";
 
//将字符串转换为对象
Student student = mapper.readValue(jsonString, Student.class);
System.out.println(student); // 结果一
 
//将对象转换为json字符串
jsonString = mapper.writeValueAsString(student);
System.out.println(jsonString); // 结果二
 
 
结果一：
Student [ name: Hyl, age: 20 ]
结果二：
{
  "name" : "Hyl",
  "age" : 20
}
```
###### （2）数组和对象之间转换
``` java
//对象转为byte数组
byte[] byteArr = mapper.writeValueAsBytes(student);
System.out.println(byteArr); // 结果一
 
 
//byte数组转为对象
Student student= mapper.readValue(byteArr, Student.class);
System.out.println(student); // 结果二
 
结果一：
[B@3327bd23
结果二：
Student [ name: Hyl, age: 20 ]
```
###### （3）集合和json字符串之间转换
``` java
List<Student> studentList= new ArrayList<>();
studentList.add(new Student("hyl1" ,20 , new Date()));
studentList.add(new Student("hyl2" ,21 , new Date()));
studentList.add(new Student("hyl3" ,22 , new Date()));
studentList.add(new Student("hyl4" ,23 , new Date()));
 
String jsonStr = mapper.writeValueAsString(studentList);
System.out.println(jsonStr); // 结果一
        
List<Student> studentList2 = mapper.readValue(jsonStr, List.class);
System.out.println("字符串转集合：" + studentList2 ); // 结果二
 
结果一：
[ {
  "name" : "hyl1",
  "age" : 20,
  "sendTime" : 1525164212803
}, {
  "name" : "hyl2",
  "age" : 21,
  "sendTime" : 1525164212803
}, {
  "name" : "hyl3",
  "age" : 22,
  "sendTime" : 1525164212803
}, {
  "name" : "hyl4",
  "age" : 23,
  "sendTime" : 1525164212803
} ]
结果二：
[
	{
		name=hyl1, 
		age=20, 
		sendTime=1525164212803
	}, {
		name=hyl2, 
		age=21, 
		sendTime=1525164212803
	}, {
		name=hyl3, 
		age=22, 
		sendTime=1525164212803
	}, {
		name=hyl4, 
		age=23, 
		sendTime=1525164212803
	}
]
```
###### （4）map和json字符串之间转换
``` java
Map<String, Object> testMap = new HashMap<>();
testMap.put("name", "22");
testMap.put("age", 20);
testMap.put("date", new Date());
testMap.put("student", new Student("hyl", 20, new Date()));
 
 
String jsonStr = mapper.writeValueAsString(testMap);
System.out.println(jsonStr); // 结果一
Map<String, Object> testMapDes = mapper.readValue(jsonStr, Map.class);
System.out.println(testMapDes); // 结果二
 
结果一：
{
  "date" : 1525164212803,
  "name" : "22",
  "student" : {
    "name" : "hyl",
    "age" : 20,
    "sendTime" : 1525164212803,
    "intList" : null
  },
  "age" : 20
}
结果二：
{date=1525164212803, name=22, student={name=hyl, age=20, sendTime=1525164212803, intList=null}, age=20}
```
###### （5）日期转json字符串
``` java
// 修改时间格式
mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
Student student = new Student ("hyl",21, new Date());
student.setIntList(Arrays.asList(1, 2, 3));
 
String jsonStr = mapper.writeValueAsString(student);
System.out.println(jsonStr);
 
结果：
{
  "name" : "hyl",
  "age" : 21,
  "sendTime" : "2020-07-23 13:14:36",
  "intList" : [ 1, 2, 3 ]
}
```