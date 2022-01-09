---
layout: post
title: "eclispe+qemu+gdb调试linux kernel全过程"
date: 2013-03-06 22:52:01
comments: true
tags: [debug,eclipse,gdb,kernel,linux,qemu]
---
# 单步调试kernel说明
恩，这个文档的目标是单步调试内核，从每一个工具软件的版本号到每一个命令，都有一个说明

<!--more-->

## ubuntu1204,32位
<http://www.ubuntu.org.cn/download/desktop>

用vmware虚拟机安装该系统。

用64位系统时，gdb有bug。报错信息为：xxx太长。所以建议用32位系统

## 编译kernel 3.5.4
下载内核的地址,北京交通大学的映像地址：<http://mirror.bjtu.edu.cn/kernel/linux/kernel/v3.x/>

我下载的内核源码版本号3.5.4

为防止系统有些组件版本号较低，考虑如下两个命令更新系统：
	
	sudo apt-get update
	sudo apt-get upgrade
	sudo apt-get install build-essential

编译步骤：进入kernel的根目录后，命令如下：

1. make menuconfig   
	//提示没有找到ncurses 安装一下:sudo apt-get install libncurses
	//贾迪提示：可以用libncurses* 
	//Kernel Hacking -> 找到Compile the kernel with debug info” and “Compile the kernel with frame pointers” 这两个选项，必须是选中的。其他，不用改设置，按照默认就行，直接按两下esc，选择保存并退出
1. make

## qemu 1.4.0 
<http://wiki.qemu.org/Download>
### 安装
按照官网的说明，编译，安装:<http://qemu.weilnetz.de/qemu-doc.html#compilation>

编译qemu报错

1. 缺glib,解决方法：sudo apt-get install libglib2.0-dev

1. 缺autoconf,解决方法：sudo apt-get install autoconf automake libtool

用较老版本qemu时，或者直接用ubuntu source中的qemu时，单步调试失败。
### 使用:验证编译的内核已经可以通过qemu运行起来
qemu-system-i386 -kernel (kernel根目录)/arch/x86/boot/bzImage -initrd /boot/initrd.img-3.5.0-25-generic

注意：kernel后的参数为自己编译的内核，initrd参数为系统自带的文件，不同系统可能版本号不同
开始运行后，它会提示在vnc 102.0.0.1:5900 启动了。

然后在ubuntu桌面界面下，按下键盘中windows键（mac按command键），输入remote，找到“Remmina Remote Desktop Client”软件，打开后，新建一个远程连接，注意 协议用VNC，server填写127.0.0.1:5900

![x](2013-03-07-1.png)

在进入系统之后，输入uname -a或者uname -r来检验该系统是不是正在运行自己编译内核，看到3.5.4即可。

![x](2013-03-07-2.png)

## JDK(运行eclipse需要的java环境)
<http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html>
版本号，7u15。一般情况下，采用最新的版本号即可。

1. 下载了文件：jdk-7u9-linux-i586.gz
1. 解压缩：

		tar -xzf jdk-7u15-linux-i586.gz

1. 移动解压缩后的文件夹到目标位置
	
		sudo mkdir /usr/lib/jvm;     
		mv jdk1.7.0_15/ /usr/lib/jvm

1. 设置符号链接：（注意更改为自己的版本号，install参数前面是两个短横）

	sudo update-alternatives –install /usr/bin/javac javac /usr/lib/jvm/jdk1.7.0_09/bin/javac 1
	
	sudo update-alternatives –install /usr/bin/java java /usr/lib/jvm/jdk1.7.0_09/bin/java 1
		
	sudo update-alternatives –install /usr/bin/jar jar /usr/lib/jvm/jdk1.7.0_09/bin/jar 1
		
	sudo update-alternatives –install /usr/bin/javadoc javadoc /usr/lib/jvm/jdk1.7.0_09/bin/javadoc 1


## eclipse JUNO
<http://www.eclipse.org/downloads/packages/eclipse-ide-cc-developers/junosr2>
下载来解压缩即可。


## eclipse插件：CDT
<http://download.eclipse.org/tools/cdt/builds/>

我下载的版本号：cdt-master-7.0.1-I201009241320

下载到的zip压缩包。不用解压缩，安装方式：打开eclipse，

Help按钮 -> Install New Software -> Add按钮
在弹出的对话框中，Name随便填一个就行，Location，点右侧的Archive按钮，选择刚下载的zip文件。
安装即可。

## qemu+eclipse+gdb调试kernel

1. 启动qemu，命令如下：

   qemu-system-i386 -s -S -kernel ~/Desktop/linux-3.5.4/arch/x86/boot/bzImage -initrd /boot/initrd.img-3.5.0-25-generic

   -s 为默认远程调试，端口号1234

   -S 为启动调试时，停止，等待gdb

   -kernel 为自己编译出的bzimage，一般放在内核根目录的arch/x86/boot/

   -initrd 后面为自己ubuntu的某个镜像。


1. Window ->Preferences -> General -> Workspace，将“build automatically”去掉
![x](2013-03-06-1.png)

2. Window -> Preferences -> c/c++ -> Indexer中，将Enable indexer取消
![x](2013-03-06-2.png)

3. File->New->Project...->c/c++ -> C project

	Location处：选择自己的内核根目录文件夹

	工程类型选择Makefile Project,EmptyProject

	工具链选择，LinuxGCC
	
   ![x](2013-03-06-3.png)

4. 在Project Explorer中，右键自己的工程，选择Debug As -> Debug Configurations,在弹出的对话框中双击“GDB Hardware Debugging”,将会让你设置调试参数等。随意写一个Name，这个Name应该是这个调试配置文件的名字。
    ![x](2013-03-06-4.png)

5. 配置调试参数：在**Main**标签中，c/c++Application的框里，选择自己编译出来的vmlinux文件，该文件位置应该就在源码的根目录下。选中Disable auto build.
    ![x](2013-03-06-5.png)
 
    
   在**Debugger**标签中，GDB Command处填gdb，勾选use remote target，port number处填1234，因为qemu模拟器默认的远程端口就是1234.
   ![x](2013-03-06-6.png)


   在**Startup**标签中，去掉三个勾。
   ![x](2013-03-06-7.png)

   点击**Debug**，开始调试。
  
6. 验证调试
   我在init/main.c的第486行设置一个断点。
   
   Window -> Show View -> Expressions，打开后，输入我要观测的变量名字，early_boot_irqs_disabled，等运行到该断点时，看到其值从false变为true，验证了基本调试功能正常。
   ![x](2013-03-06-8.png)
   
   
   
