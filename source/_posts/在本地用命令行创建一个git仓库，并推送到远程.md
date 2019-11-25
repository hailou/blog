---
title: 在本地用命令行创建一个git仓库，并推送到远程
date: 2019-04-14 23:10:17
categories: Git
tags: 
	- Git
discription: 在本地用命令行创建一个git仓库，并推送到远程
copyright: ture
---

##### 首先，进入项目目录下（没有的话自己创建一个）
```
cd funeral_import
```
##### 在funeral_import目录下 初始化一个git仓库
```
git init 
```

<!-- more -->

##### 复制一个文件到funeral_import目录下，然后执行git add .，将“修改”从当前工作区存放到暂存区
```
git add .
```
##### 将暂存区中存放的文件提交到git仓库

```
git commit -m "first commit"
```
##### 在远端新建一个git代码库：funeral_import，将本地代码库的当前分支与远程的代码库相关联
```
git remote add origin http://xxxxx/administrator/funeral_import.git
```
##### 设置仓库属性，刚开发者用户加入到仓库中
##### 将本地代码库的当前分支推送到远程的代码库
```
git push -u origin master
```
##### 另：命令补充！查看git仓库的远程代码库的地址：git remote -v