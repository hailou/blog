title: git查看某个文件的提交历史
date: 2019-11-21 10:39:42
categories: Git
tags: 
	- Git
---
##### 用法
* 显示该文件的修改记录
`git log --pretty=oneline 文件名`
* 接下来使用git show显示具体的某次的改动。
`git show <git提交版本号> <文件名>`
##### 如果出面乱码：如:
 
<!-- more -->
 
则要添加环境变量 `LESSCHARSET=utf-8`
 
若查看某一次提交的内容，执行下面命令（可以不加--stat，一般加上，不想看详细）
`git show 848255d52b288a6723849bfd5ae116ee1a7afc80 --stat`
 
