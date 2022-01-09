---
layout: post
title: "自己总结git常用命令手册"
date: 2012-12-16 17:01:01
comments: true
---
## 引言
之前偶尔也用git来存自己的代码，也fork别人的代码看看，还用什么github的图形界面来搞，后来那个图形界面实在是把我搞崩溃了，彻底决定不用GUI来操作github了，再加上学校的项目也得用这个东西，老师还让我调研清楚git各种协同合作的使用方法等等。我自己还想在github这上面弄了个博客（尽管借用的是别人用ruby写的octopress），后来自己还是决定从海量的书籍里先总结出，自己到底最常用的命令是哪些，记录在这个地方，以后自己要是忘记怎么用git了，就来这里看看
<!--more-->
## Git 使用手册

### 设置用户信息
	git config --global user.name "yyq"
	git config --global user.email yyq@me.com
如果去掉 --global选项重新配置，新的保存设定会保存在.git/config文件里
### 查看配置信息
	git config --list
	git config user.name
### 从当前目录初始化 and so on
	git init	
	git add README	
	git add .
	git commit -m 'any message'
### 从现有仓库克隆
	git clone https://github.com/yyq/blog.git
### 跟踪状态
	git status
### 提交更新
	git commit -m 'message'
一般提交更新之前，用git status检查一下，是不是都暂存起来了
	
	git commit -a -m 'message'
加上-a就不用管那么多了

### 移除文件,移动文件
	rm something
	git rm something

### 查看提交历史
	git log


## 远程仓库的使用

 * 默认情况下
 * git clone命令本质上，自动创建了本地的master分支，用于跟踪远程仓库中的master分支（假设远程仓库确实有master分支）
 * git pull ，一般的目的，都是要从原始克隆的远端仓库中抓取数据后，合并到工作目录中当前分支

### 克隆远程仓库，查看远程仓库
	git clone https://github.com/yyq/blog.git
	cd blog
	git remote -v
-v选项，显示对应的克隆地址
### 添加远程仓库
	git remote add XYZ  git://github.com....

### 从某仓库拿东西来
	git remote -v
	git fetch XYZ

## 推送到远程仓库
	git push [remote-name] [branch-name]
	git push origin master

把本地的master分支推送到origin服务器上，只有在  服务器上有写权限，并且同时没有其他人的推送数据，这才顺利执行

### 查看远程仓库信息
	git remote show origin

### 远程仓库的删除和重命名
	git remote rename XYZ XXX
	git remote rm XYZ

## 错误说明
### permission denied
生成一个ssh key了事，添加到github网站  和  工程里面
参考网址：<https://help.github.com/articles/generating-ssh-keys>
	
	cd ~/.ssh
	mkdir key_backup
	cp id_rsa* key_backup
	rm id_rsa*
	ssh-keygen -t rsa -C yyq@me.com
	
	pbcopy < ~/.ssh/id_rsa.pub




