---
Title: coursera Andrew Ng 机器学习 编程习题总结
author: Young
date: 2017-08-12
tags: [ai, machine learning, Andrew Ng, coursera]
Status: public
---
学习这些知识不是为了学习而学习，是为了解决问题而学习。所以这些知识所配套的编程练习尤为重要了。这门课程做的作业，可算是让我开了眼界，简单的一些代码和并不是特别大量的数据，就能解决一些简单的问题。受益匪浅！记录下来这些，以供之后解决复杂问题的时候给一个参考思路。

这门课程的代码我是用的matlab，我的代码参考链接[点击这里](https://github.com/yyq/coursera-machine-learning)

### Linear Regression

* 解决预测房价的问题，已知面积，房间数
* 写出线性回归的代价函数和梯度下降函数
* 画图来看，有个直观感受
* 不同的learning rate有不同的影响，收敛快慢

### Logistic Regression

* 预测某个学生是否通过了某门课程，已知数据包括两次测验的成绩，画出直线决策边界
* 预测制造工厂出产的芯片的质量合格与否，根据芯片的两次测试成绩，画出像圆形的决策边界

* sigmoid函数
* 代价函数，梯度下降函数
* 范化

### Multi-class classification and Neural Network

* 数字图片的识别，输入图片，输出数字
* 通过1-vs-N的逻辑回归函数来识别
* 通过神经网络来识别(这里只实现了前向反馈用来预测)，theta矩阵已知，效果显然比逻辑回归好


### Neural Networks Learning

* 仍然是识别数字
* 实现了神经网络里的后向反馈，代价函数，梯度下降，随机初始化，泛化，梯度检查等步骤
* 可视化隐藏层，有个直观感受

### Regularized Linear Regression and Bias v.s. Variance

* 不同的偏差，方差，通过画图观察
* 画学习曲线，发现了高偏差问题，数据量增多也不管用，那么线性改多项式
* 多项式回归，画图，调整选择lambda

### Support Vector Machines

* 识别垃圾邮件，亲测自己收到的各种广告垃圾邮件，果然有效！
* Informally, the C parameter is a positive value that controls the penalty for misclassified training examples
* SVM with 高斯内核
* 决策边界，无比扭曲的曲线都可以适应好
* 垃圾邮件case:邮件单词预处理，词库预处理，提取邮件特征，训练，预测，GoodJob!

### K-means Clustering and Princiapl Component Analysis

* K-means聚类用于压缩图像，通过减少颜色的数量
* PCA用于将脸部图片的降维，将32 * 32的图像压缩为10 * 10，体积减少很多（10X），运算加快很多，当需要运用神经网络+人脸识别的时候，用处就大了。

### Anomaly Detection and Recommender Systems

* 如题
* 推荐系统还是教程里的电影打分的问题