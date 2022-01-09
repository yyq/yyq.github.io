---
title: 'iOS逆向-学习日记2-Mac OS X上的工具'
author: young
layout: post
dsq_thread_id:
  - 3909859315
categories:
  - 计算机
  - security
tags:
  - iOS
  - security
  - theos
  - tools
  - tweak
date: 2015-07-07
---
这一次，主要是按照别人的教程，把Mac上逆向工程有关的工具挨个试了个遍。先说几个重点的：Charles，Reveal，TheOS，IDA，Hopper。其中Charles，Reveal，IDA和Hopper都是收费的，且都有demo版本可以下载适用，TheOS是免费。

## 简介四个收费工具

Charles用于分析网络流量，模拟器，mac os x，device都可以搞定各种get post connect，header cookie分门别类一目了然。价格不贵。

Reveal主攻iOS UI，模拟器和device都行，对于越狱的设备可在cydia里装个插件，然后查看并且临时修改UI，利器。

IDA和Hopper，算作一类了，主要功能是二进制文件到汇编代码，和进一步的汇编代码到C语言代码。我亲自使用过后，着实感觉到工具的强大，十分的强大。再加上工具里各种代码块和控制流的显示，就算我只是一个逆向工程的外行，也知道这样的信息会带来多么恐怖的分析成果。IDA是各个平台的产品都有，价格上千美刀了。Hopper是只有Mac平台上的，价格几百块的样子。据学习过逆向工程的同事说，IDA还是很强大很流行的。

这四个工具的使用难度，Charles比较简单，Reveal略复杂一点。反汇编工具就超级复杂了。不过都有UI，自己可以一个个试着玩玩。

## 详细介绍Theos越狱开发工具

接下来说说花了我很多时间用来实践的Theos吧。一个跨平台的development kit，用于开发编译iOS软件的，without Xcode。越狱后的iOS所用的插件，Cydia上的那些工具，大都基于Theos开发的。

<!--more-->

主要参考的两个网页，<a href="http://security.ios-wiki.com/issue-3-6/" target="_blank">http://security.ios-wiki.com/issue-3-6/</a>和<a href="http://iphonedevwiki.net/index.php/Theos/Setup" target="_blank">http://iphonedevwiki.net/index.php/Theos/Setup</a>

第一个链接中是简单的中文教程，估计可能有个别描述比较老了不适用现在的环境，那就看第二个官网链接了，那个比较准确一点。

### 第一步，ready

我用的是mac电脑，装个Xcode，就会自带SDK了，然后command line tools for Xcode也下载好，电脑上就会有其他的小工具软件了。

### 第二步, $THEOS

定义一个环境变量$THEOS，这个关键的环境变量将一直会用到。所以写在.bash_profile里吧。

<pre class="lang:sh decode:true ">export THEOS=/opt/theos</pre>

为了之后省事，我把这一行也添加到.bash_profile了：

<pre class="lang:sh decode:true ">export THEOS_DEVICE_IP=192.168.2.7 THEOS_DEVICE_PORT=22</pre>

### 第三步，下载Theos和ldid

<pre class="lang:sh decode:true ">git clone git://github.com/DHowett/theos.git $THEOS
git clone git://git.saurik.com/ldid.git
cd ldid
git submodule update --init
./make.sh
cp -f ./ldid $THEOS/bin/ldid</pre>

然后从设备上拷贝/Library/Frameworks/CydiaSubstrate.framework/CydiaSubstrate文件到电脑的`$THEOS/lib文件夹，命名为libsubstrate.dylib`

### 第四步，安装dpkg

<pre class="lang:sh decode:true ">brew install dpkg</pre>

### 第五步，创建一个jailbroken app

<pre class="lang:sh decode:true ">$THEOS/bin/nic.pl</pre>

然后就问你几个选项，随意填几个字符串就行

<pre class="lang:sh decode:true ">$ $THEOS/bin/nic.pl
NIC 2.0 - New Instance Creator
------------------------------
  [1.] iphone/application
  [2.] iphone/library
  [3.] iphone/preference_bundle
  [4.] iphone/tool
  [5.] iphone/tweak
Choose a Template (required): 1
Project Name (required): BlogDemo
Package Name [com.yourcompany.blogdemo]:
Author/Maintainer Name [YangYanQing]:
Instantiating iphone/application in blogdemo/...
Done.</pre>

然后他会创建如下的一个目录：

<pre class="lang:sh decode:true ">~/Desktop/blogdemo
$ tree .
.
├── BlogDemoApplication.mm
├── Makefile
├── Resources
│   └── Info.plist
├── RootViewController.h
├── RootViewController.mm
├── control
├── main.m
└── theos -&gt; /opt/theos

