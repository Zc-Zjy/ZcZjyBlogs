---
title: Java 将Object对象转为列表
date: 2024-01-16 10:51:26
tags: Java 将Object对象转为列表
categories: [后端技术,问题方法合集]
---


<br/>


***

<br/>


**说明：**在Java编程中，我们经常需要将一个对象（Object）转换为列表（List）的形式进行处理。这种转换通常用于数据的整理、存储和展示等操作。本文将介绍几种常用的方法来实现将Object转为列表的操作，并提供相应的代码示例。  
# 方法一：手动转换
**说明：**最简单的方法是手动将Object的属性逐个提取出来，再将其添加到列表中。  
``` java
public class Person {
    private String name;
    private int age;
    // get、set方法...
}

// 方法一
public static List<Object> convertObjectToList(Person person) {
    List<Object> list = new ArrayList<>();
    list.add(person.getName());
    list.add(person.getAge());
    return list;
}

// 方法二
public List<Object> objToList(Object obj) {
    List<Object> list = new ArrayList<Object>();
    if (obj instanceof ArrayList<?>) {
        for (Object o : (List<?>) obj) {
            list.add(o);
        }
        return list;
    }
    return null;
}
```
使用这种方法的优点是灵活性较高，可以根据实际需求选择需要转换的属性。但是缺点是代码较为冗长，对于复杂的对象结构，需要编写大量的代码。  


<br/>


***


<br/>


# 方法二：Apache Commons BeanUtils
**说明：**Apache Commons BeanUtils是一个常用的Java工具库，其中提供了一个BeanUtils类，可以方便地将Java对象的属性转换为Map对象。然后，我们可以将Map对象转换为列表。    
``` java
import org.apache.commons.beanutils.BeanUtils;

public static List<Object> convertObjectToList(Object obj) throws Exception{
    Map<String, object> map = BeanUtils.describe(obj);
    return new ArrayList<>(map.values());
}
```
使用这种方法的优点是代码简洁，不需要手动提取对象的属性。但是缺点是需要引入额外的依赖库。  


<br/>


***


<br/>


# 方法三：使用反射
**说明：**反射是Java编程中的一个重要特性，可以在运行时动态地获取和操作对象的属性和方法。我们可以利用反射机制将Java对象的属性逐个提取出来，再将其添加到列表中。  
``` java
public static List<Object> convertObjectToList(Object obj) throws IllegalAccessException{
    List<Object> list = new ArrayList<>();
    Field[] fields = object.getClass().getDeclaredFields();
    for(Field field:fields){
        field.setAccessible(true);
        list.add(field.get(obj));
    }
    return list;
}
```
使用这种方法的优点是灵活性较高，可以适用于任意类型的对象。但是缺点是性能较差，对于大量的对象转换操作，会有一定的性能影响。  


<br/>


***


<br/>


# 方法四：使用Jackson库
**说明：**Jackson是一个流行的Java库，可以方便地将Java对象转换为JSON格式的字符串。我们可以利用Jackson库将Java对象转换为JSON字符串，再将其解析为列表。  
``` java
import com.fasterxml.jackson.databind.ObjectMapper;

public static List<Object> convertObjectToList(Object obj) throws Exception{
    ObjectMapper objectMapper = new ObjectMapper();
    String json = objectMapper.writeValueAsString(obj);
    return objectMapper.readValue(json, List.class);
}
```
使用这种方法的优点是简洁、效率较高，并且可以处理复杂的对象结构。但是缺点是需要引入额外的依赖库。