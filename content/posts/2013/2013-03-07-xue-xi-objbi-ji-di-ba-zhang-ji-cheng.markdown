---
layout: post
title: "Learn the obj-c:Inheritance"
date: 2013-03-07 22:03:01
comments: true
tags: [objective-c]
---
Finally , i've done the 8th chapter of the book PROGRAMMING in Obj-C,and i have already written all the samples to test what i've leaned。

As always , i put the code on a project in [github](https://github.com/yyq/learn-obj-c).

When I am reading the book on kindle, I highlight something when I think  they are very important。 

Herer, I'll show you the sentences I marked.
<!--more-->

1. >Instance variables declared or **synthesized** in the implementation section are **private** variables and are not directly accessible by subclass.

2. >Note that the side method does not have direct access to the Rectangle's width instance variable; it's private and therefore not accessible by the Square class. However,the **getter method is inherited** from the parent class and can be used to access the value of the width.

		@implementation Square:Rectangle
		-(void) setSide:(int) s{
		.....
		}
		-(int) side
		{
			return self.width.
		}

3. the _@class_ Directive
   
   When you are writing the ClassA.h，but you want to use the ClassB.Just type the _@class ClassB_ instead of the traditional import"class" something in other languages. But when you are writing the ClassA.m ,you want to use the methods of the ClassB something, you must use the _import"ClassB.h"_.
   
   But WHY?  here is the original words in the book:
   
   >Using the _@class_ directive is more efficient because the compiler doesn't need to import and therefore process the entire XYPoing.h file(even though it is quite small);it just needs to know that XYPoint is the name of a class.If you needed to reference one of the XYPoint class methods(say in the implementation section),the _@class_ directive would not suffice because the compiler would need more information;it would need to know how many arguments the method takes,what their types are,and what the method's return type is.
   
1.  I think here are things talking about **memory copy or reference copy?**,kindle position 3931/11270
    >That is,by default,the action of a synthesized setter is to simply copy the object pointer, not the object itself.
    >For this reason,the safest way to write a getter that returns an object is to actually make a copy of the object and to return that copy.
    
1. Here are three reasons to create a subclass:
    > 1. You want to extend the functionality of a class,perhaps by adding some new methods and/or instance variables.
    > 2. You want to make a specialized version of a class(e.g. a particular type of graphic object)
    > 3. You need to change the default behavior of a class by overriding one or more of its methods