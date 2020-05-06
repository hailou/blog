title: shell在指定目录下批量执行sql脚本的实例
categories: 环境部署
tags:
  - Git
date: 2020-04-14 14:24:29
---

###### shell在指定目录下批量执行sql脚本的实例

```shell
#!/bin/bash
#execute all script in specified directory
MYDATE=`date +%F'-'%T'-'%w`
MYSQL_PATH=/tmp/scripts #指定的目录
LOG_FILE=/tmp/scripts/exec_${MYDATE}.log
confirm=
db_name=
db_pass=
for file in ${MYSQL_PATH}/*
do
if [ -f "$file" ] ; then
postfix=`echo $file | awk -F'.' '{print "."$NF}'`
 if [ $postfix = ".sql" ] ; then
  if [ ! $db_name ] ; then #如果没有指定数据库
  read -p "请输入数据库名：" db_name
  read -p "你输入的数据名是【$db_name】，确认继续请输入--yes--: " confirm
  fi
  if [ "$confirm" = "yes" ] && [ -n $confirm ] ; then
  if [ ! $db_pass ] ; then #如果没有设置密码
   stty -echo #密码输入保护关闭显示
   read -p "请输入数据库密码：" db_pass
   echo -e "\n"
   stty echo
  fi
  mysql -uroot -p$db_pass -P3306 --default-character-set=utf8 ${db_name} < $file >& error.log
  echo $file 
  echo -e "\n===========$file=============\n" >>${LOG_FILE}
  cat error.log >>${LOG_FILE} #输出执行日志
  error=`grep ERROR error.log` #读取错误日志信息
  if [ -n "$error" ] ; then #如果有错误就退出程序
   echo $error
   exit
  fi
  else
  echo "您已经取消操作!"
  exit
  fi
 fi
fi
done
```
###### 问题

今天遇到这个问题: -bash: ./yihai.sh: /bin/bash^M: bad interpreter: No such file or directory，找到问题是因为用notepad++编写的脚本，需要转换一下，在命令行输入
```shell
sed -i "s/\r//" cleanup.sh
```
再次执行./yihai.sh,就可以了