---
Title: xgboost模型调试攻略
author: Young
date: 2017-10-23
tags: [machine learning, xgboost, tune, parameters]
Status: public
---
最近做题的感受，结合网上看来的资料，总结如下：

## xgboost python api 描述

* xgboost.DMatrix, 是个类，存的是数据矩阵，自带内存优化，训练加速，可以从numpy.arrays初始化
* xgboost.Booster，是个类，模型所在地，包括了较低层次的routines for training,prediction and evalution
* xgboost.train，是个函数，给定参数来训练booster，返回一个训练完成的booster
* xgboost.cv，是个函数，给定参数做交叉验证，返回评估历史，list(string)
* scikit-learn包装的api, 
	* xgboost.XGBRegressor, 
	* xgboost.XGBClassifier
* 画图有关的api, 
	* xgboost.plot_importance, 根据fitted trees画出重要性，返回的是matplotlib Axes
	* xgboost.plot_tree，画出指定的树
	* xgboost.to_graphviz,将指定的树转化成graphviz实例

## about parameter tune

* 选用一个相对高一点的learning rate例如0.1，一般选用0.05到0.3之间对于大部分问题都可以了。定了学习率，然后决定合适数量的树，用cv
* 调整树的特定参数，比如max_depth,min_child_weight,gamma,subsample,colsample_bytree
* 调整正则化参数例如lambda和alpha，可以降低模型复杂度和提高性能
* 降低learning rate，再继续优化参数们

## tune step by step

##### 第一步，确定learning rate和estimator的数量
  
 不过我们需要初始化一些基本变量，例如：
 
 * max_depth=5，一般用3到10，456都是不错的starting points
 * min_child_weight=1，选择较小的值是因为数据分类很不对称并且叶子节点有可能具有较小的group size
 * gamma=0，选择0.1或者0.2都可以，稍后一定需要调试
 * subsample colsample_bytree=0.8, 典型的选择在0.5到0.9之间
 * scale_pos_weight=1, 数据类别高度不平衡

 learning rate就先用0.1，先用cv来寻找最优的estimators 
 
##### 第二步，max_depth和 min_child_weight

  调整着两个是因为它们对模型结果有很大的影响，开始的测试用的范围广泛一点，以后再用较小的范围来确定具体选择为多少。for example: max_depth(3,10,2) min_child_weight(1,6,2)

##### 第三步，gamma
  check from 0.1 to 0.5
  
##### 第四步，subsample, colsample_bytree
  both check from 0.6 to 0.9
  
##### 第五步，正则化参数
  reg_alpha try 1e-5, 1e-2, 0.1, 1, 100
  reg_lambda try 
  
##### 第六步，降低学习率
降低学习率的同时增加树的数量

##### last tips
It is difficult to get a very big leap in performance by just using parameter tuning or slightly better models. The max score for GBM was 0.8487 while XGBoost gave 0.8494. This is a decent improvement but not something very substantial.

**A significant jump can be obtained by other methods like feature engineering, creating ensemble of models, stacking**

神马？评论区看到的，该文章的作者说：For instance, I generally do some parameter tuning and then run 10 different models on same parameters but different seeds. Averaging their results generally gives a good boost to the performance of the model. 

## overfitting process

第一是考虑调整模型的复杂度，max_depth, min_child_weight, gamma

第二是在训练过程中增加随机性，抗噪音，比如subsample,solsample_bytree

## 处理不均衡的数据集

对于不平衡的数据集，例如鼠标点击通过的日志，肯定是极其不平衡的，这对xgb肯定有影响，有两种方法来提高

第一种，如果你在意auc，用auc来做eval，通过scale_pos_weight来平衡正面样本和反面样本的权重

第二种，如果你在意概率，你不可以重新平衡数据集，应该设置max_delta_step为一个有限数字来帮助收敛



## 主要参考的链接

* 官网描述的参数列表:[http://xgboost.readthedocs.io/en/latest/parameter.html#general-parameters](http://xgboost.readthedocs.io/en/latest/parameter.html#general-parameters)

* 官网python api描述:[https://xgboost.readthedocs.io/en/latest/python/python_api.html](https://xgboost.readthedocs.io/en/latest/python/python_api.html)

* 官网描述的过拟合处理:[http://xgboost.readthedocs.io/en/latest/how_to/param_tuning.html#control-overfitting](http://xgboost.readthedocs.io/en/latest/how_to/param_tuning.html#control-overfitting)

* google搜索的第一篇文章:[https://www.analyticsvidhya.com/blog/2016/03/complete-guide-parameter-tuning-xgboost-with-codes-python/](https://www.analyticsvidhya.com/blog/2016/03/complete-guide-parameter-tuning-xgboost-with-codes-python/)
