---
Title: coursera Andrew Ng 机器学习第十一周笔记 应用举例：photo OCR
author: Young
date: 2017-08-11
tags: [ai, machine learning, Andrew Ng, coursera]
Status: public
---
## Application example: photo OCR

#### pipeline

* text detection
* character segmentation
* character classification

#### sliding windows

* too easy to learn

#### Getting more data: artificial data synthesis

![](./2017-08-11/synthesis-1.png)

![](./2017-08-11/synthesis-2.png)

Discussion on getting more data

* make sure you have a low bias classifier before expending the effort.(Plot learning curves.) keep increasing the nuber of features/number of hidden units in neural network until you have a low bias classifier
* how much work would it be to get 10x as much data as we currently have?
 * artificial data synthesis
 * collect/label it yourself
 * Croud source.  e.g. Amazon Mechanical Turk

#### ceiling analysis, what part of the pipeline to work on next

![](./2017-08-11/ceiling-analysis-1.png)

![](./2017-08-11/ceiling-analysis-2.png)

![](./2017-08-11/ceiling-analysis-3.png)




