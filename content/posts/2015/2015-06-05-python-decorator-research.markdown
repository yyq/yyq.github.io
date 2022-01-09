---
title: Python装饰器有趣实例探究
author: sophie
layout: post
dsq_thread_id:
  - 4178795147
categories:
  - 计算机
  - Python
tags:
  - python
  - 装饰器
date: 2015-06-05
---
廖老师的教程实在太高深，没弄懂，<a href="http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/001386819879946007bbf6ad052463ab18034f0254bf355000?t=1433431549858#0" target="_blank">点击打开链接</a>

``` python
def deco_functionNeedDoc(func):
    if func.__doc__ == None :
        print func, "has no __doc__, it's a bad habit."
    else:
        print func, ':', func.__doc__, '.'
    return func
@deco_functionNeedDoc
def f():
    print 'f() Do something'
@deco_functionNeedDoc
def g():
    'I have a __doc__'
    print 'g() Do something'
f()
g()
print f
print g</pre>
```

这段代码打印结果如下：

``` python
<function f at 0x0238F930> has no**doc**, it&#8217;s a bad habit.

<function g at 0x0238F8B0> : I have a **doc** .f() Do somethingg() Do something

<function f at 0x0238F930>

<function g at 0x0238F8B0>
```
当时我就晕菜了，想了很久，原来在@装饰器函数的时候就会调用装饰器，装饰器函数return func，而func就是传进去的参数f。这个时候把代码改改。

<!--more-->

``` python
def deco_functionNeedDoc(func):
    if func.__doc__ == None :
        print func, "has no __doc__, it's a bad habit."
    else:
        print func, ':', func.__doc__, '.'
    def func_1(*args, **kw):
        print func.__name__,' : this is func_1'
    return func_1
@deco_functionNeedDoc
def f():
    print 'f() Do something'
@deco_functionNeedDoc
def g():
    'I have a __doc__'
    print 'g() Do something'
f()
g()
print f
print g</pre>
```
此时的打印结果：

``` python
<function f at 0x023CF930> has no \_\_doc\_\_, it&#8217;s a bad habit.  
<function g at 0x023CF8B0> : I have a \_\_doc\_\_ .  
f  : this is func_1  
g  : this is func_1  
func_1  
func_1
```
问题至此，应该很明了了~~只是，装饰器拿来干嘛呢？？应用在什么情况下呢？？待探索

最后再贴一个超级大团圆，太有意思了，这个不解释了，困了，理解了这个，估摸装饰器的基本原理就透彻了。

``` python
def deco_functionNeedDoc(func1):
    if func1.__name__ == "yyy" :
        print func1, "the func1 == yyy"
    else:
        print func1, ':', func1.__doc__, '.'

    def xxx() : # y + f + h + x
        func1() # y + f + h + f + h
        print "plus xxx"
        return func1() # y + f + h + f + h

    return xxx

def deco_functionNeedDocxxx(func2):
    if func2.__name__ == "hhh" :
        print func2, "the func2 == hhh"
    else:
        print func2, ':', func2.__doc__, '.'

    def yyy(): # y + f + h + f + h
        print "plus yyy"
        func2()  # f + h
        return func2() # f + h

    return yyy

def deco_functionNeedDochhh(func3):
    if func3.__name__ == "f" :
        print func3, "the func3 == f"
    else:
        print func3, ':', func3.__doc__, '.'

    def hhh() : # = f + h
        func3()  # = f
        print "plus hhh"
        return "Hello Wrold"
    return hhh


@deco_functionNeedDoc
@deco_functionNeedDocxxx
@deco_functionNeedDochhh
def f():
    print 'print original fff'

f()
print "------------"
print f
print "------------"
print f()</pre>

```
结果如下：

``` python
<function f at 0x109a2b938> the func3 == f  
<function hhh at 0x109a2b9b0> the func2 == hhh  
<function yyy at 0x109a2ba28> the func1 == yyy  
plus yyy  
print original fff  
plus hhh  
print original fff  
plus hhh  
plus xxx  
plus yyy  
print original fff  
plus hhh  
print original fff  
plus hhh  
&#8212;&#8212;&#8212;&#8212;  
<function xxx at 0x109a2baa0>  
&#8212;&#8212;&#8212;&#8212;  
plus yyy  
print original fff  
plus hhh  
print original fff  
plus hhh  
plus xxx  
plus yyy  
print original fff  
plus hhh  
print original fff  
plus hhh  
Hello Wrold
```