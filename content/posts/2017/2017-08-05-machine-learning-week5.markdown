---
Title: coursera Andrew Ng 机器学习第五周笔记 神经网络的学习过程
author: Young
date: 2017-08-05
tags: [ai, machine learning, Andrew Ng, Neural Network]
Status: public
---
## Neural Networks: Learning

#### cost function

```
L = total number of layers in the network
sl = number of units (not counting bias unit) in layer l
K = number of output units/classes
```

![](./2017-08-05/cost-function.png)

```
the double sum simply adds up the logistic regression costs calculated for each cell in the output layer
the triple sum simply adds up the squares of all the individual Θs in the entire network.
the i in the triple sum does not refer to training example i
```

#### Backpropagation Algorithm

![](./2017-08-05/backpropagation-0.png)

![](./2017-08-05/backpropagation-1.png)


#### Backpropagation in Practice

```
thetaVector = [ Theta1(:); Theta2(:); Theta3(:); ]
deltaVector = [ D1(:); D2(:); D3(:) ]
Theta1 = reshape(thetaVector(1:110),10,11)
Theta2 = reshape(thetaVector(111:220),10,11)
Theta3 = reshape(thetaVector(221:231),1,11)
```

#### gradient checking

![](./2017-08-05/gradient-checking.png)

Once you have verified **once** that your backpropagation algorithm is correct, you don't need to compute gradApprox again. The code to compute gradApprox can be very slow.

#### random initialization

![](./2017-08-05/random-init.png)

#### Whole workflow

First, pick a network architecture; choose the layout of your neural network, including how many hidden units in each layer and how many layers in total you want to have.

* Number of input units = dimension of features x(i)
* Number of output units = number of classes
* Number of hidden units per layer = usually more the better (must balance with cost of computation as it increases with more hidden units)
* Defaults: 1 hidden layer. If you have more than 1 hidden layer, then it is recommended that you have the same number of units in every hidden layer.

**Training a Neural Network**

* Randomly initialize the weights
* Implement forward propagation to get hΘ(x(i)) for any x(i)
* Implement the cost function
* Implement backpropagation to compute partial derivatives
* Use gradient checking to confirm that your backpropagation works. Then disable gradient checking.
* Use gradient descent or a built-in optimization function to minimize the cost function with the weights in theta.

```
for i = 1:m,
   Perform forward propagation and backpropagation using example (x(i),y(i))
   (Get activations a(l) and delta terms d(l) for l = 2,...,L
```
![](./2017-08-05/train-nn.png)
