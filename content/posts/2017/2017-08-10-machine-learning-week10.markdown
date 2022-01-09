---
Title: coursera Andrew Ng 机器学习第十周笔记 大数据量的机器学习
author: Young
date: 2017-08-10
tags: [ai, machine learning, Andrew Ng, coursera]
Status: public
---
#### stochastic gradident descent

![](./2017-08-10/batch-vs-stochastic.png)

![](./2017-08-10/stochastic.png)

#### mini-batch gradient descent

Batch gradient descent: use all m examples in each iteration

Stochastic gradient descent: use 1 examples in each iteration

Mini-batch gradient descent: use b examples in each iteration

#### checking for convergence

every 1000 iterations, plot cost(theta, xi, yi) averaged over the last 1000 examples processed by algorithm.

![](./2017-08-10/convergence-0.png)

![](./2017-08-10/convergence-1.png)

#### online learning example

* shiping service website where user comes, specifies origin and destination, you offer to ship their package for some asking price, and user sometimes choose to use your shipping service(y=1), sometimes not(y=0). Features x capture properties of user, of origin/destination and asking price. We want to learn p(y=1|x,theta) to optimize price.

* Product search (learning to search)
```
user searches for "android phone 1080p camera"
have 100 phones in store. Will return 10 results.
x=features of phone, how many words in user query match name of phone, 
how many words in query match description of phong
y=1 if user clicks on link y=0 otherwise
learn p(y=1|x,theta)
Use to show user the 10 phones they're most likely to click on.
```

* other examples: choosing special offers to show user; customized selection of news articles; product recommendation;