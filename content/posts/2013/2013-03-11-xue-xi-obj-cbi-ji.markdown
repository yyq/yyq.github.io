---
layout: post
title: "学习obj-c笔记"
date: 2013-03-11 00:24:01
comments: true
tags: [objective-c]
---
It took me sometime to finish the chapter nine. This chapter is about Dynamic Typing and Dynamic Binding and so on.
<!--more-->
## Dynamic Typing 
sample code:

	id  data;
	ClassA *classa = [[ClassA alloc]init];
	ClassB *classb = [[ClassB alloc]init];
	data = classa;
	[data method_of_a];
	data = classb;
	[data method_of_b];
	
>The Objective-C system always keep track of the class to which an object belongs.

when you use something like this:

	data = classa;
	[data method_of_b];
	
>No error message is reported until you run the program containing these lines;

>When you use static typing, the compiler ensures, to the best of its ability, that the variable is used comsistently throughout the program.

## Kind of,memeber of
there are a few methods to let us know if some variables are or belong to somthing kind of class.

	BOOL answer = [classname isKindOfClass [NSObject class]];
	BOOL answer = [classname isMemberOfClass [NSObject class]];
	BOOL answer = [classname respondsToSelector: @selector (alloc)];
	
>Remeber that isMemberOfClass test or direct membership in a class,whereas isKindOfClass checks for membership in the inheritance hierarchy.