---
Title: Burpsuite实践：iOS app刷点击量
author: Young
date: 2017-03-31
tags: [burpsuite, iOS, app, certificate, iOS10]
Status: public
---
### 需求
最近有个做媒体的朋友，她们发表在某app上的文章，有点击量的KPI，问我能不能帮刷一刷阅读量。最近学了burpsuite，正好练习练习。

### 分析
稍微一观察，那app还比较简陋，同一个手机里，反复的去点击文章，再退出，就会增加点击量。

那目标就比较简单了，观察在手机上点击阅读某篇文章的时候，会有什么样的网络请求，再重复就行了。之前想象的不断切换IP的难度也就没有了。

### 实践

#### 打开Burpsuite监听
首先在BurpSuite里找到Proxy,把Intercept关掉，这里不需要拦截请求。在option里添加一个新的Listeners，端口号填好，Bind to address选择All interfaces.其他默认。

#### 设置手机wifi
打开手机WIFI的设置，代理选手动，填上自己电脑的IP地址和刚刚设定的端口号。

#### 安装CA证书
**对于https的请求，这里是最重要的一步，否则你的app里只会显示各种没有网络无法连接等错误**

打开**http://burp**,按照网页右上角的提示安装**CA Certificate**

**这还没完！！更重要的一个步骤：如果你的iOS系统是比较新的，比如我手机是iOS10.3，那么你还要到手机的设置=>通用=>关于本机=>证书信任设置里，打开对PortSwigger CA证书的完全信任**

这个步骤花了我足足两个小时有没有，找便了互联网都没有看到有人说要设置手机打开证书完全信任，然后认认真真看官网页面介绍如何安装CA证书，[support.portswigger.net](https://support.portswigger.net/customer/portal/articles/1841109-Mobile%20Set-up_iOS%20Device%20-%20Installing%20CA%20Certificate.html)。终于，在这个页面正文的后面，很不起眼的一个小提示里说到了这个point，finally my app works fine! WTF! 那里还说了他的这个介绍文档是基于iOS8.1.2的iPad mini.其他版本的iOS可能需要这个操作。

目测是apple为了提升iOS的安全性，就算被意外安装了证书，也不会默认完全信任。要用app通过代理顺利的请求https的内容的话，需要设置手机完全信任这个代理的CA Certificate才行。

#### 重放网络请求
这个简单了，手机app里随意点一点，可以发现每次点击一个新的文章，都会有一个post请求和一个get请求。先拿post请求实验，用**send to Intruder**, 然后clean掉所有潜在的payload positions,在payloads选项里，选择null payloads,然后设置一下循环次数，先来个5000吧。

我滴乖乖呀，一分钟左右吧，该文章阅读量明显的从2.8万上涨到18.4万。可是我明明只有5000次请求。问了问朋友，这大概是他们统计数据的时候有作假。但是，anyway，我这5000次请求可是货真价实的给该文章增加了不少阅读量。

### lessons learned
* 实战和看视频学习使用软件是两回事儿，有的细节上的设置和设计，差一点点就是功亏一篑谬以千里
* 有不懂的知识，google里前5个链接没找到答案的话，那么认真阅读官方文档，哪怕细小的一个提示框都不可以错过