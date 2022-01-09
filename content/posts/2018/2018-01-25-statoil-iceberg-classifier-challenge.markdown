---
Title: kaggle | statoil-iceberg-classifier-challenge 捡到铜牌一个
author: Young
date: 2018-01-25
tags: [kaggle, deep learning, neural networks]
Status: public
---
此题目标为识别图像中是否有冰山，数据图像均为卫星拍摄海面获得

我的kaggle账号:[https://www.kaggle.com/yyqing/competitions](https://www.kaggle.com/yyqing/competitions)

## 0 自评

第一次开始尝试图像的题目，和普通回归类的题目还真是不一样，除了修改一些别人的模型看得懂别人模型之外，自己还没有水平直接改进到10%。coursera的课程知识不够用了，觉得自己有知识瓶颈了，计划把cs231n刷一遍（中文字幕的，编程作业一定自己完成的），目标要至少做到：在不做实验的情况下，增删改查模型的各个地方，对自己的运行结果，得利用先验理论知识进行预判。这样的话，可以在实践各种方案的同时，持续强化自己对理论知识的深刻理解。

尝试使用了keras，确实好用，方便快捷，生成图像的函数ImageDataGenerator很好用。不过听说pytorch的速度比tensorflow要快啊，keras的底端默认是调用的tensorflow，下次编程实践试试pytorch。

有几次自己思考过后，改了改模型运行完提交之后，看到public LB得分下降了，就没有继续尝试了，然后赛后才看到那几次在private LB上的得分还升高了呢。所以啊，还是得相信自己的知识，不应该一味的看着public LB然后怀疑自己的知识。

## 1 高分答案集锦

### 第一名 [David](https://www.kaggle.com/c/statoil-iceberg-classifier-challenge/discussion/48241)

* 画图理解图像很重要,把数据项inc_angle可视化出来很重要,他也发现了 
* inc_angle的重要性,不同类型的分类，很重要。
* 始终坚信自己的本地cv很重要。
* 他有个cnn pipeline，直接先100个模型训练一下看看，惊掉我下巴有没有
* 对inc_angle做聚类算法，识别分类，确认此参数非常有效，logloss对于极端值错误惩罚比较严重，设置一个阈值好了
* 重新针对另外的类别，训练100多个模型，这下训练效果非常好了，对比其他参赛者，有显著提升
* 附上作者原文:So in summary, 200+ CNN models, a handful of clustering algorithms, a few different thresholds, a heavy reliance on inc_angle, don’t push log_loss thresholds to the extreme, trust your local CV, and you have a formula for a winning solution. Cheers!

* Q:怎么调的200多个模型。A:随机搜索的，用cv只选最好的出来
* Q:post-processing很重要哈
* Q:怎么做的ensemble和stack。A:ensemble就是根据CV选出得分高的，然后直接平均，stacking就是经典的方法，基本模型的答案，用xgboost作为level-2的再训练一遍。没有用min-max因为怕过拟合。

### 第二名 [beluga](https://www.kaggle.com/c/statoil-iceberg-classifier-challenge/discussion/48294)

* 概括：训练了上百了XGB和上百个CNN，也利用好了data leak.暴力搜索对于获得稳定的local/public/private很有帮助！
* Layer1 - CNN
 * fix 5 fold cv, hundreds CNN, different random parameters. 分fold是基于incident angle来分的，怕leakage。主要用了3类model families: 1)大部分时间custom CNN from scratch on 2 channel; 2) finetuned VGG16 加上第三层的band1+band2平均； 3）double训练数据加上伪标签，不过最后觉得没啥用
 * 因为数据集相对小一点，我倾向于训练an army of weaker but different models instend of 优化最好的模型。数据增加方面，用了flip flop zoom rotation shift。随机搜索的参数如下：
 ```
 preprocessing in ['mean_std', 'minmax', 'raw']
 cnn_type in ['simple', 'double']
 conv_layers in [[32, 48, 64], [32, 64, 128], [64, 128, 256], [64, 128, 128, [32, 64, 128, 256], [64, 100, 128, 150]]
 dense_layers in [[64, 32], [128, 64], [256, 128], [512, 256], [128, 64, 32]]
 conv_dropout in [False, 0.1, 0.2]
 dense_dropout in [False, 0.25, 0.5]
 learning_rate in [0.0004, 0.0008, 0.001, 0.002, 0.004, 0.005]
 optim_type in ['Adam', 'RMSProp', 'SGD']
 flip0 in [0, 0.25, 0.5]
 flip1 in [0, 0.25, 0.5]
 rotation in [(0, 0, 0), (0.25, 0, 20), (0.5, 0, 20)]
 zoom in [(0, 0, 0), (0.25, 0.8, 1.2), (0.5, 0.8, 1.2)]
 shift in [(0, 0, 0), (0.25, 0, 0.1), (0.5, 0, 0.1)]
 ``` 
