---
Title: kaggle | favorita-grocery-sales-forecasting
author: Young
date: 2018-01-17
tags: [kaggle, machine learning, regression]
Status: public
---
我的kaggle账号:[https://www.kaggle.com/yyqing/competitions](https://www.kaggle.com/yyqing/competitions)

输入数据为某连锁商店的各个店铺各个商品的销量，预测接下来16天的个店铺的各个商品的销量

在Public LB做到了3%，等privateLB出来之后，掉到了6%，发现提交历史记录里，自己的最好的单模型成绩比自己最后提交的组合模型答案成绩还好，可是最好的但模型成绩在publicLB都没有进入10%,心里慌啊，用尽全力的调整到3%啊。终于，自己对LB做到了overfit了，呵呵，教训啊！我的最好单模型成绩public LB 510, private LB 518, 组合模型成绩public LB 508 private LB 519.

## 1 Lessons learned of myself:

1. 特征工程比模型调参更重要，重要性大出一个数量级
2. 模型调参时，学习率先从较大的数字开始，节约时间
3. 有关日期的比赛，本地cv日期的选择很重要，和最终测试日期有相似性才比较好
4. 本地的每一次运行，cv，参数，分数都需要很好的记录下来，供后期对比分析
5. 很多大神都开始在回归类题目里面开始用NN了，确实成绩会比xgb lgb等会有较大提升

## 2 Lessons learned from others:

总结两个观点先：

1. 特征工程特别重要
2. 神经网络要胜过决策树boost了

每一段摘要都标记原作者，原文章标题，和原文链接

### 2.1 [Eureka 1st place solution](https://www.kaggle.com/c/favorita-grocery-sales-forecasting/discussion/47582)

借鉴了四个别人的模型，cnn，lstm，lgbm，lgbm

只用了2017年的数据，训练集，0531-0719 or 0614-0719, 不同的模型用不同的训练集，验证集，0726-0810

数据处理：把空值和负值都填0

特征工程：

1. 基本特征，分类（store,item,family,class,cluseter）,打折否，day_of_week(only for model 3);
2. 统计特征，时间窗口，最近日期:[1,3,5,7,14,30,60,140]；等时间窗口[1] * 16, [7] * 20; **关键特征：store x item， item， store x class**, target： promotion, unit_sales, zeros, 方法：mean,median,max,min,std, day since last appearance. difference of mean value between adjacent time windows(only for equal time windows)
3. 无用的特征，节假日， 其他的keys例如，cluster x item,  store x family

单模型：

1. model_1 : 0.506 / 0.511 , 16 lgb models trained for each day [code](https://www.kaggle.com/shixw125/1st-place-lgb-model-public-0-506-private-0-511)
2. model_2 : 0.507 / 0.513 , 16 nn models trained for each day [code](https://www.kaggle.com/shixw125/1st-place-nn-model-public-0-507-private-0-513)
3. model_3 : 0.512 / 0.515，1 lgb model for 16 days with almost same features as model_1
4. 0.517 / 0.519，1 nn model based on @sjv's code

组合模型：
Stacking doesn't work well this time, our best model is linear blend of 4 single models.
final submission = 0.42*model_1 + 0.28 * model_2 + 0.18 * model_3 + 0.12 * model_4
public = 0.504 , private = 0.509


### 2.2 [Luck Yu 2nd place solution overview](https://www.kaggle.com/c/favorita-grocery-sales-forecasting/discussion/47568)

他上来先感谢了sjv CPMP。

用的wavenet

Feeding data with mini-batch with randomly sampled 128 sequence. Then randomly to choose the start of decode/target date. So, we could say somehow the model will see different data for each training iteration. because the total dataset is around 170000 (seq) x 365 days。这么训练下来，wavenet应该能够处理好过拟合

用最后16天来做验证集

### 2.3 [slonoslon 3rd place solution overview](https://www.kaggle.com/c/favorita-grocery-sales-forecasting/discussion/47560)

lgbm + nn. 遇到时间序列问题的时候，两个最重要的事情是validation和bagging。

训练集, I took a history of 80 consequtive sales days for each (item, store pair), and used the next 16 sales days as a train set target.

about bagging, our final result includes models, trained on 10 runs each. 直接平均。

另外一个部分的bagging，what the FUCK ?  After each training epoch (including the very first) I predicted the targets, and then averaged these predictions. I've read about averaging a few last epochs, but never heard about averaging everything, even the first very underfitted epoch. Yet my validation consistently showed that it does improve the results.

组合方案，组合了CNN LSTM GRU，直接平均。All of them have 3 top (convolutional or recurrent) layers, 2 or 3 bottom fully-connected layers and embedding layers for categorical features


### 2.4 [sjv 4th-Place Solution Overview](https://www.kaggle.com/c/favorita-grocery-sales-forecasting/discussion/47529):

只分享他的单模型取得.499成绩

模型：双向LSTM，他的源代码[code](https://github.com/sjvasquez/web-traffic-forecasting)

input representation：原始的时间序列的值，分类变量，手动做的特征（lags,diffs,rolling statistics, date features, conditioning time series, average sales for a given product/store/etc.）

validation: 用的是过去365天随机的，这么做的目的是为了防止过拟合于某一个validation set/weekly/monthly trends.

各路大神都吐槽了这个题目给的数据集的表示方式的错误。包括user ranking总榜第一的人也吐槽了。

### 2.5 [Lingzhi 5th Place Solution](https://www.kaggle.com/c/favorita-grocery-sales-forecasting/discussion/47556)

用了3个模型，lgbm，cnn+dnn, seq2seq RNN. 直接平均。每个模型都用不同的随机种子训练多次，求平均值。每个单模型可以做到在private LB里1%（这个就比较屌了）。

lgbm，公开kernel的的升级版本，加了更多的特征，数据，日期。

cnn+dnn, This is a traditional NN model, where the CNN part is a dilated causal convolution inspired by WaveNet, and the DNN part is 2 FC layers connected to raw sales sequences. Then the inputs are concatenated together with categorical embeddings and future promotions, and directly output to 16 future days of predictions.

RNN, This is a seq2seq model with a similar architecture of @Arthur Suilin's solution for the web traffic prediction. Encoder and decoder are both GRUs. The hidden states of the encoder are passed to the decoder through an FC layer connector. This is useful to improve the accuracy significantly.

特征工，对lgb来说，加了mean sales，count of promotions, count of zeros. 在nn中，item mean和year-ago和quarter-ago sales. categorical featues and time features(weekday, day of month) are fed as embeddings.

据说这个帖子给他帮助很大，[web traffic prediction](https://www.kaggle.com/c/web-traffic-time-series-forecasting/discussion/43795)这个很有用，比较通用的时间序列解决方案。

### 2.6 [Nicolas 6th Place Solution Overview](https://www.kaggle.com/c/favorita-grocery-sales-forecasting/discussion/47575)

组合了两个可以 517 private的单模型， 基本思路为：lgbm使用2017的数据，超过500个特征，相对可靠的cv on 0726-0810

他把他所有做的特征按照重要顺序都排列出来了，这个厉害了，我就直接引用吧

* ma_median * isd_avg / isd_week_avg - popularized early on in the competition.
* ma_median - Coupled with binary for whether or not it is equal to 0. 
* Day-of-week averages - Different time periods including 7, 14, 28, 56, 112.
* Days since appeared - Difference between the 'start date' of the training cycle and the first date the item showed up in the original train file.
* Quantiles for several different time-spans.
* Whether the item will be onpromotion.
* Simple averages for different time-spans.
* Item-cluster means - The past 5-day mean was a good predictor.
* Future promotional 'sums' - E.g: sum of onpromotion 8/16 through 8/18.
* mean_no_zero_sales - Mean of instances where unit_sales > 0.
* Frequency encoding - calculate the frequency of appearance for items, stores, families and classes (four columns that each sum to 1).

上面这些特征都是能够直接提高成绩的重要predictors，下面的这些特征是从整体上提升

* One-hot-encoding clusters, states, types, cities and families.
* Weekend and weekday means.
* Staggered mean data (e.g. 8/07 --> 8/14 for the test-set).
* Staggered quantile data.
* Past number of zero sales for different time-spans.
* Past number of promotional days.
* Past promotional sales averages.
* Past day-of-week quantile data.

他的队友也是使用的lgbm，不过都使用了不同的特征。


### 2.7 [CPMP 8th solution](https://www.kaggle.com/c/favorita-grocery-sales-forecasting/discussion/47564)

single NN got 508 on public and 515 on private. single lgb got 514 on private.

缺失值的处理：用之后的数据直接填补之前的空白。

seasonality，时间序列处理中，两个关键点是时期性和cv设定。这个题目中weekly时期影响很强而yearly很小

cv setting,采用了两段时间2017-7-15，2017-7-31 和 2017-8-1，2017-8-15

特征工程，log1p，mean,max，proportion of zero entries,过去7天的每个单天销量，过去八个礼拜的相同weekday的均值，对于每个日期列出过去364天的销量和促销状态，产品的类别，商店

神经网络模型，3个dense层 relu激活。class和store被编码成4个维度的向量，一次可以预测16个了，最后卷积一个onpromotion向量

其他模型：lgb做到了510 on public lb and 517 on private lb

Giba大佬评论说了：**I always say and continue saying, don't trust public LB. Specially in a time series challenge where public LB is calculated using 5 days ahead and private is from 6 to 16 days ahead, they are completely different models.**











