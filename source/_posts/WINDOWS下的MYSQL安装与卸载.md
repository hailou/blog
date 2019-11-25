title: WINDOWS下的MYSQL安装与卸载
date: 2019-11-21 10:43:15
categories: 开发总结
tags: 
	- mysql
---
## 安装过程
###### 下载后解压
###### 以管理员身份运行cmd，进入mysql-5.7.24-winx64\bin目录下，执行mysqld install  发现很容易就提示服务为安装成功了。
###### 创建data目录

<!-- more -->

###### 启动mysql   net start mysql 发现mysql 没有启动起来.
###### 到data目录下，查看日志，找到xxx.err，密码在里面
###### 初始化数据：mysqld  --initialize
###### data目录下，会有三个目录生成，此时启动mysql，发现启动成功了
###### set password for root@localhost = password('yihai123');报错，所以要先flush privileges;再执行set password for root@localhost = password('yihai123');
###### mysql -u  root -p登录就行了
##### 远程登录：
###### 进入mysql,执行GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'yihai123' WITH GRANT OPTION；     FLUSH PRIVILEGES;


#### 来源https://www.cnblogs.com/liongong/p/9901162.html