* Layer2 - XGBoost
 * 放弃了观察缺少incident angle的数据。用最好的5 10 15 20个模型（都从3类里面选），针对每一个fold。每一个类别都在leaderboard上验货得分在0.165到0.173之间。
 * additional features were extracted for each: 
 ```
 Observation
 L1 prediction stats(min, mean, geomean, median, max, std)
 Raw inc_angle
 Raw stats for both HH and HV(min, mean, median, max, std, quantiles)
 Distinct incident angle groups
 Count, train count, ship count, iceberg rate
 L1 prediction stats(min, mean, geomean, median, max)
 Closest K incident angle roups
 Distance
 Mean Prediction
 Incident angle bins
 Mean Prediction
 Count, train count, iceberg count
 ```
 * xgb的参数是随机的on same fix CV splits as in previous layer.
* Layer3 - Model average
 * 最后9天，尝试了更大的CNN,更多的基于angle的特征，更多的XGB，自己的trainingCV从0.116下降到0.1以下但是无法从public score提升了。
 * 最后1天，重新训练了上千个xgb，用更加保守的参数，尽全力避免CV过拟合
 ```
 round all features to the 3rd decimal
 min_child_weight >= 3
 colsample <= 0.5
 max_depth in [3,4,5]
 switched off early stopping
 ```
 * 最后结果 %95 平均了100个最好的xgb，%5 平均了100个最好的不带inc_angle的xgb
 * Quite a few interesting postprocessing technique were suggested in kernels and discussions. I experimented with clipping, geometric mean, rounding for confident predictions. They did not gave stable improvement in my local validation so I rejected any postprocessing. To be on the safe side I wanted to clip my final submission anyway but model averaging already solved the problem all my predictions were in (0.005, 0.995). I selected my best public LB and latest submission the best would be 8th on the private leaderboard while the final gave me the 2nd place.
* ENV
 * I used keras with tensorflow for the cnns. I do not have my own GPU which is actually quite painful in computer vision challenges. I used to rent spot p2.xlarge EC2 machines with K80. Unfortunately they are not exactly cheap (0.37$/hour) and they are actually quite slow compared to the best cards. In general I prefer more data however here I enjoyed the quick training times and fast iteration a lot (single model could be trained in minutes instead of hours or even days!). At the beginning I used my custom AMI with manual installed libs. It could be painful to keep everything uptodate so I was happy that free (at least free in terms of software) maintained Deep Learnning AMIs are available now. 


### 第三名 [Evgeny Nekrasov](https://www.kaggle.com/c/statoil-iceberg-classifier-challenge/discussion/48207)

* 训练了7个ANN，怕过拟合，也为了得到smooth的预测，用了5folds并且重复30遍。不同的ANN用了不同架构，数据归一化，预处理，数据增加
* 用xgboost混合7个ANN的预测，stacking way using 7 folds 重复1000次
* 利用好inc_angle, 做好伪标签，再次开始训练

### 第四名 [Andrii Sydorchuk](https://www.kaggle.com/c/statoil-iceberg-classifier-challenge/discussion/48163)

* 训练了5个神经网络，每个都用cv5同样的split，NN tuning 用了不同的架构，最好的数据增加参数，最好的CV5-seed
* 又一个好好利用inc_angle哎，我是一个正经的数据科学家啊，别总是给我data leak解决方案嘛！！！！
* LightGBM level-2 stack, tune LightGBM
* 把最后结果修整到(0.001,0.999)区间，预防措施

## 2 评论区精华摘抄

[https://www.kaggle.com/devm2024/keras-model-for-beginners-0-210-on-lb-eda-r-d/notebook](https://www.kaggle.com/devm2024/keras-model-for-beginners-0-210-on-lb-eda-r-d/notebook) 这里的3D图像画的很棒，让人肉眼直接看出一艘船和一个冰山的区别。

[https://www.kaggle.com/muonneutrino/exploration-transforming-images-in-python/notebook](https://www.kaggle.com/muonneutrino/exploration-transforming-images-in-python/notebook) 

1. 统计了图像上的像素的max,maxpos,min,minpos,med,std,mean等，有效的特征工程，
2. 先画了各种各样的图，图像的题目嘛，先看图比先分析会更加有效
3. 做了smooth等等各种图像操作，能肉眼看出一些端倪，然后可以让机器做的更好

[https://www.kaggle.com/devm2024/transfer-learning-with-vgg-16-cnn-aug-lb-0-1712](https://www.kaggle.com/devm2024/transfer-learning-with-vgg-16-cnn-aug-lb-0-1712) transfer learning from VGG16, skill point get.

[Best Single Model LB1400](https://www.kaggle.com/supersp1234/best-single-model-lb-0-1400/code)确实是我看到的最好的单模型了。 基于torch，对图片有进行随机水平翻转，随机竖直翻转

[Open sourcing my PyTorch CNN Ensembler](https://www.kaggle.com/c/statoil-iceberg-classifier-challenge/discussion/44849#273821)
















