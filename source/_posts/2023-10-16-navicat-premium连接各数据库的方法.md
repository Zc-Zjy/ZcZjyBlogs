---
title: navicat premium连接各数据库的方法
date: 2023-10-16 17:46:56
tags: navicat premium连接各数据库的方法
categories: [后端技术,问题方法合集]
---

***

# 一、navicat premium连接oracle数据库
`jdbc:oracle:thin:@111.205.100.117:1521:dhcctest`
{% asset_img navicat1.jpg %}
1.连接名：可以随便取  
2.主机：对应  111.205.100.117  
3.端口：对应 1521  
4.服务名：对应  dhcctest  （SID）  

<br/>

***

<br/>

# 二、navicat premium连接Postgresql数据库
`jdbc:postgresql://192.168.51.182:5432/ims`
{% asset_img navicat2.jpg %}
1.连接名：可以随便取  
2.主机：对应  192.168.51.182  
3.端口：对应 5432  
4.初始化数据库：对应ims  

<br/>

***

<br/>

# 三、navicat premium连接Mysql数据库
`jdbc:mysql://localhost:3306/test`
{% asset_img navicat3.jpg %}
1.连接名：可以随便取  
2.主机：对应  localhost  
3.端口：对应 3306  