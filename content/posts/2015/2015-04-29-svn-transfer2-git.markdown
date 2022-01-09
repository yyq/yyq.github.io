---
title: svn转git简单实践
author: young
layout: post
dsq_thread_id:
  - 3721193596
categories:
  - 计算机
  - Infrastructure
date: 2015-04-29
---

# 前言

涉及公司内部的名词，就用abc来代替好了。

我负责的C工程，代码之前一直是用SVN来管理代码版本。为了响应党的号召，改Git，公司还花钱买了github企业版，这年头花钱买github而不是自己搭gitlab，良心企业啊。上网搜搜教程，试了一试，便成功了。关键操作记录在此了。

# 需求一：svn转git
``` shell
git svn init http://svn_address
git svn fetch</pre>

or using git svn fetch -r 1:HEAD, 从revision 1开始fetch
```
去趟厕所，再喝杯咖啡，看stdout的信息可以看到正在从svn server上一个revision一个revision的内容往下拖，C工程（java代码为主，三万多行代码，引用的jar包10来个，其他的引用的jar包都用maven配置好了，所以不用把jar包存在工程里，SVN 的revision 700个左右），耗时大概10分钟，便转换完毕了。

然后本地的git project已经建立好了。往远端push就好了。

``` shell
git remote add origin git@git_address
git push -u origin master</pre>
```
# 需求二：将svn的改动同步到git

由于IDE使用的不熟悉，不是立马用IDE git clone过来工程就能直接用，没空来折腾。所以最近几天仍然只用svn来commit，但是想要更新svn project上的内容到git。

怎么搞。上网一搜，挺好搞。

``` shell
git svn fetch
git svn rebase

git svn fetch
git push -u origin master</pre>
```
不要问我为什么用这几句话，我也就是临时上网搜一下拿来用了。需要知道这命令干什么用的话，用help命令好了。

# 产出

之后用git来管理C工程源代码了，托github企业版的福，可以更加方便的在网页上show给别人看了，哪天我改了多少代码，代码量啦，多少个新的feature啦，release note啦，issue track啦，都可以统统的放在一个网站上搞定了。

还有，github pages的功能，给C工程建了一个宣传网页，瞬间高大上了有没有。

by the way, 我参考的网页

<http://www.blogjava.net/lishunli/archive/2012/01/15/368562.html>

