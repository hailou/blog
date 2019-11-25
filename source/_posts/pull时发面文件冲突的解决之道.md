title: pull时发面文件冲突的解决之道
date: 2019-11-21 10:41:52
categories: Git
tags: 
	- Git
---
###### stash翻译为“隐藏”，如下操作：
`git stash`
`git pull`
`git stash pop`
`git diff`

<!-- more -->

##### git stash:
备份当前的工作区的内容，从最近的一次提交中读取相关内容，让工作区保证和上次提交的内容一致。同时，将当前的工作区内容保存到Git栈中。
##### git stash pop:
从Git栈中读取最近一次保存的内容，恢复工作区的相关内容。由于可能存在多个Stash的内容，所以用栈来管理，pop会从最近的一个stash中读取内容并恢复。
##### git stash list: 
显示Git栈内的所有备份，可以利用这个列表来决定从那个地方恢复。

##### git stash clear: 
清空Git栈。此时使用gitg等图形化工具会发现，原来stash的哪些节点都消失了。

