---
Title: coursera Andrew Ng 机器学习第一周笔记 线性回归
author: Young
date: 2017-08-01
tags: [ai, machine learning, Andrew Ng, linear regression]
Status: public
---
## machine learning

supervised learning, unsupervised learning, others(reinforcement learning, recommender system)


### supervised learning

* regressing，predict input into continuous result     
* classification, predict input into distract result                

examples：


* given size, rooms, predict house price, like 1M, 100Grand, 1B    
* house prize is  expensive or cheap ?
* given picture with human, predict his age: 0.1，12.5，78.8
* given a patient with tumor, predict it's a malignant or benign


### unsupervised learning
problems with little or no idea what our results should look like

derive structure from data

clustering the data based on relationships among the variables in the data.

there is no feedback based on the prediction results

examples：

* Clustering: Take a collection of 1,000,000 different genes, and find a way to automatically group these genes into groups that are somehow similar or related by different variables, such as lifespan, location, roles, and so on.

* Non-clustering: The "Cocktail Party Algorithm", allows you to find structure in a chaotic environment. (i.e. identifying individual voices and music from a mesh of sounds at a cocktail party).
 

### Linear Regression with one var

#### cost function

![](./2017-08-01/cost-function.png)

#### gradient descent

* repeat until convergence:

![](./2017-08-01/gradient-descent.png)

![](./2017-08-01/gradient-descent-1.png)

* pay attention to the simultaneous update:

![](./2017-08-01/simultaneous-update.png)

* about the alpha, gradient

![](./2017-08-01/alpha.png)
















