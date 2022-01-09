---
Title: coursera Andrew Ng 机器学习第八周笔记 降低维度
author: Young
date: 2017-08-08
tags: [PCA, ai, machine learning, Andrew Ng, coursera]
Status: public
---
## unsupervised learning

#### k-means altorithm

![](./2017-08-08/k-means.png)

#### clustering, optimization objective

![](./2017-08-08/k-means-optimization.png)

#### clustering, random initialization

should have K < m

Randomly pick K training examples.

#### clustering, choosing the number of clusters, K

Please draw a graph. **Elbow method**

![](./2017-08-08/elbow-method.png)

## Dimensionality reduction

**Motivation I: data compressioni**

**Motivation II: Visualization**

#### Princiapl Component Analysis

reduce from n-dimension to k-dimension:
find k vectors u1 u2 u3 uk onto which to project the data,so as to minimize the projection error.

**PCA steps**

* preprocessing: feature scaling + mean normalization
* compute "covariance matrix"
* compute "eigenvectors" of matrix Sigma```[U,S,V] = svd(Sigma);```

![](./2017-08-08/pca-summary.png)

![](./2017-08-08/pca-choosek.png)

#### advice

mapping matrix(from N-d to K-d) should be defined by running PCA only on the training set.
This mapping can be applied as well to the examples X-cv and X-test in the cross validation and test sets.

**application of PCA**

* compression
 * reduce memory/disk needed to store dat
 * speed up learning algorithm
* visualization

![](./2017-08-08/pca-advice1.png)

![](./2017-08-08/pca-advice2.png)




