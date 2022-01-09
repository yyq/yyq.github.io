---
layout: post
title: "xcode调试，在模拟器运行正常，在设备运行失败"
date: 2013-01-06 20:56:01
comments: true
tags: [ios,xcode,simulator,device]
---
### Xcode will run app on simulator but not on device

在把学校项目的季度报告提交给软件所之后，今儿终于有空来弄一下我的ios编程了。今儿刚刚学会了真机调试，然后就不断的run啊，之前只会在模拟器调试实在略感憋屈啊。

不出一会，立马就有报错了，立马就上网查查，发现google真是好，搜索的时候用英文，google立马就能精确的给我定位，前三个链接中都来自stackoverflow，进去浏览一番，果然就能解决我的问题啊。
<!--more-->
**Here is the situation:**
在Provisioning Portal里面新生成了一个profile，用的新的AppIDs和identifier。然后再xcode里面改Target的identifier和证书。

再在设备上运行，之后问题就出现了：could not lunch xxx，然后在xxx目录没有发现xxx文件。

之后我试过在模拟器运行，一切正常。

google之。

解决方案来自：<http://stackoverflow.com/questions/9987192/xcode-will-run-app-on-simulator-but-not-on-device>

步骤如下：
 
 1. Disconnect your device
 1. Delete the app from your device
 1. Quit Xcode (Don't just simply close the window, quit it)**彻底关闭xcode**
 1. Delete derived data folder rm -fr ~/Library/Developer/Xcode/DerivedData (console)**恩，这里是最重要的**
 1. Start Xcode,connect device & run the project