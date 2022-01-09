---
title: ClearCase2Git，git-cc的使用
author: young
layout: post
dsq_thread_id:
  - 3409777391
categories:
  - 计算机
  - Infrastructure
tags:
  - clearcase
  - cleartool
  - git
  - git-cc
format: aside
date: 2015-01-11
---
# 简介

<p class="">
  据传，公司若干年的的代码版本管理工具一直都是ClearCase,一个完整的view都若干个GB。很庞大。有老大提出新想法，转Git。于是我做了一些个简单的小的实验，用的是开源工具git-cc,原作者的代码链接<a title="https://github.com/charleso/git-cc" href="https://github.com/charleso/git-cc" target="_blank">https://github.com/charleso/git-cc</a>，我对readme做了个汉化以增加理解和略微修改代码适用于自己的工作，链接：<a title="https://github.com/yyq/git-cc" href="https://github.com/yyq/git-cc" target="_blank">https://github.com/yyq/git-cc</a>。
</p>

# 基本环境

一个linuxVM，有cleartool，git，python。其中较为棘手的应该是在一个linux机器上安装clearcase的一系列工具了，cleartool是命令行版本的CC。总之安装Clearcase不是一件简单的事情，我是从公司要了个自带clearcase的RHEL的模板就开始了。

# 了解git-cc

关于git，看完过PRO GIT那本书(<a href="https://git-scm.com/book/en/v2" target="_blank">it&#8217;s free here</a>)，有些认识。对clearcase，不熟悉，问了同事也告诉我只能通过man来快速了解cleartool的各个命令。好像没有别的办法来速成了，git-cc这工具是网上看着免费的比较流行的cc2git的工具了。拿来直接用，报错一堆堆，搞不定，没接触过python，然后把codecademy上的python教程学完，之后再看git-cc源代码，能理解一些了。

这个工具做的主要的事情就是：下载一个snapshot view，新建一个空的git repository，cleartool lsh列出view的历史活动信息，处理好新建文件和checkin代码以及各个文件的版本号信息，然后按照从开始到现在的时间顺序，做若干次把cc中某个文件的某个版本copy到git中，然后git commit。然后结束了。

<!--more-->

# 具体过程

先写conspecs，写好cc的下载规则，也就是你需要转换cc上哪个branch上的哪些个文件夹到git。

<pre class="lang:sh decode:true  ">cleartool mkview -sna -tag view_name local_view_folder
cleartool setcs -force conspecs_file</pre>

然后等着下载完毕。

之后在git那个空repository目录下，执行

<pre class="lang:sh decode:true ">gitcc init the_path_to_a_folder_in_the_view
gitcc rebase</pre>

然后就开始等待它慢慢操作直到结束。

gitcc rebase中的最重要的cleartool命令之一是：

<pre class="lang:sh decode:true">cleartool get -to file file_with_version</pre>

file是本地的某个文件路径，file\_with\_version是个相对路径，那个文件相对于view的路径，再加上版本号，我看到的cc中记录文件版本号就是从0开始的自然数。就是通过这样的操作，将cc中的某个文件某个版本copy到git工程中，然后git commit，根据cc中的历史信息，做若干次这样的cleartool get and git commit的操作，最后就能完成了。

# 那么问题来了

看看那个源码就能知道，一些脚本，不是很复杂，估计那也意味着不是很全面，还是有一些细节的地方，要么是我没设置对，要么是源码需要改进。

问题1：git commit信息中的author的邮箱，这个需要另外一一设置，readme文档中有说明，我没有设置好。等需要具体部署的时候得搞定。

问题2：gitcc无法对单个文件做操作，其中的很多命令都是操作文件夹的，鲁棒性不够，如果我的需求是将clearcase中的一个文件拿出来转换成一个git repository，那么gitcc暂时还不能搞定。需要改进。

问题3：合并多次cc中的checkin为git中一次commit，它的规则是，相邻的两次clearcase中的checkin操作，如果作者和评论都相同的话，就就git中算作一次commit。这个其实是不符合我的需求的。如果有一个文件，连续好几天每天都只对文件做出update，评论时也只写了update，并且整个cc的历史中这几天我没有修改任何其他的文件。那么这几天的若干次update一个文件被合并到一个commit了。而我希望的是，不同日期的checkin应该算作不同的commit的，那么我在源码中相关的判断条件处再加一条必须日期同一天就OK了。

问题4：在有的目录执行cleartool get -to 命令时，会有报错Exception:cleartool:Error:Operation &#8220;get cleartext&#8221; failed: unknown error in VOB。就算不运行gitcc，我直接敲cleartool的这个命令去得到某某文件的某某版本，也会报同同样的错误。所以应该不是glitch有bug，而大概是我的cleartool哪里设置有问题，有待我继续研究。

&nbsp;

&nbsp;

&nbsp;