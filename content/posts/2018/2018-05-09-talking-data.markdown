---
Title: kaggle | talkingdata-adtracking-fraud-detection top 4%
author: Young
date: 2018-05-09
tags: [kaggle, machine learning, LightGBM]
Status: public
---
题目链接:[kaggle](https://www.kaggle.com/c/talkingdata-adtracking-fraud-detection), [talkingdata](https://www.talkingdata.com/)是一家中国公司，我将其理解成一个第三方的移动数据平台，移动端的广告分发是其一个重要业务，其中会出现一些恶意的点击，这个比赛的意义在于：他们想要通过机器学习的方式抓出其中的欺诈类型的点击。

我的kaggle账号：[https://www.kaggle.com/yyqing/competitions](https://www.kaggle.com/yyqing/competitions)

## 纪念一下自己的第一个solo 银牌

之前有个比赛进过前5%的，不过由于用了两个账号在轮番提交，被取消成绩了。这次算是记住教训了，就一个账号玩，依然solo，排名122/3967，public LB score 0.9898130, private LB 0.9821300, mark一下，离自己的mater目标迈出了重要的一步。另外值得庆幸的是，这是第若干次比赛成绩出来后，我的私榜排名是比公榜排名进步的，证明我的模型的泛化能力一直还行，至少是领先于kaggle上大家的平均水平的。

这次比赛的最后一次ensemble结果提交是在回湖南的火车上完成的，正好看到kaggle社区总积分的新晋No.1--bestfitting的访谈博客，这位同学竟然是长沙人，也是蛮意外的。我努力向老乡学习吧。

## 自评：又一场内存的战役

学到了很多，这些知识大概不会有哪本书籍有写，也不会有哪个视频教程说，都是自己摸爬滚打得来的，主要是对于大量数据的存，取，和运算。公司也没有给我集群机器玩的情况下，只能自己在谷歌云上玩了。当自费的玩大容量云主机的时候，全都按时收费，每一个GB内存和硬盘都要收费，每一个cpu core都收费的情况下，我会格外的在意，我会最大化的利用cpu，内存等"昂贵"资源。穷嘛，省时间就是省钱。

接下来就说一些在有关方面的技巧吧

* 习惯性的del不再使用的数据，then gc.collect() could save your asshole.
* 很多时候调整模型都要考虑使用或者更改不同的特征，当你有几十个特征，每个特征的存放都会占用GB级别的资源的时候，不要尝试一次性处理所有的特征。一个一个来，180Million的数据，一个特征，一次开一个进程，只计算一个特征，比如groupby过程中会爆炸到20多GB的内存，虽然最后计算完结果只有1GB内存。
* 不要尝试将包含所有特征的大表存成一个文件，将每一个特征保存成一个单独文件。之后要用到这个特征的时候，再读取进来join到dataframe中。提高灵活性，用哪些我就只载入哪些。
* pandas读取大体积（1GB或者更大）csv文件速度很慢，特别是当我有几十个特征，每个特征都有一两GB的时候，pandas读取一个30GB的csv文件所花费的时间，绝对会让你崩溃。这个时候```pandas.read_feather()```和```pandas.to_feather()```是一个很好的方式，直接把一个dataframe的内容，以feather的格式，从内存dump到磁盘，不用管csv的index等等相关参数，就是dump什么样的内容到磁盘，读取回来还是什么内容在内存，重点是，速度比read_csv快了一个数量级。
* 关于pandas有点慢的问题，期间还认真尝试过dask和pandas on ray两个开源工具，dask功能齐全一些，在有一些操作比pandas快很多，不过在这次实践中对我的时间节约没有太多的帮助，所以没用。pandas on ray的话，显然还不太成熟，各种常用函数缺失，遂放弃。
* 每一次探索新特征的时候，一定要有一个较小的可用的可靠的快速的validation的方案，来验证新的特征是否有效。不然每次都用全量的数据来计算，时间成本太高。
* 每一次运行时间较长的实验（比如全量数据，比如超小的学习率）之前，一定得有一个debug pipeline，快速走完整个过程。
* 养成随时save model的好习惯。之前做图像深度学习的时候还知道存下很多个epoch完毕之后的模型文件，没成想lightGBM在遇到稍微大点的数据集的也会有各种坑。比如有时候会遇到训练到一半，内存炸了，程序崩了，比如有时候会train完毕后没有及时清理删除训练数据集，predict的时候内存炸了。想起mxnet的李沐博士的视频里说的，我就是不小心把整栋机房弄停电了嘛，你们跑了三四天的程序都不保存中间模型，那能怪我喽，谁叫你们不随时保存的。
* 哎呀，说多了都是泪，对于我个人来说都是浪费的钱啊，以后想起来什么泪点，我再补充到这个段落来。

## 高分答案集锦

接下来欣赏欣赏别人的思路吧

#### 3rd, bestfitting, [NN based solution](https://www.kaggle.com/c/talkingdata-adtracking-fraud-detection/discussion/56262)

在同时做另外一个google landmark的题目，正好发现最近的NN模型可以用，就在这里也直接用NN了。（大神就是可以模型广泛应用到各个业务上，难道以后深度模型要统治所有，彻底打败树么）。他的主要尝试了的是23个特征，用single LGBM做到了在public LB分数0.9817。（这分数比我最终提交的分数还高），这是他第一次在kaggle里用LGBM。

设计了基于那23个特征，并且适当处理后(NA, out of vocabulary, scale)，用NN做到了公榜0.9820，然后论坛上发现click delta是个重要线索，于是多做了前五个和后五个click的时间序列加到网络里，用RNN来寻找点击序列的模式，模型可以做到公榜9821了。然后设计了不同的神经网络来增加多样性，都是很简单的，比如增加一些残差链接到dense层。四个模型，不单独考虑分数，只考虑综合起来的多样性带来的提升。在组装了神经网络和lgbm模型的成绩之后，分数到了9827。

再加上n-fold和全部数据。稍微有点花时间了，一个fold就两个多小时，一块1080i的GPU.

然后用之前的模型在整个数据集上预测的结果，再加上一些特征（基于IP的，app-os-channel），做了第二层神经网络,分数到了9833。最后的这个提升做好的时候，离比赛时间结束只有30个小时了，来不及其他改进了，时间限制，NN模型的弱点，等等，就酱。他用到的一些特征如下：
```
channel                                  1011
os                                        544
hour                                      472
app                                       468
ip_app_os_device_day_click_time_next_1     320
app_channel_os_mean_is_attributed         189
ip_app_mean_is_attributed                 124
ip_app_os_device_day_click_time_next_2     120
ip_os_device_count_click_id               113
ip_var_hour                                94
ip_day_hour_count_click_id                 91
ip_mean_is_attributed                      74
ip_count_click_id                          73
ip_app_os_device_day_click_time_lag1       67
app_mean_is_attributed                     67
ip_nunique_os_device                       65
ip_nunique_app                             63
ip_nunique_os                              51
ip_nunique_app_channel                     49
ip_os_device_mean_is_attributed            46
device                                     41
app_channel_os_count_click_id              37
ip_hour_mean_is_attributed                 21
```
(可以看出来几类特征比较有效，click_time_next，target mean code，count，unique count，还有就是大多特征都是基于ip和其他特征组合来的)

为了让神经网络可以工作，预处理非常重要，如下：1. ```Fill-NA, Fill max value for delta features, fill mean for other feature``` 2. ```Log of delta features. part of count features when values are large and all the nunique features``` 3. ```StandardScale```

最后他谈到了，关于私榜分数估计，我就直接引用过来好了：
```
As to private score estimation,it's always a interesting part of my competition :).
There is not certain method to do so,I just bear in mind:
Distribution variances lead to score variances.
For example,in this competition,some category values in test-set are not in trainset,so I changed the same ratio of category value of validation set to values unseen in trainset.And the App19 is a very important app with high ratios of download and is imbalance in train and test-set.What's more,the ratio is differenct between the public and private set,so I tried to keep the ratio of my validatition as test set...We can also use a submission which I believe it is stable as True label,and caculate AUC based on it,if the public and private score as close,then we can believe it too. 
```


#### 6th, CPMP, [link](https://www.kaggle.com/c/talkingdata-adtracking-fraud-detection/discussion/56283)

鉴于是个人solo而且数据集还这么大，那么一次提交的计算可能要花费数个小时，所以决定只focus一个类型的单模型，LightGBM。时间花费大致如下：80%的时间做特征工程，10%的时间尽快做好有效的local validation，5%超参数调优，5%组合模型。

花了大量的时间做特征选择，由于自己的机器资源，无法搞定50个特征，要是早点看到[这个帖子](https://www.kaggle.com/c/talkingdata-adtracking-fraud-detection/discussion/56105)，那么有可能使用更多的特征。

我的最终成绩是一个单模型，48个特征，9825 public and 9835 private, 另外有一个5模型组合的成绩更好，不过最后没有选择那次submit。我的机器是20核心，64GB内存和64GB swap。机器稍微有点慢了，不过可以有20个线程。我还有另外一个机器是4核心i7+2GPU的用来跑keras。

###### **validation**

这个是关键，如果没有一个好的验证方案，那么只能靠LB来验证，就会很容易导致对public LB的过拟合。最后我这么设置的：用day 8来做训练，然后用day9-hour4 和 day9-hour5,9,10,13,14两种方式来做验证。用所有数据训练的时候，树的数量用之前部分数据时的1.2倍。

用两个验证集来确保自己没有严重的过拟合。选择这些小时数据是因为test set里的小时。在早些时候，我也关注了train auc,有些特征提高了validation的分数，但是同时也拉大了与训练集分数的gap，放弃掉这部分特征。后来为了加速训练，放弃了关注train auc.

同时也关注LB，只有特征同时提升了local auc和LB，才选用。

这个策略非常有效，用hour4来做验证机可以做到和LB分数基本一致，或者有大约千分之一的方差。然而，为了获得earlystopping而一直计算auc，这个是很消耗时间的。为了快速的feature evaluation,我尝试了两个加速方案，第一个是：只用day9中，hour4的5%的数据来做validation，其他的用来训练。这个跑一次只用不到一个小时，用来过滤特征。这个方案很有效，分数对于LB的分数也有不错的相关性，但是对于lag类的features不太有效。所以后来我选用day8的数据了。第二个是：每一个feature都分别存放在一个feather文件中。一个特征一个特征的添加到训练中，如果local validation分数至少提升了0.00005才保留这个特征。也会这么干，一次添加进来多个特征，remove one by one。整个比赛期间，就是在不停的选择特征。

###### **Feature Engineering**

用了这么几类特征：

* 原始特征列中，只保留了app和os。这个被当做categorical来处理。这两个是最强的特征，第三个强原始特征类是hour in day.
* 中国时间，24小时制，从4pm开始（中国东八区）。这些被用来生成lag feature
* 用户有关：ip，device，os，triplets
* 不同特征组的聚合，很多公开kernel里都有，我用的有count, unique count, delta with previous value, next value, time to next click when grouped by user was important.其他有用的但是在公共kernel里没找到的有，delta with previous app.
* Lag feature, 基于中国时区的值，一些group的 previous count，previous target mean。
* ratio，例如clicks per ip，app to the number of clicks per app
* not last, 这个可以用来抓到leak，
* target mean code，也是用来抓leak的。先是group by了user, app, click time等。这类特征带来了0.0004到0.0005的提升。
* 矩阵因式分解（这个厉害了）。这个是用来捕捉users和app之间的相似性的。首先，构造一个矩阵，log of click counts.我用了这三个：ip x app, user x app, and os x device x app。这些矩阵都极其稀疏。前面两个，用了sklearn里的svd，，得到ip和user的embedding。对于最后一个，有三个因子，我实现了libfm with keras,也得到了对应的embeddings。有了这五个embedding，给我的分数提升带来了0.0010.这个直接带我进入top10了。我在一些不同的模型使用了不同的embedding。

###### **超参数调优**

没怎么调，scale position用的大约为400，为了防止过拟合，31个叶子，深度为8

###### **ensemble**

直接平均的几个预测。本来想用validation的预测结果，加几个day9的特征，来进行训练，但是时间太久没搞定。可能即使搞定了也对我的LB提升不大了，毕竟我的模型多样性很小。看到bestfitting用了这一招，赞他一个。

###### **take away**

kind of crazy for solo people, 对于一个人来说工作量太大了。我很羡慕那些走了同样的道路的人。一旦solo了，路就不一样了，或许不solo的话，我可以尝试更多的数量的特征。最后，我在最后一天有了很多进展，从9824提升到9832，这告诉我永不放弃。

#### 4th, KazAnova, [brief tips](https://www.kaggle.com/c/talkingdata-adtracking-fraud-detection/discussion/56243)

先赞一下我的队友们，无比努力。在赞一下大家。赞一下所有分享代码和思路的人。特别感谢[anttip](https://www.kaggle.com/anttip)带来的[wordbatch](https://github.com/anttttti/Wordbatch)的kernel，让大家眼前一亮，简直了。

我最喜欢拿第一名和第四名了，第一名可以得到最多的点数和钱，第四名没有那么多点数，但是不用复现自己的解决方案。（大神对名次的看法就是不一样，看样子常拿前几名，吾等只能最求名次越高越好，没得选）

我的验证方案如下：用day78来训练，用day9的hour[4,14]来做预测。对于测试集的预测，用day789来训练。

1）当我们开始做stack的时候，可以得到一两个点的提升，从9818到9820，在用所有特征加入到我们的标准模型后，加到了3点的提升，到了9823，一共大约50个模型，大部分是lightgbm，也有nn，FMs和线性模型。

2)提升了4个点，厉害了我的哥，用了[WoE](https://github.com/h2oai/h2o-meetups/blob/master/2017_11_29_Feature_Engineering/Feature%20Engineering.pdf)思路来做特征工程，各种他们（ip,app,device,os）的组合。发现一个有点奇怪的地方哈，相似的特征的列的不同的顺序，带来了不同的结果。可能这个和lgbm的binning方式有关。

3)又提升了4点，把group of ip,app,device,os等按照时间排序，同一时间里同样的点击，然后总是把attributed=1的排到最后。

普通的特征count之类的，groupby不同的组合。比较重要的特征有next click previous click等等。最重要的特征，是把app当做categorial来看待。因为可以发现有些app里存在非常高的概率出现is_attributed。极其有效。