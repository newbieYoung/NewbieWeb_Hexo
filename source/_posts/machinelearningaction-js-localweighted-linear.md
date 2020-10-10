---
title: UI工程师的机器学习之旅（二）决定系数与局部加权线性回归
date: 2018-02-03 18:30
tags: [机器学习,决定系数,局部加权线性回归]
categories: [机器学习]
author: Young
---

在[UI工程师的机器学习之旅（一）线性回归和梯度下降](https://newbieweb.lione.me/2017/11/23/machinelearningaction-js-linear-regression/)中简单的用`JavaScript`实践了线性回归，同时留下了两个问题：

- 怎么判断根据数据拟合出的方程模型的好坏？
- 拟合出的直线确实符合数据的整体趋势，但是要想更精确的拟合怎么办？

<!--more-->

## 问题一：怎么判断根据数据拟合出的方程模型的好坏？

其实在统计学中`决定系数`这个概念就是用来解决这个问题的，`决定系数`也被称为`判定系数`或者`拟合优度`。

如果看过上篇文章的童鞋应该还记得，上篇文章在结尾明明提起的是`相关系数`，怎么到这里变成了`决定系数`？

主要是因为`相关系数`的平方即为`决定系数`，因此`相关系数`用大写字母`R`表示而`决定系数`则用`R2`表示；

由于R2小于R，可以防止R对模型做出一些夸张的解释，因此这里采用`决定系数`来判断。

其实抛开这些概念也可以这么来理解；

判断模型的好坏关键在于模型是否能更好的反应因变量随自变量变化的波动，也就是说模型能表达的波动占总体波动的比例越高，模型越好；比例越低，模型越差。

总波动就是各个因变量值和其平均数的差的平方和。

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-localweighted-linear-0.jpg">

而模型能表达的波动等于总波动减去模型未能表达的波动，未能表达的波动其实就是线性回归的误差；因此计算R2的方法如下：

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-localweighted-linear-1.jpg">

在[简单线性回归梯度下降](https://newbieyoung.github.io/machinelearninginaction-js/Ch08/ex0.html)示例中打开调试面板可以看到拟合的线性方程的`决定系数`。

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-localweighted-linear-2.jpg">

另外`决定系数`达到多少为宜，其实并没有统一的明确界限，实际还和数据类型有关系，不过可以简单的认为0.8以上为优，0.4以下为差。

以上统计学相关知识来源于[可汗学院公开课：统计学](http://open.163.com/special/Khan/khstatistics.html)。

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-localweighted-linear-3.jpg">

其中`第68集`到`第70集`为`决定系数R2`相关内容。

## 问题二：拟合出的直线确实符合数据的整体趋势，但是要想更精确的拟合怎么办？

上篇文章拟合的整体效果如图：

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-4.jpg">

虽然`决定系数R2`等于0.97，已经是一个相当高的数值了，但是你还是不太满意，因为数据中有很明显的特征并没有被体现出来，比如每隔一段就有一个类似三角函数的波段；因此你可能觉得纯线性函数并不可取，实际模型因该是线性函数加上三角函数，比如：

```
y = a*x+c*sin(x)+b;
```

在不断的改变模型和特征（多元线性回归中存在多个自变量，你可以加入新的自变量或者去掉已有的自变量）然后拟合的过程中得到你满意的结果，但是其实还有另外一种相对简单的算法来解决这个问题，这个算法被称为`局部加权线性回归`。

`局部加权线性回归`关键在于预测一个点的值时，选择与这个点相近的点而不是所有的点做线性回归；在这个算法中，离预测值越近，权重越大，对回归系数的贡献就越多，如下：

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-localweighted-linear-4.jpg">

其中对于一元线性回归权重是自变量`X`的距离的平方，在多元线性回归中则是自变量组成的向量差的模的平方。

对于权重方程可能看起来有点眼熟，有点类似正态分布函数；

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-localweighted-linear-5.jpg">
<img src="https://newbieyoung.github.io/images/machinelearningaction-js-localweighted-linear-6.jpg">

当然这里并没有任何正态分布的意义，在这个算法之所以使用这个权重函数，仅仅只是因为这个函数恰好满足，距离无穷远时权重为0，距离不断靠近时，权重不断靠近1。

因为预测某个值需要计算所有训练数据相对当前预测值的权重，有点类似于批量线性回归，因此代码可以基于批量线性回归算法修改；不过需要注意以下几个地方：

首先新增了权重计算函数；

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-localweighted-linear-8.jpg">

批量梯度下降时考虑权重的影响；

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-localweighted-linear-9.jpg">

在局部加权线性回归中并没有固定的线性回归方程，而是每次预测都需要重新根据所有数据进行计算得到最适合当前预测值的线性回归方程；

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-localweighted-linear-10.jpg">

根据该算法可以得到预测数据然后把预测数据和训练数据一起绘制出来就可以可视化该算法的实际的效果了，如下：

[局部加权线性回归](https://newbieyoung.github.io/machinelearninginaction-js/Ch08/ex1.html)

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-localweighted-linear-7.jpg">

另外还需要注意一个地方，关于权重函数中波长参数的取值；

+ `k=1`

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-localweighted-linear-11.jpg">

+ `k=0.1`

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-localweighted-linear-12.jpg">

+ `k=0.01`

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-localweighted-linear-13.jpg">

+ `k=0.001`

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-localweighted-linear-14.jpg">

可以看到随着波长不断变小，预测数据分布慢慢从普通的线性回归方程变成和训练数据接近完全重合；对于`k=1`和`k=0.1`这两种情况我们可以称之为`欠拟合`；而对于`k=0.001`这种情况我们称之为`过拟合`。

基本上到这里文章内容已经结束，但还有两个问题并没有阐述清楚，比如：波长参数应该取多少才能既避免欠拟合情况又能避免过拟合情况？局部线性回归算法每次预测都需要计算全部训练数据，如果训练数据很庞大时性能显然很低，这时又应该怎么办？

后续有机会再聊`树回归`和`权衡方差和偏差`等。















