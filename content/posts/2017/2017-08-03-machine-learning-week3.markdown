---
Title: coursera Andrew Ng 机器学习第三周笔记 逻辑回归与泛化
author: Young
date: 2017-08-03
tags: [ai, machine learning, Andrew Ng, Logistic Regression]
Status: public
---
## Logistic Regression

#### Classification

The classification problem is just like the regression problem, except that the values y we now want to predict take on only a small number of discrete values. For now, we will focus on the **binary classification problem** in which y can take on only two values, 0 and 1.

#### Logistic Regression Model

![](./2017-08-03/sigmoid-1.png)

![](./2017-08-03/sigmoid-2.png)

![](./2017-08-03/sigmoid-3.png)

#### Decision Boundary

The **decision boundary** is the line that separates the area where y = 0 and where y = 1. It is created by our hypothesis function.

#### Logistic Regression Cost Function

We cannot use the same cost function that we use for linear regression because the Logistic Function will cause the output to be wavy, causing many local optima. In other words, it will not be a convex function.

**我们不可以用和线性回归一样的代价函数了，因为如果使用了同样的代价函数，那么这个函数会波动起伏，有很多局部最优解，简而言之，它就不是凸函数了，不适合用来当做代价函数了**

![](./2017-08-03/cost-function-1.png)

![](./2017-08-03/cost-function-2.png)

![](./2017-08-03/cost-function-3.png)

If our correct answer 'y' is 0, then the cost function will be 0 if our hypothesis function also outputs 0. If our hypothesis approaches 1, then the cost function will approach infinity.

#### simplified cost function and Gradient Descent

![](./2017-08-03/cost-full-1.png)

**向量化的代价函数**

![](./2017-08-03/cost-full-2.png)

![](./2017-08-03/gradient-1.png)

**向量化的梯度下降**

![](./2017-08-03/gradient-2.png)

#### Advanced Optimization,高级优化算法

"Conjugate gradient", "BFGS", and "L-BFGS" are more sophisticated, faster ways to optimize θ that can be used instead of gradient descent. We suggest that you should not write these more sophisticated algorithms yourself (unless you are an expert in numerical computing) but use the libraries instead, as they're already tested and highly optimized.

![](./2017-08-03/optimized.png)

例如matlab里的fminunc()，一般来说都是把自己写好的代价函数，初始化的theta，等等参数传入

#### multicalss calssification

one vs all = N * one vs rest

one vs rest = logistic regression classifier

## Regularization

#### problem of overfitting

* Reduce the number of features
 * Manually select which features to keep
 * use a model selection algorithm

* Regularization
 * keep all the features, but reduce the magnitude of parameters theta-j
 * Regularization works well when we have a lot of slightly useful features.

#### with lambda

The λ, or lambda, is the regularization parameter. It determines how much the costs of our theta parameters are inflated.

cost function with lambda

![](./2017-08-03/lambda-cost.png)

gradient descent with lambda

![](./2017-08-03/lambda-gradient.png)

