---
Title: kaggle | Machine Learning for Insights Challenge
author: Young
date: 2018-09-25
tags: [kaggle, machine learning, insights, explanation]
Status: public
---
# 前言

最近比较忙，这次不是比赛分享了，是来分享一个“挑战”，kaggle上用词说是Challenge，在我理解这个是某专家管理员建立了一个简短的course，介绍了某主题的玩法，然后让大家做做简单的练习，一起讨论交流一下。

这次学习的是一个关于模型洞察力的主题，原文链接[https://www.kaggle.com/ml-for-insights-signup](https://www.kaggle.com/ml-for-insights-signup)， 本文这里主要是将我看到的有用的内容按照自己的理解翻译，记录下来。这个ML for insights，字面直译为模型洞察力了，在我理解来则是模型可解释性分析。

## My  Insights

我个人在玩kaggle和工作中用到最多的主要是树类模型(lgb,xgb)和神经网络(cnn, rnn)，确实很少去思考其中的含义和解释性，如果让我自己回答这个问题，我理解到用决策树的信息熵计算统计概率得出叶子节点的重要性，再加上拟合残差的思路就是xgb类的算法了。而神经网络方面，我则简单的理解为求导拟合。高中课本里的一次函数二次函数的求导大家都会，神经网络只是用链式法则给若干个矩阵求导罢了，思路还是朝着目标去拟合。

如此之粗浅，见笑了，下面看看别人的insights玩法。

# User Case

## 一个模型的哪些东西是可解释的

许多人认为机器学习模型是黑盒子，在他们认为模型可以做出很好的预测，但是大家无法理解这些预测背后的逻辑。确实是，很多数据科学家不知道如何用模型来解释数据的实际意义。这里的文章将会从这么几个方面来讨论：

* 哪些特征在模型看到是最重要的？
* 关于某一条记录的预测，每一个特征是如何影响到最终的预测结果的？
* 从大量的记录整体来考虑，每一个特征如何影响模型的预测的？

## 为什么这些解释信息是有价值的

* 调试模型用
* 指导工程师做特征工程
* 指导数据采集的方向
* 指导人们做决策
* 建立模型和人之间的信任

### 调试模型用

一般的真实业务场景会有很多不可信赖的，没有组织好的脏数据。你在预处理数据时就有可能加进来了潜在的错误，或者不小心泄露了预测目标的信息等，考虑各种潜在的灾难性后果，debug的思路就尤其重要了。当你遇到了用现有业务知识无法解释的数据的时候，了解模型预测的模式，可以帮助你快速定位问题，balabala

### 指导特征工程

特征工程通常是提升模型准确率最有效的方法。特征工程通常涉及到到反复的操作原始数据(或者之前的简单特征)，用不同的方法来得到新的特征。

有时候你完成FE的过程只用到了自己的直觉。这其实还不够，当你有上百个原始特征的时候，或者当你缺乏业务背景知识的时候，你将会需要更多的指导方向。

这个[预测贷款结果](https://www.kaggle.com/c/loan-default-prediction)的kaggle竞赛就是一个比较极端的例子，这个比赛有上百个原始特征。并且因为隐私原因，特征的名称都是```f1, f2, f3```等等而不是普通的英文单词来描述。这就模拟了一个场景，你没有任何业务方面直觉的场景。有一位参赛者发现了某两个特征相减```f527, f528```可以创建出特别有用的新特征。拥有这个新特征的模型比没有这个特征的模型优秀很多。但是当你面对几百个特征时，你如何创造出这样优秀的特征呢？

在这门课程里，你将会学习到找到最重要的特征的方法，并且可以发现两个特别相关的特征，当面对越来越多的特征的时候，这种方法就会很重要啦。

### 指导未来数据收集

对于网上下载的数据集你完全控制不了。不过很多公司和机构用数据科学来指导他们从更多方面收集数据。一般来说，收集新数据很可能花费比较高或者不是很容易，所以大家很想要知道哪些数据是值得收集的。基于模型的洞察力分析可以教你很好的理解已有的特征，这将会帮助你推断什么样子的新特征是有用的

### 指导人们决策

一些决策是模型自动做出来的，虽然亚马逊不会用人工来决定展示给你网页上的商品，但是很多重要的决策是由人来做出的，而对于这些决定，模型的洞察力会比模型的预测结果更有价值。

### 建立信任

很多人在做重要决策的时候不会轻易的相信模型，除非他们验证过模型的一些基本特性，这当然是合理的。实际上，把模型的可解释性展示出来，如果可以匹配上人们对问题的理解，那么这将会建立起大家对模型的信任，即使是在那些没有数据科学知识的人群中。


# Permuation Importance

一个最基本的问题大概会是什么特征对我模型预测的影响最大呢？ 这个东西就叫做“feature importance”即特征重要性。anyway，字面意思看这个东东就很重要啦。我们有很多方法来衡量特征的重要性，这里呢，将会介绍一种方法：排列重要性。这种方法和其他方法比起来，优势有：

 * 计算速度快
 * 广泛使用和理解
 * Consistent with properties we would want a feature importance measure to have

## 工作原理

排列重要性，一定是在model训练完成后，才可以计算的。简单来说，就是改变数据表格中某一列的数据的排列，看其对预测准确性的影响有多大。大概三个步骤：

* 训练好模型
* 拿某一个feature column, 然后随机打乱顺序。然后用模型来重新预测一遍，看看自己的metric或者loss function变化了多少
* 把上一个步骤中打乱的column复原，换下一个column重复上一个步骤，直到所有column都算一遍

## 代码示例

这个case是利用FIFA 18很多场的足球比赛的数据统计，来预测"Winner of The Game"

代码片段1，这个算是把模型训练完毕

```
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier

data = pd.read_csv('../input/fifa-2018-match-statistics/FIFA 2018 Statistics.csv')
y = (data['Man of the Match'] == "Yes")  # Convert from string "Yes"/"No" to binary
feature_names = [i for i in data.columns if data[i].dtype in [np.int64]]
X = data[feature_names]
train_X, val_X, train_y, val_y = train_test_split(X, y, random_state=1)
my_model = RandomForestClassifier(random_state=0).fit(train_X, train_y)
```

代码片段2，用eli5库，计算排列重要性

```
import eli5
from eli5.sklearn import PermutationImportance

perm = PermutationImportance(my_model, random_state=1).fit(val_X, val_y)
eli5.show_weights(perm, feature_names = val_X.columns.tolist())
```

![p1](permutation1.png)

## 解读permutation importance

在上图中，顶部的是对结果影响最大的，底部的是影响最小的。因为是随机的打乱排列计算出来的，所以多做了几次这个操作，对metric的影响大小，用的是mean加减std这两个数字来表示的。

偶尔会看到重要性的值是负数，这意味着那一列特征被打乱后比打乱前的预测准确率还要高。这意味着这列数据基本没啥用。在较小的数据集里会更容易出现这种情况。

这里例子里，最重要的特征是“Goals scored”, 这看起来是合理的。足球运动员对这些特征的重要性的顺序会有自己的见解。

# Partial Dependence Plots

特征重要性可以告诉你哪些特征是最重要的或者是不重要的，而partial dependence图可以告诉你一个特征是如何影响预测的。

这个图特别适合用来回答类似这样的问题：

 * 在所有的房屋的特征中，经度和纬度是如何影响房价的？或者说，在不同的地区同样面积的房屋价格有多少相似呢？
 * 预测人们健康状况时，饮食结构的不同会带来多大的影响？还是有其他更重要的影响因素？

如果你对线性回归或者逻辑回归比较熟悉，那么partial dependence可以被类比为这两类模型中的“系数”。并且partial dependence在复杂模型中的作用比在简单模型中更大，抓出更复杂的特性。 

## 工作原理

和permutation importance一样，partial dependence plots是需要在模型训练完毕后才能计算出来。

同样还是用FIFA2018的数据集，不同的球队在各个方面都是不一样的。比如传球数，射门数，射进数等等。一眼看过去，很难区分这些特征对结果的影响有多大。为了清晰的分析，我们还是先只拿出某一行数据，比如说这一行数据里，有控球时间50%，100次传球，10次射门和一个进球。我们将会用已有模型来预测结果，将这一行的某一个变量，反复的进行修改和重新预测，比如将控球时间修改从50%修改为60%，等等。持续观察预测结果，在不同的控球率时有什么样的变化。

这里的例子，只用到了一行数据。特征之间的相互作用关系通过这一行来观察可能不太妥当，那么考虑用多行数据来进行试验，然后根据平均值画出图像来。

## 代码示例

代码片段3，还是先训练好一个决策树模型

```
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.tree import DecisionTreeClassifier

data = pd.read_csv('../input/fifa-2018-match-statistics/FIFA 2018 Statistics.csv')
y = (data['Man of the Match'] == "Yes")  # Convert from string "Yes"/"No" to binary
feature_names = [i for i in data.columns if data[i].dtype in [np.int64]]
X = data[feature_names]
train_X, val_X, train_y, val_y = train_test_split(X, y, random_state=1)
tree_model = DecisionTreeClassifier(random_state=0, max_depth=5, min_samples_split=5).fit(train_X, train_y)
```

一般是为了解释起来比较方便，所以这里用的是决策树，自己实际玩的时候可以用更复杂的模型。

代码片段4，决策树结构

```
from sklearn import tree
import graphviz

tree_graph = tree.export_graphviz(tree_model, out_file=None, feature_names=feature_names)
graphviz.Source(tree_graph)
```

![p2](decisiontree1.png)

这个图大意如下：1. 拥有child节点的节点里的信息，第一行标记的是如何决策拆分的；2.每个节点下面几行的数据，都是表示该节点上的已经确定的value

代码片段5，用[PDPBox](https://pdpbox.readthedocs.io/en/latest/)库来画出Partial Dependence Plot

```
from matplotlib import pyplot as plt
from pdpbox import pdp, get_dataset, info_plots

# 创建好画图所需的数据
pdp_goals = pdp.pdp_isolate(model=tree_model, dataset=val_X, model_features=feature_names, feature='Goal Scored')

# 画出“Goal Scored”这一特征的partial dependence plot
pdp.pdp_plot(pdp_goals, 'Goal Scored')
plt.show()
```

![p3](pdp1.png)

在解读这个图的时候，有几点是值得注意的：第一， y轴是预测结果的变化量，第二，蓝色阴影区域代表了信心的大小。从这幅图可以看出，进球数的增加肯定可以增加赢得MVP的概率，但是进更多的球的时候，对这个概率影响不大了。

代码片段6，再来看另外一个特征：

```
feature_to_plot = 'Distance Covered (Kms)'
pdp_dist = pdp.pdp_isolate(model=tree_model, dataset=val_X, model_features=feature_names, feature=feature_to_plot)

pdp.pdp_plot(pdp_dist, feature_to_plot)
plt.show()
```

![p3](pdp2.png)

这幅图看起来更单调一点。因为模型是用的比较简单。你可以从决策树的图中看出这个图和决策树上的数据一一匹配的样子。(咦？这个是咋看出来的？我咋没看出来)

代码片段7，如果换一个模型，用随机森林的话，图会略有不同

```
# Build Random Forest model
rf_model = RandomForestClassifier(random_state=0).fit(train_X, train_y)

pdp_dist = pdp.pdp_isolate(model=rf_model, dataset=val_X, model_features=feature_names, feature=feature_to_plot)

pdp.pdp_plot(pdp_dist, feature_to_plot)
plt.show()
```

![p4](pdp3.png)

随机森林认为一个球队差不多跑动距离大约100KM左右是最容易拥有MVP球员的，跑动距离多了概率会下降一点，符合逻辑，毕竟跑动多了会疲劳而影响发挥。

总的来说，这个曲线的平滑程度看上去比决策树更加合理一点。在处理小数据集的时候，解读模型要十分小心。

## 2D Partial Dependence Plots

如果你对特征之间的相互关系感兴趣的话，那么2D PDP图肯定用得上了。

代码片段8，还是用决策树模型，来创建一个简单的图，看看是否匹配之前的树结构。

```
# Similar to previous PDP plot except we use pdp_interact instead of pdp_isolate and pdp_interact_plot instead of pdp_isolate_plot
features_to_plot = ['Goal Scored', 'Distance Covered (Kms)']
inter1  =  pdp.pdp_interact(model=tree_model, dataset=val_X, model_features=feature_names, features=features_to_plot)

pdp.pdp_interact_plot(pdp_interact_out=inter1, feature_names=features_to_plot, plot_type='contour')
plt.show()
```

![p5](pdp4.png)

上图展示了进球数和跑动距离的各种组合情况下的预测结果。例如，我们看到了最高的预测值来自于当一个球队进了一个球并且跑动距离接近于100km的时候。当进球数为0的时候，跑动距离无论多少都没啥关系了。你能不能从决策树的图中分析出当进球数为0时的这种情况呢？

这个图能清晰的显示出，跑动距离在有进球的时候才会影响预测结果。

# SHAP Values

前面已经介绍了两种技术，现在将要介绍的技术可以让你观察到某一个item预测中各个特征对预测结果产生的影响：SHAP(是SHapley Additive exPlanations的缩写) values。这个技术应用场景可以是：1. 一个模型说银行不可以贷款给你，但是在法律上银行需要基本的解释为什么拒绝了你的贷款申请；2. 一个医疗保健中心想要知道在某一类疾病患者中，每一个患者的最大的病因是什么，然后医疗中心可以对症下药

## How They Work

NIPS 论文地址：[A Unified Approach to Interpreting Model Predictions](http://papers.nips.cc/paper/7062-a-unified-approach-to-interpreting-model-predictions)，也可以参考这篇博客[One Feature Attribution Method to (Supposedly) Rule Them All: Shapley Values](https://towardsdatascience.com/one-feature-attribution-method-to-supposedly-rule-them-all-shapley-values-f3e04534983d)，这个计算SHAP值的想法来自于博弈论中的shapley value(shapley单词源自于2012年诺贝尔经济学奖获得者Lloyd Stowell Shapley)。膜拜了一圈，很深奥的样子，我理解的大意就是：计算一个特征加入到模型时的边际贡献，然后考虑到该特征在所有的特征序列的情况下不同的边际贡献，取均值，即某该特征的SHAP baseline value。

这里还是用FIFA 18足球比赛的数据集来举例，SHAP值的计算原理如下：

```
sum(SHAP values for all features) = pred_for_team - pred_for_baseline_values
```

他有一个baseline value，然后呢，他给出所有特征基于这个baseline的值是多少，当然也会有最后预测值是多少。可以看到如下图所示：
![p6](shap0.png)
如何解读这张图呢，我们预测值是0.7，基准值是0.4979，增加预测值的特征颜色是红色，长条的面积代表了数值的大小。降低预测值的特征颜色为蓝色。可以看到最大的增加预测值的特征是“进球数”为2。但是我们可以看到控球率这个还比较有意义的特征在这里是降低了预测值。

它用到了复杂的技术来保证了那个公式，baseline值加上各个特征的影响值等于最后的预测值（听起来不是那么直观到底是为什么）。这里我们不继续探讨细节，这对于使用SHAP values不是critical的。偏理论一点的解释文章，参考这个[博客链接](https://towardsdatascience.com/one-feature-attribution-method-to-supposedly-rule-them-all-shapley-values-f3e04534983d)。

## 代码示例

这里我们用到的库是宇宙第一无与伦比的[Shap(github link)](https://github.com/slundberg/shap)，快去给作者们点个赞。

代码片段9：还是先训练好一个随机森林

```
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier

data = pd.read_csv('../input/fifa-2018-match-statistics/FIFA 2018 Statistics.csv')
y = (data['Man of the Match'] == "Yes")  # Convert from string "Yes"/"No" to binary
feature_names = [i for i in data.columns if data[i].dtype in [np.int64, np.int64]]
X = data[feature_names]
train_X, val_X, train_y, val_y = train_test_split(X, y, random_state=1)
my_model = RandomForestClassifier(random_state=0).fit(train_X, train_y)
```

代码片段10：计算出单次预测的SHAP值

```
import shap  # package used to calculate Shap values

row_to_show = 5
data_for_prediction = val_X.iloc[row_to_show]  # use 1 row of data here. Could use multiple rows if desired
data_for_prediction_array = data_for_prediction.values.reshape(1, -1)

# Create object that can calculate shap values
explainer = shap.TreeExplainer(my_model)

# Calculate Shap values
shap_values = explainer.shap_values(data_for_prediction)
```

上面计算出来的shap_values里有两个数组。第一个数组里存放的是给预测值负面影响的数字，第二个数组里存着正面影响的数值。直接查看两个数组的数字可能会有点蠢，所以shap library上场来一个漂亮的可视化。可视化之前，我们先看一预测结果，```my_model.predict_proba(data_for_prediction_array)```的输出是```array([[0.3, 0.7]])```，可以看到这个队伍有70%的概率有成员可以获得最终的MVP。

代码片段11：shap可视化

```
shap.initjs()
shap.force_plot(explainer.expected_value[1], shap_values[1], data_for_prediction)
```

![p7](shap0.png)
如果你仔细观察我们创建SHAP values时的代码，你回注意到我们用的是一个TreeExplainer。shap库其实对其他的模型也有对应的解释器(explainers):
 * shap.DeepExplainer 是用于解释深度学习模型的
 * shap.KernelExplainer 可用于解释所有的模型，尽管它运行时间比其他解释器时间长一点，但是它能够给出一个近似的的Shap values.

代码片段12：KernelExplainer的使用，可以和treeExplainer给出同样的结果

```
# use Kernel SHAP to explain test set predictions
k_explainer = shap.KernelExplainer(my_model.predict_proba, train_X)
k_shap_values = k_explainer.shap_values(data_for_prediction)
shap.force_plot(explainer.expected_value[1], shap_values[1], data_for_prediction)
```

![p8](shap0.png)


# SHAP Value的进阶玩法


## 前情提要

Shap values可以告诉我们一个特征对我们的一条预测结果产生了多大的影响。

[Shap library](https://github.com/slundberg/shap) 是个不错的工具，可以告诉我们具体的shap values是多少并且有不错的可视化来展现

## Summary Plots

Permutation importance很不错，因为它用很简单的数字就可以衡量特征对模型的重要性。但是它不能handle这么一种情况：当一个feature有中等的permutation importance的时候，这可能意味着这么两种情况：1：对少量的预测有很大的影响，但是整体来说影响较小；2：对所有的预测都有中等程度的影响。

这个时候，SHAP summary plot上场了，它可以对特征重要性来一个鸟瞰图，看是FIFA足球的数据集，如下图：

![p9](summaryplot0.png)

这个图上有很多点，对吧，每一个点都有三个含义：

 * 竖直坐标是说明它属于哪个特征
 * 颜色代表了这个特征在某一行数据里的数值是高还是低
 * 水平位置代表了这个特征在某一行数据里是提高预测值还是降低预测值

比如左上角的那个蓝点的意思就是，进球数特别少，给预测值降低了0.25。
其他几个很容易观察出来的方面包括：

 * 这个模型几乎忽略了**Red and Yellow**和**Red**这两个特征
 * **Yellow Card**这个特征，通常不会有大的影响，但是也有几个极端的例子大幅降低了预测值
 * **Goal Scored**这个特征，大体上和预测值有正相关性

如果你认真观察这个图，可以抓到更多有效信息。

## 代码示例: Summary Plots

代码片段13，建立随机深林模型

```
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier

data = pd.read_csv('../input/fifa-2018-match-statistics/FIFA 2018 Statistics.csv')
y = (data['Man of the Match'] == "Yes")  # Convert from string "Yes"/"No" to binary
feature_names = [i for i in data.columns if data[i].dtype in [np.int64, np.int64]]
X = data[feature_names]
train_X, val_X, train_y, val_y = train_test_split(X, y, random_state=1)
my_model = RandomForestClassifier(random_state=0).fit(train_X, train_y)
```

代码片段14，画图summary plots

```python
import shap  # package used to calculate Shap values

# Create object that can calculate shap values
explainer = shap.TreeExplainer(my_model)

# calculate shap values. This is what we will plot.
# Calculate shap_values for all of val_X rather than a single row, to have more data for plot.
shap_values = explainer.shap_values(val_X)

# Make plot. Index of [1] is explained in text below.
shap.summary_plot(shap_values[1], val_X)
```

![p9](summaryplot0.png)

这段代码不是很复杂，但是有几点需要注意的:
 * 当画图的时候，我们用变量`shap_values[1]`，如果是分类问题，那么这变量是一个array，每一个元素里都是某一个类别的SHAP values。在这个例子里，这里是存放的为True的概率
 * 计算SHAP values可能会比较慢。在这里不会是大问题，因为这个数据集比较小。不过将来你用的时候得小心使用的数据集不要太大。不过有一个例外：当使用xgboost的时候，SHAP会有一些优化来加速。

上面这个图给了一个不错的overview，但是想要深入了解某一个特征的话，看下面的dependence contributions plots吧。

## SHAP Dependence Contribution Plots

之前我们用Partial Dependence Plots来展示过某一个特征对模型的影响。那是非常有用的洞察力工具，并且还可以解释给非技术类的朋友听。

但是，有很多信息PDP无法展示。例如，某一特征的影响分布是怎样的，这种影响是一个常量，还是大多依赖于其他特征的值。SHAP dependence contribuition plots可以画出一种类似PDP的图，但是有更多的细节。

![p10](shapDCP0.png)

我们先关注形状，之后再来看颜色。图中的每一个点都代表了一行数据。横坐标是控球率(直接来自特征Ball Posssession)，纵坐标表示的是这个特征的这个值对最后的预测结果有什么影响。从大体图像上的斜坡形状可以看出控球率越高就会给预测值带来越多正效益。

我们再来看下图中标记出来的两个点，有几乎相同的控球率，但是SHAP value却差距很大。这里看不出有什么不同，觉着吧，如果和一起其他的重要特征一起看看，估计能发现什么。

![p11](shapDCP1.png)

而下面这个图中，可以看到，尽管控球率很高，但是由于进球数不够，所以也是预测值会降低，说明当一个球队长期持球而不进球，那么也是属于表现不好。

![p12](shapDCP2.png)


## 代码示例：SHAP Dependence Contribution Plots

代码片段15，画图SHAP DCP

```
import shap  # package used to calculate Shap values

# Create object that can calculate shap values
explainer = shap.TreeExplainer(my_model)

# calculate shap values. This is what we will plot.
shap_values = explainer.shap_values(X)

# make plot.
shap.dependence_plot('Ball Possession %', shap_values[1], X, interaction_index="Goal Scored")
```

![p10](shapDCP0.png)

如果`interaction_index`那里不指定参数的话，Shapley会根据算法自动选择一个比较有趣的来。几行代码就可以画好这个图，更重要的是洞察出有意义的信息。


# End

总结一下，这里介绍了三种方法Permutation Importance，Partial Dependence Plots，SHAP Values，分别都有各自的python库可以调用，以达到洞察数据的目的，希望大家看完之后能有所收获。

如果文中有描述错误的地方请多多包涵，给我留言联系我改正过来。

有任何机器学习相关的知识都欢迎探讨。