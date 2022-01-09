---
title: 'iOS逆向工程学习日记3--实践'
author: young
layout: post
dsq_thread_id:
  - 3922267620
categories:
  - 计算机
  - security
date: 2015-07-11
---
## Results：

学会了使用反汇编工具hopper，找了一些个程序，试了试，有的二进制文件还是比较容易分析的，有的比较难，我暂时还不会。我搞定的成果如下：

Mac OS X，迅雷，VIP离线下载权限

Mac OS X，iExplorer，免注册

iOS，搜狐视频，跳过所有广告，解除美剧下载限制

update:

iOS PPTV，乐视视频，可以观看VIP影片

&nbsp;

## Key：

改掉几个函数的返回值就好了，类似isRegister这个函数或者isVIP这样的函数，看看逻辑，分析分析代码流程，再把关键位置的汇编代码改掉，即可。

x86汇编指令，mov eax, 0x1，把数字1放到eax寄存器，通常函数的返回值存在eax中，然后用ret指令

arm汇编指令，movs r0, #0x1，寄存器用r0

&nbsp;

## Thinking：

想要改变一个iOS软件的行为，可以从三个层次去着手：

1. 改掉汇编代码，重新生成新的可执行文件，替换

2. hook一些函数，例如Theos的常用手法

3. 拦截并且修改网络请求

&nbsp;