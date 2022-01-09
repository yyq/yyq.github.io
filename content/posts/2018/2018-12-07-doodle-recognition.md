---
Title: kaggle | quickdraw-doodle-recognition top 6%
author: Young
date: 2018-12-07
tags: [kaggle, machine learning, cv, ]
Status: public
---
### 前言

这次是google举办的一个“你画我猜”的题目，题目链接：[kaggle - quickdraw-doodle-recognition](https://www.kaggle.com/c/quickdraw-doodle-recognition)。给的数据是游戏玩家在屏幕上画画的每一笔的坐标起始位置，然后让训练模型来猜出这幅画是画的什么，4900万张画，340个category。我记得微信上有个小游戏就是这么个东东。

本来没打算做这个比赛的，家里事情多了一些，然后毕竟cv也不是自己擅长的，后来吧，觉着GCP上还有1500多刀的过期时间和这个比赛时间一样，不用白不用，那就突破一下自己的comfort zone。比赛前期一直是自己在刷，最后8天的时候，自己做的两个单模在0.932，然后有找cv方面比较擅长的朋友组队，最后成绩还是略遗憾，没拿到银牌，和实力有关，也和运气有关吧，最后选交的不是队伍的最高分数。队友还是有比较伤心的，同时感觉自己年龄大了，对这个成绩和分数不是那么看重了，在论坛浏览了很多，和队友讨论了很多，过程中自己实践了很多，这才是对我最有价值的体验。

### 自己的收获

总的来说，认真从头到尾实践了cv的比赛，谢谢google送我的free credits，大幅修炼了我多显卡的实践能力（双卡？四卡？nonono，小伙子同时使用8个v100了解一下，还为了训练速度认真调优了显卡，cpu的使用率和io的效率等）。

图像有关的题嘛，卷积神经网络是肯定得用上了，然后笔画顺序有关嘛，循环神经网络肯定能有贡献，最后自己训练的两个单模se-resnet50和xception都是CNN类的，双向的lstm玩崩了，试了若干次都是第一个epoch顺利完成第2个epoch中间崩掉，至今没找到错误原因在哪。

下面总结一下自己的收获和踩过的坑吧：

* 先看来自蛙神的建议：bigger img size matters, larger batch_size matters, training number matters, more iterations matters,  简单验证过他这几个idea，全对！但是，卧槽了，这么些牛逼的建议再加上4900万张图，云主机党表示钱包好慌张
* 最新版的TF里自带的keras版本号是2.1.6，多GPU有神坑，性能奇差无比，别问我怎么知道的，花了老子好多钱之后才知道！后来单独用的最新版2.2.4的keras有很大提升。
* 2.2.4的keras的multiGPU的model也有坑，save_model的时候，得找到多GPU的model里的那个单GPU model的那一层，然后存下来
* keras的fit_generator之前用过，这次很熟悉的再来一次
* json.loads比ast.literal_eval快了10倍不止
* 制作img的速度，cv2和pillow这两个库不相上下，其他的都是垃圾
* 做3-channel彩色图的结果比灰度图的结果更好
* 做图的时候，line width matters，我猜是太细的笔画pool了几次之后就损失了细节
* pretrained并没有比随机初始化的好，至少我自己实验是这样子的，论坛上意见不一
* 图片数量太多，极其费时，用二分法查找到自己最大的batch_size
* 再次看到蛙神的帖子，感谢paper[https://arxiv.org/pdf/1810.00736.pdf](https://arxiv.org/pdf/1810.00736.pdf) ,这篇里有一张图展示了各类state of the art models的性能对比吧，主要是top-1 accuracy, top-5 accuracy, Operations(G-FLOPs)，这对资源吃紧而又想要分高的朋友特别重要了
* 16 cpu core的电脑，训练时别把16个core都用完了，还得留几个给处理多GPUmodel，10个workers比较合适
* 以前只知道在若干个epoch之后调整学习率，这一个epoch至少20个小时的训练肯定不能这么玩，习得新的调整LR的技法，比如[Cyclical Learning Rate](https://github.com/bckenstler/CLR) 或者是余弦学习率衰减, 原来keras的callback还有个默认的操作手法是在batch_end时有所动作，论坛上还有其他调整LR的算法，值得慢慢把玩
* 实在是时间有限，精力有限，以后参赛想要高分，对于我这类水平的人说，尽早开始多试错才能弥补专业知识的不足

### 观摩大佬分享

#### 第一名 [ods.ai]Pablos  [link](https://www.kaggle.com/c/quickdraw-doodle-recognition/discussion/73738)

CNN方面，尝试了这么多的模型: resnet18, resnet34, resnet50, resnet101, resnet152, resnext50, resnext101, densenet121, densenet201, vgg11, pnasnet, incresnet, polynet, nasnetmobile, senet154, seresnet50, seresnext50, seresnext101，1 channel的3 channel的图像，112的256的图像都试了，一共搞了40来个模型
(大佬是有多少机器资源？或者收敛速度是我的几十倍？)

RNN方面，他是参考了这个[kernel link](https://www.kaggle.com/huyenvyvy/bidirectional-lstm-using-data-generator-lb-0-825), 调整到了893

用LightGBM做ensemble，是参考的他队友之前发过的帖子[kaggle link](https://www.kaggle.com/c/cdiscount-image-classification-challenge/discussion/45733)，中心思想如下：如何解决多分类模型的组装？对于每一个模型预测的每一个sample，选取10个最高概率的标签，然后转换成10个sample，with 1 True 9 False，这样就很容易喂给booster了，然后他再加入一些时序特征，这里最significant的时间特征是最后一个笔画的最大时间戳。

秘籍分享：参考蛙神的[帖子](https://www.kaggle.com/c/quickdraw-doodle-recognition/discussion/70540#416772)，他发现测试集里的每一个类别的数量几乎是均分的，发现了这个之后，postprocessing就好做了，把结果里最popular的class的概率降低一点点，知道所有class的数量差不多为止，这么做给所有模型平均带来了%0.7的提升

融合：参考这个[weighted ensembling kernel](https://www.kaggle.com/paulorzp/ensemble-weighted-voting)，直接用提交的文件，拿来调整weights, 提升了千分之一。

上述三个步骤依次做下来。

数据：用了34000个随机样本作为valid，1M的样本用来做第二层模型，第一层模型都是用的49M

在评论区说起内存的时候，Pavel介绍了一个库，说这个很好用[pyro4](https://github.com/irmen/Pyro4)

我很好奇第一名用了多少显卡，评论区果然找到了有人这么问了，大神是这么回复的：in order not to discourage other participants let's say we had much more gpus than needed to win this competition.

#### 第五名 [ods.ai]resnet34  [link](https://www.kaggle.com/c/quickdraw-doodle-recognition/discussion/73708)

处理数据，4900w的图，我们每一张都单独保存下来了，400GB的磁盘，这样有利于后面的实验

模型主要是三个：se-resnext50, DPN-92, se-resnext101，全都用的imagenet pretrained，不同的图像大小，128，192，224，256，很快就直接上分到944了。

加入full dataset里的时间信息，把这么几个信息加入到3 channel里，delay value, draw time per stroke, number of strokes，local valida分数上涨了很多

#### 第11名  Draw me a star [link](https://www.kaggle.com/c/quickdraw-doodle-recognition/discussion/73808)

这是一个越南团队，他们队伍名称里还标了越南国旗，他们仅用了2个P40显卡搞定了比赛

CNN:
 * 先实验相对小的模型，100k/class, 64, 128, 224 
 * 重新训练with full data, se-resnext101 (.944), se-resnext50 (.943), se-resnet50 (.942), resnet50 (.942), densenet169 (.939), xception(.938), densenet121(.934) with batch size >= 400, 128x128 input size
 * 这个过程用了4个礼拜

RNN:
 * 试了kernel区的方案，改了改，deeper, bigger, stringer, 换LSTM为attention/GRU，加上raw data里的时间戳信息，先用100k/class first
 * 然后，用上述最好的一些超参数，全量数据训练，最好结果到了0.93x
 * 大概花了两个礼拜

Inference:
 * TTA (hflip) + Ensemble (0.8 * CNN + 0.2 * RNN) + optimization(Secret Sauce + magic wand)

弱点分析：
 * 图像像素和batch_size都没有尝试更大的
 * 没有找到一个合适的方式来训练好 unrecognized images
 * 最后的优化过程中出现了overfit












