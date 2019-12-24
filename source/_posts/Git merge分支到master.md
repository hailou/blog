---
title: git merge分支到master
date: 2019-04-14 23:10:17
categories: Git
tags: 
	- Git
discription: Git merge分支到master
copyright: ture
---

###### 本地拉一个分支出来
 
```
git checkout -b xxx
```
###### 开发完以后提交到远程分支
```
git add .
git commit -m "commit xxx"
git push -u origin xxx
```

<!-- more -->

###### 返回master
```
git checkout master
```
###### 把本地的分支合并到master
```
git merge xxx
```
###### 把本地的master同步到远程
```
git push origin master
```
###### 如果不需要本地或者远程的xxx分支了，你可以选择删除。