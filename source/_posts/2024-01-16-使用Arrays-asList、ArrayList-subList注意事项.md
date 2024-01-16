---
title: 使用Arrays.asList、ArrayList.subList注意事项
date: 2024-01-16 10:56:19
tags: 使用Arrays.asList、ArrayList.subList注意事项
categories: [后端技术,问题方法合集]
---

<br/>


***


<br/>

# 一、Arrays.asList
1、例子：
``` java
List<Integer> list = Arrays.asList(1, 2);
System.out.println(list);
System.out.println(list.contains(1));
System.out.println(list.contains(2));

// 输出
[1, 2]
true
true
```
往list中添加元素3：
``` java
list.add(3);
System.out.println(list.contains(3));

// 输出
抛出java.lang.UnsupportedOperationException异常
```
2、原因：  
查看源码得知，Arrays.asList方法中，返回的ArrayList不是我们new List那个ArrayList，我们new List那个ArrayList是位于java.util包下的，而这个Arrays.asList方法中返回的ArrayList是Arrays类的内部类，它继承了AbstractList类，没有重写add方法，所以当我们调用add方法时才会抛出异常。  
3、总结：  
使用工具类Arrays.asList()把数组转成集合时，不能使用其修改集合相关的方法，比如：add、remove、clear等方法，会抛出异常。  


<br/>


***


<br/>


# 二、ArrayList.subList
1、例子：  
``` java
List<String> list = new ArrayList<>();
lsit.add("三国演义");
list.add("西游记");
list.add("红楼梦");
list.add("水浒传");

List<String> listSub = list.subList(2, 4);

System.out.println(list);
System.out.println(listSub);

// 输出
["三国演义", "西游记", "红楼梦","水浒传"]
[ "红楼梦","水浒传"]
```
可以看出，subList方法返回的是list中索引从2（包含）到4（不包含）的元素集合。  
2、问题注意：  
（1）修改原集合的值，会影响子集合  
``` java
// 修改原集合第二个元素红楼梦 -》简爱
list.set(2, "简爱");

System.out.println(list);
System.out.println(listSub);

// 输出
["三国演义", "西游记", "简爱","水浒传"]
[ "简爱","水浒传"]
```
可以看出，修改原集合的值，子集合也会跟着被修改。  
（2）修改原集合的结构，对子集合操作时会抛出异常  
**注意：**只有在对子集合操作时才会抛出异常。  
``` java
// 向原集合添加新元素，会抛出ConcurrentModificationException异常
list.add("简爱");

System.out.println(list);
System.out.println(listSub);

// 输出
["三国演义", "西游记", "简爱","水浒传", "简爱"]
// 子集合输出时，会抛出ConcurrentModificationException异常
```
（3）修改子集合的值，会影响原集合  
``` java
listSub.set(0, "简爱");

System.out.println(list);
System.out.println(listSub);

// 输出
["三国演义", "西游记", "简爱","水浒传"]
[ "简爱","水浒传"]
```
可以看出，修改子集合元素，原集合元素也跟着被修改了。  
（4）修改子集合的结构，会影响原集合  
``` java
listSub.add("简爱");

System.out.println(list);
System.out.println(listSub);

// 输出
["三国演义", "西游记", "红楼梦","水浒传", "简爱"]
[ "红楼梦","水浒传", "简爱"]
```
3、问题解析：  
（1）查看subList方法源码  
``` java
// 得知subList方法中，调用了SubList类的构造函数
public List<E> subList(int fromIndex, int toIndex) {
    subListRangeCheck(fromIndex, toIndex, size);
    return new SubList(this, 0, fromIndex, toIndex);
}

// SubList类的构造函数源码
SubList(AbstractList<E> parent, int offset, int fromIndex, int toIndex) {
    this.parent = parent;
    this.parentOffset = fromIndex;
    this.offset = offset + fromIndex;
    this.size = toIndex - fromIndex;
    this.modCount = ArraryList.this.modCount;
}
```
可以看出，SubList类是ArrayList的内部类，该构造函数中也并没有重新创建一个新的ArrayList，所以修改原集合或子集合的元素，会相互影响。  
4、总结：  
（1）在定义方法时，如果返回值类型是List的话，一定要思考是否允许修改，如果不允许修改，在方法注释上，一定要说明清楚！  
（2）我们可以使用Guava提供的方法，去创建一个新的List：  
``` java
Lists.newArrayList(E... elements)
Sets.newHashSet(E... elements)
```