2 directories, 7 files</pre>

  1. control: 包含applicaton/tweak的信息，当你从Cydia安装时，你可以看到这些信息，包括名字，作者，版本，等等。
  2. main.m，这个不用多说。
  3. [applicationName]Application.mm:appdelegate文件。
  4. Makefile：包含必要的编译命令
  5. Resources：包含info.plist文件等
  6. RootViewController.h/mm

以上六行字，摘抄自：[link][1]

### 第六步，设置环境和编译

<pre class="lang:sh decode:true">export SDKVERSION=8.4</pre>

设置一下自己的SDK版本号，我的版本号是8.4，在目录

/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs找到的。

然后

<pre class="lang:sh decode:true">make
make package
make install</pre>

会出现报错，什么internal-install error来着，解决方法是把$THEOS/makefiles/package/deb.mk文件中的

<pre class="lang:sh decode:true ">$(ECHO_NOTHING)COPYFILE_DISABLE=1 $(FAKEROOT) -r dpkg-deb -b "$(THEOS_STAGING_DIR)" "$(_THEOS_DEB_PACKAGE_FILENAME)" $(STDERR_NULL_REDIRECT)$(ECHO_END)
</pre>

替换为：

<pre class="lang:sh decode:true ">$(ECHO_NOTHING)COPYFILE_DISABLE=1 $(FAKEROOT) -r dpkg-deb -Zgzip -b "$(THEOS_STAGING_DIR)" "$(_THEOS_DEB_PACKAGE_FILENAME)" $(STDERR_NULL_REDIRECT)$(ECHO_END)
</pre>

至于为什么这么替换，我也不明白，参考的教程。

## 我的第一个Tweak程序

教程链接：<http://security.ios-wiki.com/issue-3-6/>

这个demo hook了springboard的init方法，多添加了一个UIAlertView。学会此方法，可以应用于任何class的任何method打补丁了。

### 第一步，ready

/opt/theos/lib目录下的libsubstrate.dylib文件不可少

### 第二步，头文件

copy iPhone的一些头文件，万一编译的时候有点头文件错误，估计就是这一步的文件有点问题了。教程中的参考的头文件获取方法是：

<pre class="lang:sh decode:true ">git clone https://github.com/rpetrich/iphoneheaders.git
mv iphoneheaders/* theos/include/</pre>

然后把OSX library中IOSurfaceAPI.h拷贝到theos/include/IOSurface目录下，在Xcode.app下面搜索这个文件好了，估计不同版本的mac系统，这文件位置略有不同，我的此文件路径是在：

<pre class="lang:sh decode:true ">/Applications/Xcode.app
$ find . -name IOSurfaceAPI.h
./Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.10.sdk/System/Library/Frameworks/IOSurface.framework/Versions/A/Headers/IOSurfaceAPI.h
./Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.9.sdk/System/Library/Frameworks/IOSurface.framework/Versions/A/Headers/IOSurfaceAPI.h</pre>

两个文件中随意copy一个过来好了，然后在这个头文件中注释掉IOSurfaceCreateXPCObject 和IOSurfaceLookupFromXPCObject。

### 第三步，添加工程代码，修改make file

用$THEOS/bin/nic.pl创建工程，工程类型选5.iphone/tweak.

在生成的目录中会有文件Tweak.xm，添加自己的代码进去，如下：

<pre class="lang:objc decode:true">#import &lt;SpringBoard/SpringBoard.h&gt;

%hook SpringBoard

-(void)applicationDidFinishLaunching:(id)application {
%orig;

UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Welcome" 
message:@"Welcome to your iOS Device Young!" 
delegate:nil 
cancelButtonTitle:@"I'm learning iOS security" otherButtonTitles:nil];

[alert show];
[alert release];

}

%end</pre>

在makefile文件中添加我们用到了的framework，在demo\_FILES=Tweak.xm那一行的下面添加demo\_FRAMEWORKS = UIKit

### 第四步，make

三个命令喽，make; make package; make install.

make命令的时候有报错信息，大概是说iOS 8.0  UISearchDisplayController 改成UISearchController了，有三个地方都报这个错，根据报错信息中提示的三个报错的位置，把UISearchDisplayController直接改成UISearchController就能顺利make出东西了。

产品出来了，就是在设备每次重启的时候，springboard会执行init函数，也就会弹出我们写的alert对话框了。

&nbsp;

 [1]: http://security.ios-wiki.com/issue-3-6/