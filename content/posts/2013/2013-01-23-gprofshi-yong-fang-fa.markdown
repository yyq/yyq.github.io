---
layout: post
title: "Gprof使用方法"
date: 2013-01-23 11:44:01
comments: true
tags: [gprof]
---
### 简介
调研能够动态分析代码的工具，了解了一点Gprof。

简短总结：

* 必须完整执行一遍可执行文件，才可进行Gprof分析。
* 各个函数相互调用次数描述很到位，无调用的具体代码行号等信息，有助于分析流程。
* 各个函数时间消耗统计，有利于改进程序效率。

<!--more-->
主要功能：打印出程序运行中各个函数消耗的时间。找出众多函数中耗时最多的函数。产生程序运行时候的函数调用关系，包括调用次数，帮助分析程序的运行流程。

### 实现原理（这一段内容为摘抄）
通过在编译和链接你的程序的时候（使用 -pg 编译和链接选项），gcc 在你应用程序的每个函数中都加入了一个名为mcount ( or  “_mcount”  , or  “__mcount” , 依赖于编译器或操作系统)的函数，也就是说你的应用程序里的每一个函数都会调用mcount, 而mcount 会在内存中保存一张函数调用图，并通过函数调用堆栈的形式查找子函数和父函数的地址。这张调用图也保存了所有与函数相关的调用时间，调用次数等等的所有信息。

### 基本用法
 1. 编译的时候加上-pg的参数
 1. 执行应用程序，使之生成供gprof分析的数据
 1. 使用gprof程序分析数据
 
### Simple Example
 
 1. 新建一个c文件test.c
 
		#include "stdio.h"
		#include "stdlib.h"
		
		void a(){
			printf("\t\t+---call a() function\n");
		}
		
		void c(){
			printf("\t\t+---call c() function\n");
		}
		
		int b(){
			printf("\t+--- call b() function\n");
			a();
			c();
			return 0;
		}
		int main(){
			printf(" main() function()\n");
			b()；
		}
 1. 编译，不加-c选项，会默认进行编译并链接生成a.out
 
		gcc -pg test.c
	如果没有编译错误，gcc会在当前目录下生成一个a.out文件，当然你也可以使用 –o 选项给生成的文件起一个别的名字，像 gcc –pg test.c –o test , 则gcc会生成一个名为test的可执行文件,在命令行下输入[linux /home/test]$./test , 就可以执行该程序了，记住一定要加上 ./ 否则程序看上去可能是执行，可是什么输出都没有。
	
 1. 执行程序，得到给gprof分析的数据
 
		./a.out
	
	会生成一个gmon.out文件，这个文件就是供gprof分析使用的

 1. 使用gprof程序分析你的应用程序生成(在linux下有效，在mac下用了此命令无结果，奇怪)
 
		gprof -b a.out gmon.out
		
	![result](/images/20130123-1145gprof-test.png)
	
 1. 其他常用Gprof命令选项,请参考[link](http://os.51cto.com/art/200703/41426_1.htm)
		
		
### Complex Example
这里做的操作是将其应用到工程CFlow.

 1. 下载源码的链接：<http://www.gnu.org/software/cflow/>,我随意下载了一个1.0版本的
 1. 解压缩等命令
 	
		tar zxvf cflow-1.0.tar.gz
		./configure
		cd src
		make CFLAGS=-pg LDFLAGS=-pg
	
	如果没有出错,会在/home/cflow-1.0/src 目录下有一个名为cflow的可执行文件，这就是我们加入-pg编译选项后编译出来的可以产生供gprof提取信息的可执行文件。
	
	**记住一定要在编译和链接的时候都使用-pg选项**，否则可能不会产生gmon.out文件。对于cflow项目，CFLAGS=-pg 是设置它的编译选项，LDFLAGS=-pg是设置它的链接选项。
	
	当然你**也可以直接修改它的Makefile**来达到上述相同的目的，不过一定要记住编译和链接都要使用-pg选项。

 1. 执行这个程序,当前的src目录就有一个parser.c的c文件
 
		cflow parser.c
		
	在当前目录会出现一个gmon.out文件。
	
 1. 看结果
		
		gporf -b cflow gmon.out | less



		
		
		
		
		
		
		
		
		
		
		
		
