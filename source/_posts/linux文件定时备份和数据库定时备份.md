title: linux文件定时备份和数据库定时备份
categories: 环境部署
tags:
  - linux
  - mysql
date: 2019-12-26 15:19:24
---
##### 图片文件复制
说明：每日凌晨1点0分定时将文件从<从数据库服务器>复制到<主数据库服务器>，只复制新增的文件。
在<主数据库服务器>上面运行如下命令

<!-- more -->

```
[root@exchange-db db-backup]# crontab -e
0 1 * * * rsync -va root@172.17.0.112:/home/nfs_yihai/* /home/nfs_yihai/
```
查看设置是否成功
```
crontab -l
```
##### mysql数据库定时备份
说明：每日凌晨1点0分定时备份数据。
创建db_backup.sh文件
内容如下：
```
#!/bin/bash
mysqldump -uroot -ppssword exchange | gzip > /home/db-backup/dbname_$(date +%Y%m%d_%H%M%S).sql.gz
```
加入到计划任务里
```
[root@exchange-db db-backup]# crontab -e
0 1 * * * /home/db-backup/db_backup.sh
```
查看设置是否成功
```
crontab -l
```
##### 遇到的问题
1. mysql和mysqldump出现`command not found `问题解决
- 查找mysql安装路径
```
find / -name mysql
```
通常mysql安装路径在:`/usr/local/mysql/bin/mysql`
- `mysql:command not found`建立软连接
```
ln -s  /usr/local/mysql/bin/mysql  /usr/bin
```
- `mysqldump:command not found `建立软连接
```
ln -s  /usr/local/mysql/bin/mysqldump  /usr/bin
```
2. 如果密码中包含@时会报错`-bash: !@#": event not found`，需要修改命令，给-p密码加上单引号，如`-p'yihai@123!@#'`
