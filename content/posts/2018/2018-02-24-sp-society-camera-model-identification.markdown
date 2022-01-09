---
Title: kaggle | sp-society-camera-model-identification 看图认相机
author: Young
date: 2018-02-24
tags: [kaggle, deep learning, neural networks]
Status: public
---
此题目标为根据照片来判断牌照相机的型号

这件事情的原理是：每一家设备都有自己的数字图像处理算法，总会有属于自己的图像特征

这件事情的意义在于：警察破案，图片是否有被软件修改等等

我的kaggle账号:[https://www.kaggle.com/yyqing/competitions](https://www.kaggle.com/yyqing/competitions)

## 0 自评

自己做的第二个图像类的题目，最大的收获就是体验到了extra data的威力，在论坛区看到有大家讨论说使用额外的一些的数据，50GB的训练集果然效果不错，直接把我的public LB成绩从0.87提升到0.92，后来经过自己的各种transfer learn和fine tune keras里的一些模型，然后组合，在private LB拿到了0.96的分数。看看排名前几的大神的总结里，很多人都使用了比我找的数据集更大的数据集，所以准确率也比我高出几个数量级（第一名0.9896）。

训练大量图像是一件非常非常消耗时间的事情，优化的方向有这么几个：1)切取大图像中的小块来训练;2)显卡计算能力利用率提高，注意显卡的频率，内存，注意程序中的CPU处理部分，做到及时处理一直给显卡喂数据，别让显卡时忙时闲 3)算法的优化，能并行做计算的部分尽量，不过注意估算好显卡的内存消耗，过大过小都是一种浪费 4)transfer来的不同的模型，参数数量不一样，最小图片像素也不一样 5) 做好pipeline 

## 1 高分答案集锦

### 第一名 [Pavel Pleskov](https://www.kaggle.com/c/sp-society-camera-model-identification/discussion/49367)

##### 模型
参考了Andres的代码，用pyTorch实现了，如下模型有较好的效果
```
984_densenet201_antorsaegen_29_0.98624
976_densenet201_antorsaegen_62_0.98271
977_resnet50_antorsaegen_119_val_0.9815
976_DenseNet201_do0.3_doc0.0_avg-epoch072-val_acc0.981250
967_InceptionResNetV2_do0.1_avg-epoch154-val_acc0.965625
962_Xception_do0.3_avg-epoch079-val_acc0.991667 (leaky validation, pls ignore)
```
所有模型的输入图像都用的512*512，并且都是Test Time Augmentation(测试时也用增强过的数据)。这里是最大败笔，因为后来发现及时用较小size的图片可以更快的训练，然后更多的TTA可以达到相似的准确率

##### 数据
我服，他另外下载了300GB从flickr和yandex.foto的图片。过滤掉分别率不适合的图片，质量不好的图片（图片质量可以用ImageMagick得到），自己用selenium做爬虫抓来的图片。

##### 硬件

五个队员，有五个1080ti和一个1070

##### 提交
* 平均所有的TTA所有的模型，by power mean powers of 1 2 4.最后有较大的LB overfit, public LB 0.991 private LB 0.985
* 通过hold-out，融合预测结果。20个xgboost，20个lightGBM，12个Keras，public LB 0.986, private LB 0.989.

##### 关键点
* 收集尽可能多的图像数据，做好清理
* 尝试更小的图片切图，尝试所有的架构类型
* LB probing是恶魔，相信自己的CV. (哈哈哈，完全同意,我本人就是按照自己最好的cv提交的，然后从public LB shake up从110名上升到64名的) 

### 第二名 [n01z3](https://www.kaggle.com/c/sp-society-camera-model-identification/discussion/49299)

这货直接搞来了500GB图片，不服不行，9个imagenet类似的模型，3个版本的pipeline。最后，几个平均了27个checkpoints。

关键点，按重要性先后排名：

* large and clean extra data
* consistant local validation
* classic competitive approach to learning models
* diverse models

