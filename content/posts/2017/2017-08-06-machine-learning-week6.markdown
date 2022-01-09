---
Title: coursera Andrew Ng 机器学习第六周笔记 系统设计与优化技巧
author: Young
date: 2017-08-06
tags: [ai, machine learning, Andrew Ng, coursera]
Status: public
---
## Advice for Applying Machine Learning

### 重要的笔记，必须手写上传

![](./2017-08-06/advice-1.png)

![](./2017-08-06/advice-2.png)

![](./2017-08-06/advice-3.png)

![](./2017-08-06/advice-4.png)

## Machine Learning System Design

### email spam example

Given a data set of emails, we could construct a vector for each email. Each entry in this vector represents a word. The vector normally contains 10,000 to 50,000 entries gathered by finding the most frequently used words in our data set. If a word is to be found in the email, we would assign its respective entry a 1, else if it is not found, that entry would be a 0. Once we have all our x vectors ready, we train our algorithm and finally, we could use it to classify if an email is a spam or not.

![](./2017-08-06/email-spam.png)

So how could you spend your time to improve the accuracy of this classifier?

* Collect lots of data (for example "honeypot" project but doesn't always work)
* Develop sophisticated features (for example: using email header data in spam emails)
* Develop algorithms to process your input in different ways (recognizing misspellings in spam).

It is difficult to tell which of the options will be most helpful.


### error analysis

The recommended approach to solving machine learning problems is to:

* Start with a simple algorithm, implement it quickly, and test it early on your cross validation data.
* Plot learning curves to decide if more data, more features, etc. are likely to help.
* Manually examine the errors on examples in the cross validation set and try to spot a trend where most of the errors were made.

### error metrics for skewed classes

![](./2017-08-06/skewed-class-example.png)

这下查准率和召回率就比较重要了

```
TP = True Positive
FP = Fake Positive
FN = Fake Negative

Precision = TP / (TP + FP)
				
Recall = TP / (TP + FN)

```

### trading off precision and recall

F1-Score = 2 * P * R / (P + R)

### Data for machine learning

**数据量的重要性**

![](./2017-08-06/large-data-importance.png)