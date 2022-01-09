---
Title: kaggle | avito-demand-prediction top 10%
author: Young
date: 2018-06-29
tags: [kaggle, machine learning, LightGBM]
Status: public
---
题目链接:[kaggle](https://www.kaggle.com/c/avito-demand-prediction), [avito](https://www.avito.ru/rossiya?verifyUserLocation=1)是一家俄罗斯公司，从网站上来看是一个线上购物平台，这一次题目的目标，就是预测某一个商品在某一天被售出的概率，给定的数据有一段时间内的商品的销售情况（数量，价格，地区，品类，商品的全面俄文描述，商品的图片）等信息。

我的kaggle账号：[https://www.kaggle.com/yyqing/competitions](https://www.kaggle.com/yyqing/competitions)

# 丢了top10%

自己的时间规划出了点问题，工作上临时加了点任务，所以玩比赛这边就少花了点时间，水平也不到位，本来在勉强拿到10%的最后一名，当剔除违规竞赛者之后，排在了10%之外。:(

很少有题目同时用到了GBDT,CV,NLP的，这次算是遇到一个，算是涨了见识，有人取胜重点靠抽取到商品图像的特征，有人取胜重点靠全量商品描述的词向量，有人取胜在于强大的ensemble各路模型各路特征。这一次我入场比较晚，找人组队就很难了，一个人打还是比较累的，虽然利用到了之前储存好的各种代码片段，不过还是时间太紧，精力有限，再加上我是球迷，我德表现如此糟糕，于是，哎哟喂，我也表现如此糟糕

# 自己的收获

短短10天，虽然成绩不佳，但是知识和技能上的收获还是感觉满满的：

* 组合模型在这一题发挥了重要的作用，我除了自己训练了xgb，lgb和catboost之外，还从讨论区拷贝来了rnn，ridge regression的成果
* 自己没怎么实践过自然语言处理，照猫画虎罗列好了tfidf，pca，svd等等，微调一下ngram，对自己分数提高还有有用的
* 认真按照自己的思路对着各路category实践了target encode，感觉用处不大，过拟合有点重了，后来在讨论中得知，大神们用的高阶的target encode还有加噪音加平滑，据说也有用，下次我也玩高级的TE
* 由于是第一次搞词向量的题目，第一次实践了稀疏矩阵，lgb和xgb都支持，catboost还不支持，不过用pca将稀疏矩阵做成少量几列还是可以喂给catboost用
* 拼接各种数据的时候，numpy的运算速度比dataframe要快，有的时候直接用pandas做数学运算，慢到出奇，不如把数据先导出到numpy，运算完再重填回dataframe，节约时间大于一个数量级
* 压缩数据在内存中占的体积，[link](https://www.kaggle.com/frankherfert/tips-tricks-for-working-with-large-datasets)这个参考很不错
* 多线程提取图像特征，参考[这里](https://www.kaggle.com/learnmower/ideas-for-image-features-and-multiproccessing/notebook)，让我差不多一天时间就把大约180万张图的基本特征都拿出来了。
* 在处理大量小图片的时候，固态硬盘比机械硬盘的优势就显示出来了，机械硬盘的IOPS实在是跟不上cpu gpu的推理速度，分分钟就到瓶颈，不过GCP上的几百GB ssd价格好贵啊，玩了一会儿做完图像就赶紧换回了机械硬盘。

# 高分答案集锦

这一次由于数据的丰富性，值得学习的思路很多，且很多大神分享的思路会有部分重合，这里只摘抄我认为最值得学习并且去重的部分思路

## 1st, Little Boat, [Dance with Ensemble](https://www.kaggle.com/c/avito-demand-prediction/discussion/59880)

有一些lgb，有一些nn，有一些xgb作为第一层，第二层还是一些lgb+nn+xgb,第三层就是一个nn了。不过这么组合出来分数提升了大约0.0002到0.0004，和直接线性组合比起来，可能差不多。

最佳单模型的nn和lgb都做到了215x的分数，另外队友加来一个大量商品描述+RNN做出来的特征，给lgb分数boost到了213x。

stacking在这一局非常重要，模型的多样化是取胜的关键，

#### 神经网络

我自己是重点时间都放NN了，所以也没怎么做特征工程。我就来分享一下如何用一个nn做到215x的吧。

 * 数字特征加上类别特征，分数有0.227x
 * 加上了title和description都用rnn训练一下，再加上fastText预训练的embedding，调整一下，分数到了0.221x
 * 自训练的fastText embedding，在全量数据上，分数到了0.220x
 * 加上vgg16的top layer的平均池化，这个让分数变差了。tuning一下，在合并文本、图像、类别、数字特征之前，分别加了一层，有了效果，0.219x了
 * 尝试调整文本模型，CNN或者attention等等，没有一个work。最后，两层的LSTM加一个dense，提高了分数0.0003
 * 给图像尝试不同的CNN，没有一个“fine tune”的模型有用，并且还花了很多时间。最后发现fixed ResNet50的中间层有用，提高了0.0005， 到了0.218x了
 * 开始各种tuning，都是基于自己想法的去调，发现在text和LSTM之间加上spatial dropout超级有用，提升了大约0.0007到0.001，又结合调整了dropout的比例，提升了0.001到0.0015，分数到了0.2165到0.217
 * 开始把队友做的特征都加进来，队友做的文字类的特征加进来都没啥用，不过其他类的特征都有用，最后一个NN做到0.215x
 * 如果一路上把所有模型都存下来（少量特征训练的模型，加了特征分数变差的模型），然后再训练一个全连接on top of those,可以得到0.008的提升，这个分数可以进入前10名了

秀一下我的模型架构图吧：
![img-0](01-littleboat.jpg)

#### 评论区摘抄

我在评论中看到他有说，keras的RNN模型，用CuDNN实现的来运行的话，会快很多。

然后他贴出了他的rnn代码：

```
seq_title_description = Input(shape=[max_seq_description_length], name="seq_description")
emb_seq_title_description = Embedding(vocab_size, EMBEDDING_DIM1, weights=[embedding_matrix1], trainable=False)(
    seq_title_description)
emb_seq_title_description = SpatialDropout1D(0.25)(emb_seq_title_description)
rnn_layer1 = CuDNNLSTM(128, return_sequences=True)(emb_seq_title_description)
rnn_layer1 = CuDNNLSTM(128, return_sequences=False)(rnn_layer1)
fc_rnn = Dense(256, init='he_normal')(rnn_layer1)
fc_rnn = PReLU()(fc_rnn)
fc_rnn = BatchNormalization()(fc_rnn)
fc_rnn = Dropout(0.25)(fc_rnn)
```

## 1st Arsenal
很早就和小船开始组队了，我们分别把精力花在nn和树模型。花大约一半的时间在自己的xgblgb，另外一半的时间合并四个人的特征共享。训练了6个base xgb和7个level 2 xgb，13个lgb和12个level2 lgb。最好的base lgb到了0.2138分，最好的level2 lgb到了0.2110分

#### 特征工程
* 文本特征：
	* tfidf用在title, description, title+description, title+description+param_1, etc.稀疏矩阵都直接给xgb用，然后做了svd和oof ridge特征给lgb用，来保持多样性。
	* 文本统计，例如单词长度，字母和单词在不同level的长度，unique数量
* 图像特征：
	* 一些基本的image meta info
	* 三个(ResNet50, InceptionV3, Xception)预训练模型的top 5 category.
	* 从vgg16拿出来的 512个raw特征，和first 15 PCA特征
	* keypoint数量特征
* 类别特征：
	* count/count unique在不同level的统计，都基于train+test算一遍，然后train+test+active再算一遍
	* 不同level的target encode,包括了mean deal_probability对于count大于5000的catetory，mean predicted
* 预测出来的独立变量特征
	* xgb预测的price，image_top_1,lgb的price，image_top_1,
	* 平均预测的price,image_top_1,item_seq_number,day_diff在不同的cat level上
* user_id特征
	* 玩到过拟合了
#### 其他
lgb我用了三组不同的的参数，用了baysian optimization找的，增加模型多样性嘛。

特征方面，我只用了能达成99%gain，在大部分lgb模型里留下了700多个特征，level 2 lgb里大约300多个特征，xgb大约用到了3000个稀疏特征

## 1st thousandvoices

* 类别特征的interactions
* 针对上述每一个特征，做TE, mean target of previous ads, total occurence count, median price
* 超参数调优，我每一次增加num_leaves都有用哎，一直增加到2014，再加下去我内存就不够了

## 1st Georgiy Danshchin

* validatioin方案：time rolling window比较稳，组队之后用common kfold了，
* predicted price 和log1(price) 的差距，是很有用的特征


## 2nd Sergei Fironov [kaggle link](https://www.kaggle.com/c/avito-demand-prediction/discussion/59871)

#### 特征

* 图像：从几个公共模型提取出向量来
* 文本：全量数据训练fasttext，生成向量（title, discription, title-category, stemmed title, stemmed description）
* 统计：各种groupby，interaction
* 非监督：用了autoencoder来对category提取向量，训练了一个user2vec来表示user_id，作为特征之一

#### 模型

* 神经网络的：最好的nn得分到了0.2163，用到的特征有：FM形式的cat特征的embedding；数字特征；拼接的fasttext；拼接图像特征；BiLSTM for words, BiLSTM for char......好多好多
* LGBM：fasttext向量对这个帮助很大，SVD over TFIDF有用，数字特征和target encode有用
* 比较弱的模型有：FM_FTRL, Ridge,CatBoost

#### 评论区摘抄

* 神经网络用了两种loss， ['mse','binary_crossentropy']，平均

## 3rd KazAnova [kaggle link](https://www.kaggle.com/c/avito-demand-prediction/discussion/59885)

#### 集成模型方案

平均了两组集成模型。第一组是用的标准的5 fold validation方案。

第二组是用的时序方案。测试数据日期开始的几天和训练数据最后的几天是重合的，我们尝试用这几天的数据用来做内部验证。所以，我们用训练集的前面六天来做训练，用第10到13天来做验证。没有用第6到9天的数据，这被用来模仿训练集和测试集的gap。最后，生成预测的时候，我们再用整个训练集。

为了保证category的相似性，我们一直用着gap4天的方式来做。比如计算day4的训练数据，我们用day0来做。为了模仿day 5的相似性，我们用day0和day1的平均值来做。这种方式，一直可以在cv和lb上给出类似的性能

我们最好单模型是用这个方法得来的，lgb最好0.2163，nn最好.2180

我们主要用了xentropy来作为objective，因为概率也是分布在0到1之间的

#### LGBM 特征

* 。。。。。。，自己去看吧，和上面差不多
* 3-way interactions of 分类特征

#### NN

我们最好的nn用了两stacked的 双向的 GRU, 额，后面的描述我不知道咋翻译了，对NLP不熟，以后慢慢补。objective用的是binary cross-entropy with sigmoid output.

几个注意的关键点：

* 用vgg16和vgg19获得特征，当做dense input
* 文本基于nltk处理
* 关于文本，我们合并了description,title, params到一个域，然后用间隔符分开

后面还有很多分享，太多了，这里就不写了。

## 4th Joe Eddy

价格有关的统计特征：因为发现价格比较重要，于是对价格开始基于各种分类的聚合。然后比如有20th percentile, median, max, std, skew。然后计算相对价格，比如每一个价格相对于同样类别均价有多少便宜或者多少贵。后来猜测到购物网站上，有基于价格来排序商品的选项，这个肯定是很重要的特征了。然后做了价格关于文字类描述的聚合，文字类描述做了词向量然后聚类成少量的类别。

#### 评论

```
Q: 
Why did you choose tf-idf and svd as the matrix factorization method of categorical data? Coming from talkingdata, my first instinct was to use LDA(talkingdata 1st place solution) on label-encoded categories.
A:
I chose tf-idf -> SVD because it's the classic approach used for latent semantic analysis on text / the easiest one for me to implement and tune, but it's very possible that LDA would have worked well for this too. 
```


# 最后

小伙子还是太年轻，要想取得好成绩，一定是早早就准备开始比赛，审题，分析，多多在论坛参与讨论，你看，夺得前三名的队伍，都是很早开始，一个个思路的尝试，每一步都踏平一个坑走到冠军的位置的

### 失败原因概括

* 尽早准备，才有时间机会尝试各种idea，各种玩法
* NLP实践确是我的弱点，需要找一门课复习一遍，再找一道老的题目实践实践
* layer stacking还没搞过，以后一定找机会试试
* 提交没几次发现组合模型特别有效的话，组队刷才是王道