##### data mining
下载了500多GB的数据，来自flickr，yandex，wikipedia commons, movile reviews.另外，最后一晚，我们还下载了另外的22k张图片，Andres提供的flickr图片

##### filtering data
lightroom photoshop 处理会抹掉相机的信息。我们过滤图片的条件有：相机类型，分辨率，jpeg压缩质量，软件处理。

##### 训练模型,不想翻译了，直接看原文比较合适
```
 Our code is based on pytorch version of Andres solution. For all the models, the binary flag is_manip was used as an additional feature for the classifier of a net. All models had an input of 480. For the majority of our models, five crops + flip photo orientation (10TTA) was applied to the pictures, for D4 models five crops + the whole group of D4 were applied (40TTA); then geometric mean was applied to the predictions. Useful tricks: 1. Adam, reducing LR on plateau with patience 2-4 2. Cyclic LR with SGD 3. Pseudo-labeling 4. Averaging 3 checkpoints with the best loss for validation
```

然后他贴了一张图，说明他用了哪9个模型，数据是，多少个ckp，然后LB得分

比较新奇的是，他除了用到其他帖子都用的resnet和densenet之外，还用了dpn，我上网查了查，dual path network：[https://arxiv.org/pdf/1707.01629v1.pdf](https://arxiv.org/pdf/1707.01629v1.pdf)
```
DPN是一种结合了ResNet和DenseNet优势的新型卷积网络结构。
在ImagNet-1k数据集上，浅DPN超过了最好的ResNeXt-101（64×4d），具有26％更小的模型尺寸，25％的计算成本和8％的更低的内存消耗
```

在评论区，第二名的作者也说道了，**DPN is fantastic! Our best single model: dpn92 with pseudo labeling, TTA, 3 checkpoints TTA: 0.987 (private) and 0.984 (public).**


##### what did not work ? Demosizing

##### 硬件
```
i7 7700k, 64gb, 2x 1080 (just for development)
i7 6700k, 32gb, 2x Titan X (Maxwell)
i7 5930K, 32GB, 3x1080Ti
Xeon 2696v3, 64gb, 4x1080Ti
i7 3770k, 16gb, 2x 1080Ti
```

### 第三名 [Luisa Verdoliva](https://www.kaggle.com/c/sp-society-camera-model-identification/discussion/49602)

竟然是个妹子！意大利妹子！浪漫豪放的意大利妹子，竟然还有做机器学习的，服

##### 数据
每个相机类别，只用了450张图哎，400个训练，50个cv

##### 训练


##### 方案总结



##### 硬件



##### 特别提要



##### lessons learnt



## 2 评论区精华摘抄

[https://www.kaggle.com/devm2024/keras-model-for-beginners-0-210-on-lb-eda-r-d/notebook](https://www.kaggle.com/devm2024/keras-model-for-beginners-0-210-on-lb-eda-r-d/notebook) 这里的3D图像画的很棒，让人肉眼直接看出一艘船和一个冰山的区别。

[https://www.kaggle.com/muonneutrino/exploration-transforming-images-in-python/notebook](https://www.kaggle.com/muonneutrino/exploration-transforming-images-in-python/notebook) 

1. 统计了图像上的像素的max,maxpos,min,minpos,med,std,mean等，有效的特征工程，
2. 先画了各种各样的图，图像的题目嘛，先看图比先分析会更加有效
3. 做了smooth等等各种图像操作，能肉眼看出一些端倪，然后可以让机器做的更好

[https://www.kaggle.com/devm2024/transfer-learning-with-vgg-16-cnn-aug-lb-0-1712](https://www.kaggle.com/devm2024/transfer-learning-with-vgg-16-cnn-aug-lb-0-1712) transfer learning from VGG16, skill point get.

[Best Single Model LB1400](https://www.kaggle.com/supersp1234/best-single-model-lb-0-1400/code)确实是我看到的最好的单模型了。 基于torch，对图片有进行随机水平翻转，随机竖直翻转

[Open sourcing my PyTorch CNN Ensembler](https://www.kaggle.com/c/statoil-iceberg-classifier-challenge/discussion/44849#273821)
















