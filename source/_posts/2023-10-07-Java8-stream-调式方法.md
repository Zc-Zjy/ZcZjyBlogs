---
title: Java8 stream 调式方法
date: 2023-10-07 16:18:28
tags: 问题方法合集
categories: [后端技术,问题方法合集]
---

<br/>

---

<br/>

**说明：**`Java`的`Stream`编程给调试带来了极大的不便，IDEA推出了stream trace功能，可以详细看到每一步操作的关系、结果，非常方便进行调试。  

<div align="center">
	<font size="50">StreamTrace 用法</font>
</div>

### 1、StreamTrace 例子  
这里简单将字符串转成它的字符数，并设置断点开启debug模式。
{% asset_img 1.png %}
如上图所示，可以看到每一步操作的元素个数、操作的结果、元素转换前后的对应关，非常清晰明了；还可以查看具体的对象内容。  

### 2、使用 StreamTrace 方法
StreamTrace只有在debug模式下才能使用，当在Stream代码上设置断点后，启动debug，点击流按钮，如图所示：
{% asset_img 2.png %} 
<br/>

点击后，默认Split 模式显示：
{% asset_img 3.png %} 
<br/>

可以点击左下方按钮切换到FlatMode模式，当然也可以再切换回去：
{% asset_img 4.png %}

### 3、实战演练
这里演示一段字符转长度并过滤长度小于5的stream操作：  
``` java
@Test    
public void TestTrace() {
    Stream.of("beijing","tianjin","shanghai","wuhan")                
        .map(String::length)               
        .filter(e -> e > 5)               
        .collect(Collectors.toList()); 
}
```
{% asset_img 5.png %}
