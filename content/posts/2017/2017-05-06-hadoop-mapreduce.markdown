---
Title: 学习hadoop和mapreduce
author: Young
date: 2017-05-06
tags: [Hadoop, MapReduce, udacity, data]
Status: public
---
### 前言
4月底离职了，最近赋闲在家，趁着有空拓宽一下视野，学学大数据有关的基础知识吧。扫了一圈公开课的网站，[Udacity](https://cn.udacity.com/)网站有了专门的中文网站,不过教学视频还是英文的， 找了**Hadoop和MapReduce入门**这个课程，链接[https://cn.udacity.com/course/intro-to-hadoop-and-mapreduce--ud617](https://cn.udacity.com/course/intro-to-hadoop-and-mapreduce--ud617)，略有收获。

### Hadoop生态系统

#### 大数据简介

首先对大数据得有一个概念，我理所当然认为，数据量很大，比如有1TB的数据，那么就是大数据。稍加修饰一下，一台单独的电脑无法轻松处理的数据，则是big data。再看看别人的理解：

大数据概念中重要的的3个V, Volume, Variety, Velocity.
 * Volume,特指数据量的大小，比如说1TB,1PB
 * Variety,指数据的类型格式什么的，数据会从很多个不同的来源过来，有很多不同的格式样子
 * Velocity,生成数据的速度,处理数据的速度, 数据的生产者和消费者都得速度快啊。

#### Hadoop来源

![hdfs-mapreduce](./2017-05-06/1-hdfs-mapreduce.png)

某工程师有一个还不会太会说话的儿子，他儿子给自己的玩具取了个名字，发音为Hadoop.

Hadoop工程的核心是两个部分，一个是存储数据，HDFS(Hadoop Distributed File System),另外一个部分是处理数据，MapReduce。

#### Hadoop生态系统

![hello](./2017-05-06/2-hadoop-ecosystem.png)

底层是HDFS来存储数据的，基于此，有一个默认的MR来处理数据。
其他的开源软件，有的是把数据导入进Hadoop集群的，有的是让Hadoop更加容易使用的。例如Hive和Pig等。

Hive,不用写麻烦的mapper和reducer的代码，写的和标准SQL比较像就行了。

Pig也是一种更容易的脚本

Impala，也是可以使用类似标准SQL的语法，直接访问HDFS里的数据，而不需要经过MapReduce。

sqoop，把传统数据库的内容导入到HDFS。Flume也类似。

HBase,实时数据库，基于HDFS的。

### 看图学习Hadoop并解决问题

#### HDFS图解
![hdfs0](./2017-05-06/3-hdfs-0-datablock.png)
一个体积较大的文件会被分块存储，例如按照64MB来分割，150MB的就会被分成3块

![hdfs0](./2017-05-06/3-hdfs-1-namenode.png)
集群中有一个namenode节点，会将这三块数据分别存储在不同的数据节点上

![hdfs0](./2017-05-06/3-hdfs-2-dataredundancy.png)
然后每一块数据都会复制两份到其他的数据节点上

namenode是个非常重要的节点了，一个namenode如果挂了，整个集群就废了。那么一般用什么方法来保障namenode的高可用呢，一个方法是备份，在集群里放入第二个namenode，备用。另外一个方法是，把namenode里的数据通过NFS的方式，存储在另外的存储机器里，namenode节点机器可以坏掉，不过数据没有丢失，可以拿来恢复。

#### MapReduce图解

我们通过下面这个具体的问题来学习MR.
![mr](./2017-05-06/4-mr-0-question.png)
我们有一个连锁店的销售数据，数据形式应该是很多本账单。然后我们想统计每个单独的店铺销售总量。

![mr](./2017-05-06/4-mr-1-data.png)
每一条销售数据的格式大概就是，日期，销售店铺（即城市名称），销售类别，销售价格

![mr](./2017-05-06/4-mr-2-mr.png)
按照MapReduce的思路来做这个事情，先找来三个人作为Mappers，分给他们一些账本，然后他们分别将自己账本里的数据，按照店铺分个类，一类一类分好，然后再找来reducers，他们每个人负责某些特定的店铺，收集Mapper手里的某店铺的所有数据，收集完毕之后，统计，给出结果。

![mr](./2017-05-06/4-mr-3.png)
上面的事例还有几个重要的点没有说明，1. 中间结果都用哈希表的方式来存储，比如某一条销售记录可以存储为，key是销售店铺名称，value是销售额 2. map之后，会有一个shuffle and sort的过程，会将数据按照key的字母顺序排序 3. Hadoop中默认设置reducer的数量是1.

### MapReduce设计模式

用图说话

![x](./2017-05-06/5-0.png)

有三种情况，是比较合适使用MR的。

#### 过滤模式

* 简单过滤
* 布隆过滤，不懂的请自行谷歌。我记得这是搜索引擎中用到的技术，大量数据的查重，效率很高
* 采样
* 随机采样
* 求值Top N

#### 概括模式

* 反向索引，也是搜索引擎中用到的技术，根据关键字来检索网页/文章/帖子
* 数值概括
 * 计数
 * 最小、最大
 * 第一，最后
 * 中位数，平均值，标准差

#### 结构模式

当你要把数据从RDBMS移植到Hadoop后，这种模式就有用了。在对数据集进行分析的时候，MapReduce的过程中，中间数据的格式和内容都可以自己重新定义，就可以免去你的RDBMS中对各个数据集的live join操作，这可是能省下不少时间。

前提条件：为了要使用这种方法，首先你数据库里的数据得是有外键连接的，第二数据库中的数据得是row-based，基于行的数据。


### 编程练习

在这个课程中，也有一些编程时间，做了如下几个练习来体验Hadoop和MapReduce

数据集是商店购物数据，体积50MB

 * 统计每一个类别的销售总额
 * 找出每一个店铺的最高单笔销售额
 * 统计一共有多少笔销售和所有的销售总额

某个web server的日志，体积200MB

 * 该网站所有文件被访问的次数
 * 统计哪个IP访问该论坛次数最多
 * 找出该网站被访问次数最多的文件

某个论坛的真实数据，体积120MB

 * 在哪个时间段（小时），是学生们发帖提问的数量最多
 * 统计每一个发帖中问题的长度和回答的平均长度
 * 每个发帖提问都有标签，统计top10的标签
 * 统计哪些学生更加喜欢回答问题