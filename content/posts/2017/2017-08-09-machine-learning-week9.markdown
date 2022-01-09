---
Title: coursera Andrew Ng 机器学习第九周笔记 异常检测与推荐系统
author: Young
date: 2017-08-09
tags: [ai, machine learning, Andrew Ng, coursera]
Status: public
---
## Anomaly Detection

#### Density estimation
Dataset: x1, x2, x3, ... xm
Is X-test anomalous ?

#### algorithm

![](./2017-08-09/anomaly-detection.png)

#### anomaly detection vs supervised learning

![](./2017-08-09/vs-1.png)

![](./2017-08-09/vs-2.png)

#### choosing what features to use

non-gaussian features, maybe log(x),sqrt(x),x^2 to gaussian-like features

#### multivariate G distributon

Don't model p(x1) p(x2) separately.

Modle p(x) all in one go.

![](./2017-08-09/multi-G-1.png)

![](./2017-08-09/multi-G-2.png)

![](./2017-08-09/multi-G-3.png)

![](./2017-08-09/multi-G-4.png)

## Recommender Systems

![](./2017-08-09/recommender-formulation.png)

![](./2017-08-09/recommender-example.png)

#### collaborative filtering algorithm

![](./2017-08-09/collaborative-algo.png)

#### finding related movies

smallest|| xi - xj ||

#### users who have not rated any movies

mean normalization