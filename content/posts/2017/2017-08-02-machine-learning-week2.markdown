---
Title: coursera Andrew Ng 机器学习第二周笔记 多个变量的线性回归
author: Young
date: 2017-08-02
tags: [ai, machine learning, Andrew Ng]
Status: public
---
## Linear Regression with Multiple Variables

#### Multiple Features

![](./2017-08-02/multi-feature.png)

![](./2017-08-02/multi-feature-1.png)

#### Gradient Descent for Multiple Variables

![](./2017-08-02/gradient-for-m.png)

![](./2017-08-02/gradient-for-m-1.png)

#### Feature Scaling

We can speed up gradient descent by having each of our input values in roughly the same range.

* **feature scaling**: involves dividing the input values by the range of the input variable resulting in a new range of just 1.
* **mean normalization**: involves subtracting the average value for an input variable from the values for that input variable resulting in a new average value for the input variable of just zero.

![](./2017-08-02/feature-scale.png)

ui是平均值，si是最大-最小或者是标准差

#### Learning Rate

* **Debugging gradient descent**: Make a plot with number of iterations on the x-axis. Now plot the cost function, J(θ) over the number of iterations of gradient descent. If J(θ) ever increases, then you probably need to decrease α.
* **Automatic convergence test**: Declare convergence if J(θ) decreases by less than E in one iteration, where E is some small value such as 10−3. However in practice it's difficult to choose this threshold value.

![](./2017-08-02/learning-rate.png)

**If α is too small: slow convergence.**

**If α is too large: ￼may not decrease on every iteration and thus may not converge.**

#### Features and Polynomial Regression

We can **change the behavior or curve** of our hypothesis function by making it a quadratic, cubic or square root function (or any other form).

One important thing to keep in mind is, if you choose your features this way then feature scaling becomes very important.

#### Normal Equation

![](./2017-08-02/normal-equation.png)

![](./2017-08-02/normal-equation-1.png)

if XtX is **noninvertible**, the common causes might be having :

* Redundant features, where two features are very closely related (i.e. they are linearly dependent)
* Too many features (e.g. m ≤ n). In this case, delete some features or use "regularization" (to be explained in a later lesson).





