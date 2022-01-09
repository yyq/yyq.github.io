---
title: 论在程序中写log的重要性
author: young
layout: post
dsq_thread_id:
  - 3568196742
categories:
  - 计算机
  - Infrastructure
tags:
  - java
  - log
  - log4j
format: aside
date: 2015-03-05
---
别人开发的工具，没几个人用，然后别人比较忙不管这摊子了，然后没人弄，组织上说，需要我来维护。正好碰上某leader推广这工具，这下完了，一堆bug，必须摆平。基本上一天一个到两个的速度，两三个礼拜的时间，紧急的bug都搞定了。之后心中最大最大的感受就是关于在代码中写Log。

从前自己写写小程序自己怎么折腾都没所谓，错了大不了就调试呗，从来没有写log的习惯，偶尔就在控制台输出输出就完事儿。这一轮忙碌的修bug事件，让我彻底改变了写log存log读log的看法。

我感受到的写log的好处有三点：

<!--more-->

  * easy to troubleshoot：快速定位错误 
      * 比如我软件的某个弹窗出来说哪哪错了，需要用户自己手动改一改系统哪里的设置就好；
      * 或者在标准输出流输出，运行这软件的时候从命令行启动，然后就可以在命令行console或者terminal看到错误信息；
      * 最好是打印出来call stack，就能直接定位到某一行代码了。
  * no need to reproduce：无须错误重现 
      * 当用户知道在什么条件下会产生bug时，用户可以重现这bug，那很好说，我自己也照着重现一遍就是。It&#8217;s OK when user **can** reproduce the bug. So, the question is: what if not?
      * 这个时候就需要把log写在某个文件中了，主要是方便事后来看。对于这两类bug很有帮助：一类是对于难以重现的在极端特定的条件下才会出现的错误，另外一类就是人家的机器一直就比较忙，没有空借给你来调试或者仔细观察。这两种情况下，请他把log文件发送给我就好了。
      * 关于此，还有一个改进我没有做的。就是平常自己用Mac电脑或者iOS程序时，碰到了崩溃，会弹一个框，问用户是不是把bug信息发送给开发者。我估计这个做法是在程序很多地方都捕捉异常，当异常发生的时候，dump出所有相关信息，然后建议用户email给开发者。这样一来的好处就是，开发者就可以收到很多很多bug了，并且能根据bug list来做一个分析，bug的出现频率啦，bug的严重程度啦，bug是否在某种特定机型出现啦等等，这些信息对于修bug来说是极佳的。
  * performance issue：性能问题 
      * 前面个两点都做好了之后，以为差不多。直到某一天，有leader以及他的好几个teammember跟我反映说，慢。这一下，我头就大了。没有抛出异常或者call stack，程序也没有终止运行，仅仅是运行时间明显比原来慢。我只能跟人家回复说，我先看看log。
      * 拿来log文件，庆幸自己在写log的时候，所有的事件都打了时间戳。拷贝一份运行很慢的log，再来一份之前运行过的不慢的log。开始慢慢看，最终找到了原因，因为某设置参数，导致多线程被关闭了。Issue done.

最后，我分享一下我写log用到的知识，这程序的java的，上网查了查和问了问我mentor，建议用log4j来写log，方便快捷易上手。

从官网找到log4j-core-2.1，log4j-api-2.1，添加到工程的依赖。我这个是maven工程，搜索一下log4j就好。然后写log4j的配置文件，当然你也可以不用写一个log4j的配置文件，用代码配置也是OK，上网搜搜应该到处都有教程。注意一下log4j 2.1版本的配置文件的参数，在resources文件夹中添加一个log4j2.xml文件，那么程序启动时，它会自动找到这个文件作为配置文件。

<pre class="lang:xhtml decode:true " title="log4j2.xml">&lt;?xml version="1.0" encoding="UTF-8" ?&gt;
&lt;configuration&gt;
    &lt;appenders&gt;
        &lt;Console name="Console" target="SYSTEM_OUT"&gt;
            &lt;PatternLayout pattern="%d{HH:mm:ss.SSS} - %msg [%t] %-5level %logger{36}%n" /&gt;
        &lt;/Console&gt;
        &lt;RollingFile name="FastRollingFile" fileName="Z:/CIMResults/logs/cim.log" filePattern="Z:/CIMResults/logs/$${date:yyyy-MM-dd}/server-%d{yyyy-dd-MM-HH}-%i.log"&gt;
            &lt;PatternLayout&gt;
                &lt;pattern&gt;%d %p %c{1.} [%t] %m%n&lt;/pattern&gt;
            &lt;/PatternLayout&gt;
            &lt;Policies&gt;
                &lt;TimeBasedTriggeringPolicy interval="1" modulate="true" /&gt;
                &lt;SizeBasedTriggeringPolicy size="20 MB" /&gt;
            &lt;/Policies&gt;
        &lt;/RollingFile&gt;
    &lt;/appenders&gt;
    &lt;loggers&gt;
        &lt;root level="DEBUG"&gt;
            &lt;appender-ref ref="Console" /&gt;
            &lt;appender-ref ref="FastRollingFile" /&gt;
        &lt;/root&gt;
    &lt;/loggers&gt;
&lt;/configuration&gt;</pre>

在代码中需要引入这个包

<pre class="lang:java decode:true ">import org.apache.logging.log4j.Logger;
import org.apache.logging.log4j.LogManager;</pre>

然后初始化

<pre class="lang:java decode:true ">private static final Logger logger = LogManager.getLogger(A.class.getName());</pre>

然后使用

<pre class="theme:tomorrow-night lang:java decode:true ">//normal record using log
logger.info("This is info message");
//If in some exception block, we can use error
logger.error( e.getMessage() );</pre>

之后，开始运行我的程序。

然后根据配置文件，我就能在Z盘的CIMResults文件夹下的logs文件夹下找到自己的所有的log，按照每天每小时安放好了的，并且每当log超过20MB自动拆分文件。

That&#8217;s it.

&nbsp;