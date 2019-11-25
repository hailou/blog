---
title: 服务器安装JDK和supervisor
date: 2019-04-14 23:10:17
categories: 环境部署
tags: 
	- supervisor
discription: 服务器安装JDK和supervisor
copyright: ture
---
##### 查询系统自带的openjdk
```
rpm -qa | grep java | grep openjdk
```

##### 如果有，使用如下命令逐个全部卸载
```
rpm -e --nodeps 系统自带的jdk名
```

<!-- more -->

##### 检查是否卸载完成，卸载完成再进行下一步
```
rpm -qa | grep java | grep openjdk
```
##### 安装jdk，需要事先把jdk的rpm文件放到/root下
```
cd /root
rpm -ivh jdk-8u162-linux-x64.rpm
```
##### 检查是否安装成功
```
java -version
javac -version
```
##### 配置环境变量
```
vim /etc/profile
```
##### 在文件的最后增加如下内容
```
JAVA_HOME=/usr/java/jdk1.8.0_162
JRE_HOME=$JAVA_HOME/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export JAVA_HOME JRE_HOME PATH CLASSPATH
```
##### 使修改生效
```
source /etc/profile
```
 
##### 安装supervisor
```
yum install -y supervisor
```

##### 检查状态、启动、设置开机自动启动
```
systemctl status supervisord.service
systemctl start  supervisord.service
systemctl enable supervisord.service
systemctl status supervisord.service
```
 