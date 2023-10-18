---
title: Postgresql学习笔记
date: 2023-10-17 13:38:07
tags: Postgresql
categories: 数据库
---

***

# 一、Postgresql在linux（虚拟环境中）中安装教程
1、官网下载地址：  
[https://www.postgresql.org/download/](https://www.postgresql.org/download/)
{% asset_img postgresql1.jpg %}

**注意**：Postgresql15的版本，navicat连接不了，会报错误，原因自查。  

2、安装教程官网会给出  
{% asset_img postgresql2.jpg %}
``` cmd
# 下载安装rpm仓库（sudo表示使用root用户权限来进行操作）
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
# 安装
sudo yum install -y postgresql14-server
# 初始化数据库
sudo /usr/pgsql-15/bin/postgresql-14-setup initdb
# 允许自启
sudo systemctl enable postgresql-14
# 启动服务
sudo systemctl start postgresql-14
```

3、初始化（初始化后会创建一个用户postgres，我们需要重新给它设置密码）  
**注意**：这里的postgres用户就是linux中的用户，属于PostgreSQL Server的用户组。  
``` cmd
#删除密码
sudo passwd -d postgres
#设置密码
sudo passwd postgres
#切换成postgres用户（如果报权限不够就使用：su -postgres）
su postgres
#进入postgresql client
psql
#或者使用这个命令直接从别的用户进入postgresql client
sudo -u postgres psql
#修改数据库中postgres用户密码
ALTER USER postgres WITH PASSWORD 'root';
#退出
\q
```

4、因为我们想通过本地的navicat来进行连接，所以需要关闭防火墙或者打开5432端口  
**注意**：下面的命令可能会让你输入root密码来进行验证，如果不想可以在命令前面加上 sudo，如果出现“postgres 不在 sudoers 文件中。此事将被报告。”，[可点击此处来解决](https://blog.csdn.net/m0_59133441/article/details/121511380)。  
（1）查看防火墙状态
```
firewall-cmd --state
```
（2）停止firewall
```
systemctl stop firewalld.service
```
（3）禁止firewall开机启动
```
systemctl disable firewalld.service
```
或者
（1）开放指定端口
```
firewall-cmd --zone=public --add-port=5432/tcp --permanent
```
（2）关闭指定端口
```
firewall-cmd --zone=public --remove-port=5672/tcp --permanent
```
（3）重启防火墙
```
firewall-cmd --reloadl
```

5、修改配置文件  
（1）用root用户打开配置文件或者在下面命令前加上 sudo  
```
// 注意先看自己的版本是几，然后把下面路径中的14改为自己的版本
vim /var/lib/pgsql/14/data/postgresql.conf
```
在 postgresql.conf 文件中取消注释，修改listen_addresses为'*'表示监听任意地址
{% asset_img postgresql3.jpg %}
（2）同样修改另一个配置文件
```
// 注意先看自己的版本是几，然后把下面路径中的14改为自己的版本
vim /var/lib/pgsql/14/data/pg_hba.conf
```
在 IPv4 local connections下面新增一行：
```
host  all  all 0.0.0.0/0 scram-sha-256
```
{% asset_img postgresql4.jpg %}
（3）重启服务（ 需要查看自己的版本号，[查看方法请点击](https://dandelioncloud.cn/article/details/1597226823283064833) ）
```
sudo systemctl restart postgresql-14
```

6、然后就可以用navicat等工具连接了  

7、卸载Postgresql的方法，[请点击这里。](https://www.cnblogs.com/june-/articles/14276416.html)