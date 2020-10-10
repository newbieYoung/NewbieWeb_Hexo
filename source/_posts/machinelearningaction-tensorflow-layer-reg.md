---
title: UI工程师的机器学习之旅（五）隐藏层和正则化
date: 2019-08-02 15:16
tags: [机器学习,Tensorflow,隐藏层]
categories: [机器学习]
author: Young
---

在[Tensorflow MNIST 数字识别](https://newbieweb.lione.me/2019/06/30/machinglearningaction-tensorflow-mnist/)使用`单层神经网络`的正确率大概在 91% 左右，有很多方法对其进行优化，这次简单聊一聊`隐藏层`和`正则化`。

<!--more-->

## 隐藏层

在单层神经网络中只存在`输入层`和`输出层`，而当使用多层神经网络时在`输入层和输出层之间`的其它层级就是`隐藏层`。

引入隐藏层主要原因在于：

- 单层神经网络无法模拟异或运算，但是加入隐藏层之后异或问题就可以得到很好地解决；

- 深层神经网络有组合特征提取的功能，对于解决不易提取特征向量的问题（比如图片识别、语音识别）有很大帮助。

在[单层神经网络](https://github.com/newbieYoung/tensorflow_learn/blob/master/demo8.py)的基础上加入一层隐藏层的改动如下：

<img src="https://newbieyoung.github.io/images/machinelearningaction-tensorflow-layer-reg-0.jpg">

首先定义隐藏层的节点数

```
LAYER1_NODE = 500
```

然后定义隐藏层参数和计算

```
# 隐藏层参数
w1 = tf.Variable(tf.truncated_normal([INPUT_NODE, LAYER1_NODE], stddev=0.1))  # truncated_normal 正态分布产生函数
b1 = tf.Variable(tf.constant(0.1, shape=[LAYER1_NODE]))

y_1 = tf.nn.relu(tf.matmul(x_i, w1)) + b1  # relu 激活函数去线性化
```

需要注意的地方在于`tf.nn.relu`激活函数去线性化；

上述模型`y = w * x + b`本质上来说是线性模型，而任意线性模型的组合依然是线性模型；也就是说只通过线性变换，任意层的全连接神经网络和单层神经网络模型的表达能力没有任何区别；

为了解决这个问题，就需要使用一个非线性函数将神经元的输出进行转换，这个`非线性函数`就是激活函数。

常用的激活函数如下：

<img src="https://newbieyoung.github.io/images/machinelearningaction-tensorflow-layer-reg-1.jpg">

以上是和隐藏层有关的改动。

另外由于计算交叉熵一般会与 Softmax 回归一起使用，所以 Tensorflow 把这两个功能统一封装为`tf.nn.softmax_cross_entropy_with_logits`函数。

完整代码在[https://github.com/newbieYoung/tensorflow_learn/blob/master/demo8-1.py](https://github.com/newbieYoung/tensorflow_learn/blob/master/demo8-1.py)

加入隐藏层后正确率上升到`0.978`左右，效果还是很明显的。

<img src="https://newbieyoung.github.io/images/machinelearningaction-tensorflow-layer-reg-2.jpg">

单层神经网络的计算图和加入隐藏层神经网络计算图对比如下：

<img src="https://newbieyoung.github.io/images/machinelearningaction-tensorflow-layer-reg-3.jpg">

## 正则化

正则化是一个非常常用的避免`过拟合`的方法，以前在[UI工程师的机器学习之旅（二）决定系数与局部加权线性回归](https://newbieweb.lione.me/2018/02/03/machinelearningaction-js-localweighted-linear/)中聊过过拟合和欠拟合；

由于任何函数都可以用多项式去趋近，那么可以假定一个模型：

```
F = W0 * X0 + W1 * X1 + W2 * X2 + ... Wn * Xn;
```

在上述模型`X0、X1、X2...Xn`参数中肯定存在一些不重要的参数，要避免过拟合就需要去掉那些不重要参数的影响，从而`简化模型`；

但是对于一个未知的模型，很难判断哪些参数不重要哪些参数重要！

要解决这个问题其实可以换一种思路，在训练模型时会不断优化`W0、W1、W2...Wn`这些权重参数，此时如果不能去掉那些不重要的参数，那就让这些参数的权重变小，甚至等于 0 就可以了；

因此`正则化的思想就是在损失函数中加入刻画模型复杂度的指标`，从而保证优化过程中既保证模型和训练数据的误差尽可能减小，又能保证得到最优解时会是一个相对简单的模型；

假设描述模型在训练数据上误差的函数为`J(Θ)`，在优化时不直接优化`J(Θ)`，而是优化`J(Θ) + λ*R(W)`；

`λ`表示模型复杂度损失在总损失中的比例；

`Θ`表示神经网络中的所有参数，包括权重`W`和偏置项`b`；

`R(W)`表示模型复杂度，一般来说模型复杂度只由权重 W 决定；

常用的刻画模型复杂度的函数`R(W)`有两种，分别是`L1正则化`和`L2正则化`：

- L1 正则化

<img src="https://newbieyoung.github.io/images/machinelearningaction-tensorflow-layer-reg-4.png">

- L2 正则化

<img src="https://newbieyoung.github.io/images/machinelearningaction-tensorflow-layer-reg-5.png">

这两种正则化有一些区别：

- L1 正则化会让权重参数变得稀疏，L2 正则化则不会，因为使用L2正则化时如果参数很小了，那其平方会更小，接近忽略不计的程度，这时候模型就不会进一步将这个参数调整为 0；

- 因为在优化时需要计算损失函数的偏导数，而 L1 正则化的计算公式不可导，L2 正则化的计算公式可导，因此对含有 L2 正则化损失函数的优化要更加简洁。

在实践中也可以将 L1 正则化 和 L2 正则化同时使用：

<img src="https://newbieyoung.github.io/images/machinelearningaction-tensorflow-layer-reg-7.png">

继续完善代码如下：

<img src="https://newbieyoung.github.io/images/machinelearningaction-tensorflow-layer-reg-6.jpg">

首先定义正则化损失系数`REGULARIZATION_RATE`，然后使用 Tensorflow 框架封装的`tf.contrib.layers.l2_regularizer`函数计算隐藏层权重参数和输出层权重参数的 L2 正则化损失；

总损失等于交叉熵损失加正则化损失，最后梯度优化总损失即可。

完整代码在[https://github.com/newbieYoung/tensorflow_learn/blob/master/demo8-2.py](https://github.com/newbieYoung/tensorflow_learn/blob/master/demo8-2.py)。

由于 MNIST 问题本身相对简单，加入正则化损失对最终正确率的提升效果不明显。

<img src="https://newbieyoung.github.io/images/machinelearningaction-tensorflow-layer-reg-8.jpg">













