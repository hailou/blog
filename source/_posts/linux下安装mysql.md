title: linux下安装mysql
categories: 环境部署
tags:
  - linux
  - mysql
date: 2019-12-24 16:12:09
---

下面记录了我在Linux环境下安装Mysql的完整过程，如有错误或遗漏，欢迎指正。

##### 安装前准备 

检查是否已经安装过mysql，执行命令

<!-- more -->

```
[root@localhost /]# rpm -qa | grep mysql
```
从执行结果，可以看出我们已经安装了mysql-libs-5.1.73-5.el6_6.x86_64，执行删除命令
```
[root@localhost /]# rpm -e --nodeps mysql-libs-5.1.73-5.el6_6.x86_64
```
再次执行查询命令，查看是否删除
```
[root@localhost /]# rpm -qa | grep mysql
```
查询所有Mysql对应的文件夹
```
[root@localhost /]# whereis mysql
mysql: /usr/bin/mysql /usr/include/mysql
[root@localhost lib]# find / -name mysql/data/mysql
/data/mysql/mysql
```
删除相关目录或文件
```
[root@localhost /]#  rm -rf /usr/bin/mysql /usr/include/mysql /data/mysql /data/mysql/mysql 
```
验证是否删除完毕
```
[root@localhost /]# whereis mysql
mysql:[root@localhost /]# find / -name mysql
[root@localhost /]# 
```
检查mysql用户组和用户是否存在，如果没有，则创建
```
[root@localhost /]# cat /etc/group | grep mysql
[root@localhost /]# cat /etc/passwd |grep mysql
[root@localhost /]# groupadd mysql
[root@localhost /]# useradd -r -g mysql mysql
[root@localhost /]# 
```
从官网下载是用于Linux的Mysql安装包
```
[root@localhost /]#  wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
```
##### 安装Mysql
在执行wget命令的目录下或你的上传目录下找到Mysql安装包：mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
执行解压命令：
```
[root@localhost /]#  tar xzvf mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
[root@localhost /]# ls
mysql-5.7.24-linux-glibc2.12-x86_64
mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
```
解压完成后，可以看到当前目录下多了一个解压文件，移动该文件到/usr/local/mysql执行移动命令：
```
[root@localhost /]# mv mysql-5.7.24-linux-glibc2.12-x86_64 /usr/local/mysql
```
在/usr/local/mysql目录下创建data目录
```
[root@localhost /]# mkdir /usr/local/mysql/data
```
更改mysql目录下所有的目录及文件夹所属的用户组和用户，以及权限
```
[root@localhost /]# chown -R mysql:mysql /usr/local/mysql
[root@localhost /]# chmod -R 755 /usr/local/mysql
```
编译安装并初始化mysql,**务必记住初始化输出日志末尾的密码（数据库管理员临时密码）**
```
[root@localhost /]# cd /usr/local/mysql/bin
[root@localhost bin]# ./mysqld --initialize --user=mysql --datadir=/usr/local/mysql/data --basedir=/usr/local/mysql
```
##### 补充说明：如果报错，请移步这里[查看解决方案](https://www.jianshu.com/p/276d59cbc529)

运行初始化命令成功后，输出日志如下：
**记录日志最末尾位置root@localhost:后的字符串，此字符串为mysql管理员临时登录密码。**

编辑配置文件my.cnf，添加配置如下
```
[root@localhost bin]#  vi /etc/my.cnf

[mysqld]
datadir=/usr/local/mysql/data
port = 3306
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
symbolic-links=0
max_connections=400
innodb_file_per_table=1
#表名大小写不明感，敏感为
lower_case_table_names=1
```
启动mysql服务器
```
[root@localhost /]# /usr/local/mysql/support-files/mysql.server start
```
查看是否存在mysql和mysqld的服务，如果存在，则结束进程，再重新执行启动命令
```
#查询服务
ps -ef | grep mysql
ps -ef | grep mysqld

#结束进程
kill -9 PID

#启动服务
 /usr/local/mysql/support-files/mysql.server start
```

添加软连接，并重启mysql服务
```
[root@localhost /]#  ln -s /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql 
[root@localhost /]#  ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql
[root@localhost /]#  service mysql restart
```

登录mysql，修改密码(密码为步骤5生成的临时密码)
```
[root@localhost /]#  mysql -u root -p
Enter password:
mysql>set password for root@localhost = password('yourpass');
```

开放远程连接
```
mysql>use mysql;
msyql>update user set user.Host='%' where user.User='root';
mysql>flush privileges;
```

设置开机自动启动
```
1、将服务文件拷贝到init.d下，并重命名为mysql
[root@localhost /]# cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
2、赋予可执行权限
[root@localhost /]# chmod +x /etc/init.d/mysqld
3、添加服务
[root@localhost /]# chkconfig --add mysqld
4、显示服务列表
[root@localhost /]# chkconfig --list
```
说明：本教程非原创，来源[这里](https://www.jianshu.com/p/276d59cbc529)
