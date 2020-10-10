---
title: UI工程师的机器学习之旅（一）线性回归和梯度下降
date: 2017-11-23 17:00
tags: [机器学习,线性回归,梯度下降]
categories: [机器学习]
author: Young
---

不知道从什么时候开始人工智能突然就火了起来，某天机缘巧合被`Mxnet`框架里边的一个把任意一副图片转换成梵高绘画风格的例子给震惊了，感觉非常神奇，竟然还有这种操作，然后就开始坎坷的学习机器学习的旅程。

<!--more-->

开始学习是在网易云课堂上的[深度学习工程师微专业](https://study.163.com/my#/smarts)；

要想大概了解一些背景知识或者行业现状可以看看第一周的课程和里边的人工智能行业大师访谈；

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-0.jpg">

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-1.jpg">

后续课程直接是从`logistic回归`开始；

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-11.jpg">

初一接触感觉理解有点困难，在网上查找资料的时候发现`线性回归`相对来说更好理解一点，就干脆先把`logistic回归`放一放转而去学习`线性回归`，并在网易公开课上找到了另外一门课程[斯坦福大学公开课 ：机器学习课程](http://open.163.com/special/opencourse/machinelearning.html)。

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-2.jpg">

第一集主要是一些概览型的知识，第二集和第三集主要讲解的是`线性回归`和`梯度下降`，第四集开始讲解`logistic回归`。

说了这么一堆流水账主要是想说`下文中所有关于理论的知识都是来源上述两个课程，本人初学阐述起来可能会解释不清或者甚至可能存在错误，因此建议还是看看权威资料为妙！`。

在觉得自己大概了解`线性回归`和`梯度下降`相关知识后，想着大概需要实践一下加深理解了，但是实际却发现首先是缺乏数据，就算是写爬虫去爬也一时间不知道爬啥，另外也不能确定爬取的数据符合线性模型；

因此又去网上找了一本书[《机器学习实战》](https://book.douban.com/subject/24703171/)，书中有一些自带数据且说明模型的的算法实践例子，所有代码都在[machinelearninginaction](https://github.com/pbharrin/machinelearninginaction)这个项目里边。

由于本人对 Python 不熟再加上书中 Python 代码注释较少，看起来略吃力，后来一想如果只是学习练习的话，使用什么语言应该是小事才对，所以就有了这个项目[machinelearninginaction-js](https://github.com/newbieYoung/machinelearninginaction-js)，准备用自己熟悉的 JS 实现一些机器学习算法，顺便还能做些可视化的东西。

比如下边这个[基于线性回归演示各种梯度下降过程](https://newbieyoung.github.io/machinelearninginaction-js/Ch08/ex0.html)的例子。

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-3.gif">

## 线性回归

在统计学中，线性回归是利用称为线性回归方程的最小二乘函数对一个或多个自变量和因变量之间关系进行建模的一种回归分析。

简单来说如果有一堆如下数据，可以近似看成线性模型，并拟合出那条黑色的回归线，且能根据回归线做出符合预期的预测，这整个过程就叫线性回归吧......

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-4.jpg">

上边的模型中只有一个因变量因此可以叫做`一元线性回归`，回归线一般可以写成如下形式：

```
y = p0+p1*x
```

如果存在多个自变量时，回归模型表示为以下形式：

```
y = p0+p1*x1+p2*x2+...+pn*xn;
```

## 梯度下降		

以一元线性模型为例，我们要求得线性回归线就必须找到在一条拟合模型中所有数据的误差最小的线。

在机器学习中优化算法，除了梯度下降以外，还有最小二乘法、牛顿法以及拟牛顿法等多种方法，最小二乘法适合样本数据量较小的情况，可以直接求解；牛顿法和拟牛顿法是用二阶的海森矩阵的逆矩阵或伪逆矩阵求解，收敛更快，但是每次迭代时间比梯度下降长。

误差我们取实际值和预测值距离的平方和，如图：

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-5.jpg">

对于`L`来说其误差为`l1*l1+l2*l2+l3*l3+l4*l4+l5*l5`；

假设某一元线性模型样本数据数量为`m`，分别为`(x1,y1)`、`(x2,y2)`......`(xm,ym)`；回归线为`H(x) = p0+p1*x`；那么其误差如下：

```
J(p0,p1) = (H(x1)-y1)+(H(x2)-y2)+......+(H(xm)-ym)
```

可以简单写为如下形式：

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-6.jpg">

关于为什么误差是实际值和推测值差的平方的和，课程里边其实也没做过多解释，可以先简单的理解为误差是实际值和推测值差的平方的和时误差符合正态分布；实际推导我准备放到`logistic回归`里边再聊，因为需要用到`极大似然估计`；

另外为什么我们假定误差符合正态分布？这个问题我很难解释清楚，可以去了解了解高斯的一些事以及极限中心定理，正态分布在误差分析中的地位应该是高斯奠定的，而且极限中心定理指出，如果一个随机变量是若干独立随机变量的总和，当被加的个数趋于无穷大时，它的概率密度函数近似高斯密度...BA...LA...BA...LA...

最后取二分之一则是为了计算方便，后边需要求偏导数。

上述误差函数同样适用于多元线性回归模型，假设某多元线性回归模型样本数量还是`m`，分别为`(x11,x12,x13...,x1n,y11)`、`(x21,x22,x23...,x2n,y21)`......`(xm1,xm2,xm3...,xmn,ym1)`，回归线为`H(x) = p0+p1*x1+p2*x2+...+pn*xn`；那么其误差函数可以表示为`J(p0,p1...pn)`；

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-10.jpg">

现在我们知道了误差函数，接下来按照梯度下降的方法来求误差函数的最小值，在开始之前我们得明确以下几点：

- 得确保误差函数存在最小值（如果不存在可以对整体结果取负）并且可微（也就是在定义域内所有点都存在导数）；
- 其二把某次样本数据代入到误差函数中后，函数中的自变量实际为参数`p1、p2......pn`；

还是以简单的一元线性模型误差函数举例，其函数图如下：

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-9.jpg">

`Z`轴表示误差、`X`轴是参数`p0`、`Y`轴是参数`p1`；因此显示出来有点类似三维地形图；当然你可能把偏导数都给忘记了，觉得还是有点复杂，没关系我们可以用更简单的例子来说明，如下：

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-8.jpg">

按照梯度下降方法来，首先我们假定初始化位置在点`A`，然后把点`A`的自变量`x`减去`A`点导数的`alpha`倍，得到点`B`；依此类推，因为越靠近最小值导数越接近0，因此从`A`点按照这种规律迭代只会发生一种情况就是不断的向最低点靠近，就算某次变化到了最低点的左侧也会因为导数变为负数从而向右侧运动。

但是我们需要注意两个地方：

- 学习速度`alpha`取值，如果取太大容易导致始终在点`A`和点`C`间循环迭代甚至越迭代越远离最低点，如果取太小会导致迭代速度很慢，因此学习速度的选取需要多次运行后才能得到一个较为优的值；
- 如果初始化位置在点`E`那么使用梯度下降取得得最小值可能会是局部最小值，因此梯度下降有局部最优解的风险，还是需要多次运行，选取不同的初始值才能在存在多个局部最优解时相对靠近真正最优解的值。

综上所述使用梯度下降法可得`P`的更新过程：

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-7.jpg">

关于什么是`偏导数`、什么是`梯度`？我们以三维空间中的曲面为例，在曲面的每一点都有无穷多条切线，`偏导数`就是选取其中一条切线，一般取垂直于`Y`轴以及垂直于`X`轴的切线；而这两个偏导数组成一个向量就是这个曲线在该点的`梯度向量`，从几何意义上来说`梯度向量`的方向就是曲面函数在该点增加最快的方向，沿着梯度方向更容易找到函数的最大值，反过来说，沿着梯度方向的反方向也就更容易找到函数的最小值了。

最终求出的更新过程方程式如下：

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-12.jpg">

## 算法实现

首先定义线性模型；

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-13.jpg">

然后实现梯度更新过程方程式；

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-14.jpg">

最后进行梯度下降迭代；

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-15.jpg">

为了观察到整个过程我们还可以把每次迭代的值保存下来，然后进行可视化显示（散点图和直线图都是使用`HighCharts`绘制）；

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-16.jpg">

## 算法扩展

上边实现的梯度下降算法是`批量梯度下降`，每次更新我们都需要在整个数据集上求出所有的偏导数，计算量较大，在数据集很大的情况下会消耗大量内存而且每次迭代的速度也会很慢，很难使用；因此诞生了另外两种方法，分别是`随机梯度下降`和`小批量梯度下降`；相比于`批量梯度下降`而言`随机梯度下降`的每次更新是对数据集中的随机一个样本进行计算，因此它的运行速度被大大加快，但是因为使用的数据太少会导致另外一个问题，即迭代时会产生剧烈波动的情况，导致迭代次数较多；`小批量梯度下降`结合了上述两种方法的优势，每次更新时会随机取一部分数据进行计算，这种方式既加速了运算速度又使得收敛过程较为稳定。

对上述两种方法简单实现如下：

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-17.jpg">

聊到这实在按耐不住内心的惶恐，忍不住补充一句，上边只是`简单`的实践和阐述了一些学习收获，实际情况比这些复杂的多哈......

## 结语

东西太多，为了防止文章过长就暂时到此为止了，如果你仔细思考就会发现，几乎任意一数据集都可以使用上述方法来建立模型，但是我们怎么判断这些模型的好坏呢？上述例子数据量小而且自变量只有一个，可以很简单的被可视化出来，我们可以肉眼观察，但是那些数据量很大，变量因素很多的怎么办？

另外我们可以看到虽然上述例子可以近似看成一条直线，但是每隔一段距离存在一段有规律的曲线，但是我们回归出来的直线很明显没办法表现这种规律，针对这种情况我们又应该怎么处理？

后续有机会再聊`相关系数`和`局部加权线性回归`等。

对了，有兴趣的同学一起来玩哈！[machinelearninginaction-js](https://github.com/newbieYoung/machinelearninginaction-js)。














