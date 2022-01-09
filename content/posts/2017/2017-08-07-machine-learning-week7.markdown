---
Title: coursera Andrew Ng 机器学习第七周笔记 支持向量机
author: Young
date: 2017-08-07
tags: [svm, ai, machine learning, Andrew Ng, coursera]
Status: public
---
## Support Vector Machines

#### optimization Objective

![](./2017-08-07/optimize-objective-0.png)

![](./2017-08-07/optimize-objective.png)

#### Large Margin Intuition

![](./2017-08-07/margin.png)

#### Kernels

![](./2017-08-07/kernel.png)

![](./2017-08-07/parameter.png)

#### Using an SVM

**choice of parameter C**

**choice of kernel(similarity function)**

if Gaussian Kernel, need to choose sigma^2

Not all similarity functions make valid kernels(need to satisfy technical condition called "Mercer's Theorem" to make sure SVM packages' optimizations run correctly, and do not diverge).

Polynomial kernel:
More esoteric: String kernel, chi-square kernel, histogram intersection kernel

**multi-class calssification**

K SVMs, one to distinguish one from the rest. 

![](./2017-08-07/using-svm.png)
