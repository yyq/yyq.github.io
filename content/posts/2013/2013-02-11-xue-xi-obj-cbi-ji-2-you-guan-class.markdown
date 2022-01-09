---
layout: post
title: "学习obj-c笔记2:有关class"
date: 2013-02-11 11:11:01
comments: true
tags: [objective-c,class] 
---
在obj-c中的类，主要包含以下两个部分：
 
 * @interface:classname
 * @implementation:classname

如果是一个程序而不是一个单独的类时，还得包括一个

 * program section，也即main函数等等
 
在interface section中，应该包括有property和Method的声明，关于property的使用，暂时还没注意到，不过看到了对于method的描述，简单的说就是，+开头的方法，都是类方法，-开头的方法，都是实例方法。不过我觉得自己的表述还是缺乏精确性。摘抄原文如下：
<!--more-->

	@interface NewClassName:ParentClassName
		prepertyAndMethodDeclarations;
	@end

> The leading minus sign(-) tells the Objective-C compiler that the method is an instance method.They only other option is a plus sign(+),which indicates a class method. A class method is one that performs some operation on the class itself,such as creating a new instacne of the class.

恩，class方法就是应用在class本身，当我们创建一个新的instance的时候，调用的alloc方法，应该就是一个class method.一般我们自己写的方法，估计八成都是instance method了。

在implementation部分，格式大多如下：

	@implementation NewClassName
	{
		memberDeclatations;
		//这里有些个变量，instance variables,嗯，也可以理解为该class的数据结构？
	}
	methodDefinitions;
	//这里就应该根据interface中声明的方法，进行一一实现，其中大概会有各种操作各位实例变量等。
	@end