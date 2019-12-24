title: WINDOWS下的MYSQL安装与卸载
date: 2019-11-21 10:43:15
categories: 开发总结
tags: 
	- mysql
---
##### 安装mysql
以管理员身份运行cmd，进入mysql-5.7.24-winx64\bin目录下，执行
 
```
mysqld install
```
发现很容易就提示服务为安装成功了。

<!-- more -->

创建data目录

启动mysql，发现mysql 没有启动起来.
```
net start mysql 
```
到data目录下，查看日志，找到xxx.err，密码在里面

初始化数据：
```
mysqld  --initialize
```
```
flush privileges;
```
```
set password for root@localhost = password('yihai123');
```
注意：一定要先flush privileges，再set，不然会报错，再启动mysql
data目录下，会有三个目录生成，此时启动mysql，发现启动成功了
```
mysql -u  root -p
```
 
##### 远程登录：
进入mysql,执行
 
```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'yihai123' WITH GRANT OPTION；
```
```
 FLUSH PRIVILEGES;
```
##### [来源](https://www.cnblogs.com/liongong/p/9901162.htm)