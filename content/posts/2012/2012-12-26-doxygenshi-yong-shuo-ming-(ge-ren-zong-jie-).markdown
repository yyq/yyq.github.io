---
layout: post
title: "Doxygen使用说明（个人总结）"
date: 2012-12-26 13:10:01
comments: true
tags: [doxygen,tools,linux]
---
## Doxygen是什么
简单的说，在我看来，这就是一个生成注释的工具，跨平台，很好用。输入一个工程的源代码，它可以很好的把所有文档里的注释等信息挑出来。然后结合工程组织成html或者tex等。方便对我们对工程的了解。
可以参考[wikipedia](http://zh.wikipedia.org/wiki/Doxygen)

正好项目里有这样的需求，我就做个这么一个使用记录吧。
<!--more-->
## Doxygen使用说明
### 获得doxygen
 1. 如果用ubuntu系统的话，可以直接用如下这个命令下载安装即可，如果说找不到试试把sofeware source改成美国的服务器。如果直接从软件源下载的doxygen版本号较低，想升级，参考下一条。
 	
>sudo apt-get install doxygen doxygen-gui graphviz
 
 1. 如果使用其他系统，可以直接从[官网](http://www.stack.nl/~dimitri/doxygen/)下载最新版本可执行文件就ok。
 
 
### 图形界面使用doxygen
 
 1. 在ubuntu下，使用命令打开图形界面程序；(mac&rarr;doxygen.app;windows&rarr;doxygen.exe)
 
>doxywizard

>sudo doxywizard		//如果没有读写文件权限的话，加sudo
 	
 1. 设置参数：
 
 	*Specify the working directory from which doxygen will run*
 	
 	运算过程中的临时文件，以及默认的结果文档都会在这个目录

 1. **Wizard** &rarr; **Project**标签中，
 
 	*Source code directory* ：源代码的根目录位置
	
 	*Scan recursively*		：这个参数要画勾，递归浏览目录
 	
 1. **Wizard**&rarr;**Output**标签中
 
  	*HTML*&rarr;*with navigation panel*这个最好标记上，在生成的html文档中有导航栏
 	

 1. **Expert**&rarr;**Build**标签中，
 
  	*EXTRACT_STATIC* 得画上勾，不然会出现生成的文档中缺少静态函数变量等
  	
 1. **Run**标签中
 
  	*Run doxygen*

### 命令行使用doxygen

 1. 生成配置文件	
 		
>doxygen -g config_file_name
 
 1. 打开这个配置文件，修改参数。

	一定要修改的三个参数：**INPUT** ， **RECURSIVE** ， **output**

	可以根据上述图形界面来修改参数，或者：在图形界面自己选好配置，然后导出配置文件。

 1. 开始执行
	
>doxygen config_file_name









