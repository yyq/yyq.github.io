---
title: Python @property装饰器
author: sophie
layout: post
dsq_thread_id:
  - 3964940401
categories:
  - 计算机
  - Python
tags:
  - python
  - 装饰器
date: 2015-06-14
---
廖老师的博客链接如下，一开始没看懂，搜罗了一大堆，有点感觉了

<a href="http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/001386820062641f3bcc60a4b164f8d91df476445697b9e000" target="_blank">点击打开链接</a>

其实@property装饰器就是把class的方法变成属性，见下面这个class，它有两个私有属性。

通过第一个@property和第二个@score.setter，我们可以像访问属性一样来调用类里面的方法，例如：

s=Student(&#8220;David&#8221;,99)

s.score = 100

至于用这种装饰器的原因，我想，就是为了简洁吧，直接用属性赋值的方式，执行了方法。Python 肯定是个懒人发明的。

<pre class="lang:python decode:true ">class Student(object):

    def __init__(self,name,score):
        self.__name = name
        self.__score = score

    @property
    def score(self):
        return self.__score

    @score.setter
    def score(self,value):
        if not isinstance(value,int):
            raise ValueError("invalid score!!!")
        elif value &lt; 0 or value &gt; 100:
            raise ValueError("score must be between [0,100]!!!")
        else:
            self.__score = value

    @property
    def name(self):
        return self.__name</pre>

&nbsp;