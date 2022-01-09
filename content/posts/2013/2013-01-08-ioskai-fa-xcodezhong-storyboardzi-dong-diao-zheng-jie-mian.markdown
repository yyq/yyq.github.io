---
layout: post
title: "Ios开发Xcode中storyboard自动调整界面，autolayout等"
date: 2013-01-08 11:39:01
comments: true
tags: [ios,xcode,storyboard,autolayout]
---

这个应该是很大一堆条条框框的使用策略。今儿只遇到了一点点问题，就记录一点点问题。

另外，值得参考的网址：<http://www.raywenderlich.com/20881/beginning-auto-layout-part-1-of-2>

笔记：
<!--more-->

刚刚接触xcode不久，在使用storyboard时遇到了问题如下：因为iphone4s和iphone5的手机高度不一样，所以，我想某一个按钮在屏幕的位置，需要锁定从该按钮到屏幕顶端的位置。但是，在我把一些个按钮拖到storyboard中的时候，系统自动给锁定了从该按钮到屏幕底端的位置。

从代码的角度来说，这要是个nib文件啥的，别人八成就用代码设定这个参数就ok了。作为新手我再去了解nib界面的代码设定，参数属性啥的，我都不大乐意了，所以就从storyboard里面找这个东西，在哪里可以精确的锁定某一个button到某一边边框的距离。找了很久，终于找到了。**其实就在storyboard界面的右下方。**
![x](2013-01-11-10.28.32.png)


尽管找到了这个东西，但是我觉得，等有空，学校的事情不是那么多，我很有必要学会从利用nib文件创建每一个按钮和属性等。据说高手都不用storyboard的，用nib来设定界面很干净利落。这如今都ios sdk 6了，不知道哪里还有比较好的介绍nib的网站啊。
