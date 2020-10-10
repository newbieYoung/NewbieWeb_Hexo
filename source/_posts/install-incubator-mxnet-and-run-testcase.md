---
title: 在 MAC 操作系统中安装 Mxnet 并运行其中的梵高绘画风格模拟例子
date: 2017-08-23 17:41
tags: [Mxnet]
categories: [机器学习]
author: Young
---

前些天偶然间看到大家在讨论某些活动页面风格不一致的情况，然后想到了很久以前看过的一篇文章关于深度学习框架 Mxnet 的文章；

文章中有一个例子能把任意图片转换成梵高的绘画风格，结合上述大家讨论的情况，开了个脑洞：

能不能通过这个例子做一个判断页面风格是否一致的工具呢？

<!--more-->

比如：

+ 首先确定某个页面为标准风格；
+ 然后使用 Mxnet 框架中的例子把活动页面转换为标准风格；
+ 然后比较该活动页面标准风格和原始风格的相似程度来判断该活动页面的原始风格是否符合既定的标准风格。

折腾了一下午证实了这个脑洞是个失败的脑洞，主要原因是 Mxnet 框架里边的这个例子中的算法不具备通用性，只能把任意图片转换成“特征“很明显的图片风格。

而不同网页的风格相比不同的绘画风格，其实是差异很小的。

虽然脑洞失败了，但是毕竟折腾了一下午，在安装 Mxnet 以及运行例子程序时遇到了很多问题，鉴于网上的教程大多不靠谱，还是记录一下，万一后边对深度学习之类有兴趣了呢！

## 安装

其实安装流程参照[官网流程](http://mxnet.io/get_started/install.html)即可，比网上的教程靠谱很多，一般不会出现啥问题。

## 运行

Github 上有很多信息，包括梵高绘画风格模拟例子的怎么运行的[指引](https://github.com/apache/incubator-mxnet/tree/master/example/neural-style)。

但是在执行`python nstyle.py`时却报错`ImportError: No module named skimage`；

根据错误信息可知，Python中某个叫`skimage`的模块导入失败，有可能是该模块不存在或者版本不对，此时需要安装或者更新该模块；

执行`sudo pip install scikit-image --upgrade`；

此时还是有问题：

`Found existing installation: six 1.4.1
	DEPRECATION: Uninstalling a distutils installed project (six) has been deprecated and will be removed in a future version...`

大致原因是因为安装skimage时本地已经安装的six库版本过低需要更新，结果更新时没有权限就报错了；

因此需要忽略six安装skimage；

`sudo pip install scikit-image --upgrade --ignore-installed six`；

结果还是报错：

`OSError:[Errno 1] Operation not permitted: '/System/*/Python.framework/Version/2.7/share'`

新版的MAC操作系统有一个叫SIP的机制禁止任何用户直接修改System目录的文件，解决办法是取消这种机制，具体做法是：

+ 重启电脑，按住Command+R（直到出现苹果标志）进入恢复模式；

+ 左上角菜单中打开终端；

+ 执行`csrutil disable`；

+ 重启电脑即可；

如果想要启用 SIP 机制，按上述流程，执行`csrutil enable`即可。

到此应该不存在其它环境问题了，但是运行例子程序时还是报错：

`Operator _zeros is not implemented for GPU.`

原因在于该例子程序默认是在GPU上执行的，但是我们安装的时候是安装的CPU版本，因此需要在运行指定在CPU上执行；

`python nstyle.py --gpu -1`。

## 结语

虽说脑洞失败，但这个例子其实还是有那么一点意思的，比如生成一张梵高绘画风格的图片......

<img src="https://newbieyoung.github.io/images/incubator-mxnet-0.jpg">







