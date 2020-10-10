---
title: 像素差逐帧动画
date: 2017-02-13 12:34
tags: [逐帧动画]
categories: [前端开发]
author: Young
---

首先这是个从来没有在生产环境中使用的技术，虽然对于交互较简单的展示动画可以使用视频直接替换，但是视频存在很多兼容性问题，比如：

- 目前绝大部分移动端浏览器只支持同时渲染一个视频，因此不能通过切换视频实现具有复杂交互的动画；
- 对于单个视频的时间控制，移动端浏览器的渲染能力也不足以支持不缝隙跳转，因此也不能通过控制视频的时间实现具有复杂交互的动画。

像素差逐帧动画的原理实际就是资源的再压缩然后用还原计算时间换网络加载时间；

大部分情况下逐帧动画前后帧或多或少存在一些相似或者相同的部分，而我们在使用精灵图制作传统逐帧动画时，从动画的整体上考虑实际有很多部分是重复加载了很多次的；

如下图：

<img src="https://newbieyoung.github.io/images/frame-animation-tradition.png">

而像素差逐帧动画主要做的就是剔除掉后一帧中和前一帧相似或者相同的部分（因为这部分在前一帧中已经被加载了），然后根据前一帧以及前一帧和后一帧的差还原出后一帧；这样的话就能既不会丢失数据又能减少加载时所需的资源，而且由于一些新技术的引入（Canvas、WebWorker）实现像素差的计算和还原也变得比以前简单很多。

<!--more-->

### 逐帧动画

##### 1、简介

逐帧动画是一种常见的动画形式（Frame By Frame），其原理是在“连续的关键帧”中分解动画动作，也就是在时间轴的每帧上逐帧绘制不同的内容，使其连续播放而成动画。 因为逐帧动画的帧序列内容不一样，不但给制作增加了负担而且最终输出的文件量也很大，但它的优势也很明显：逐帧动画具有非常大的灵活性，几乎可以表现任何想表现的内容，而它类似与电影的播放模式，很适合于表演细腻的动画。

##### 2、在网页中的实现方式

+ Image标签

<img src="https://newbieyoung.github.io/images/frame-animation-img.png">

`例子`:[http://grupowprojects.com/vw/thinkblue/site/productos-y-soluciones/#tsi](http://grupowprojects.com/vw/thinkblue/site/productos-y-soluciones/#tsi)

> 这种方式的优点是兼容性好，基本没什么浏览器不支持Img标签；缺点则是效率较低，加载资源较多。

+ CSS3

<img src="https://newbieyoung.github.io/images/frame-animation-css3.png">

`例子`:[http://http://tid.tenpay.com/?p=5983https://idiotwu.me/understanding-css3-timing-function-steps/](http://http://tid.tenpay.com/?p=5983https://idiotwu.me/understanding-css3-timing-function-steps/)

> CSS3实现逐帧动画效率高，但是会有兼容性问题。

+ Canvas

<img src="https://newbieyoung.github.io/images/frame-animation-canvas.png">

`例子`:[https://www.futu5.com/activity/threeyear-collection-h5](https://www.futu5.com/activity/threeyear-collection-h5)

> Canvas实现逐帧动画效率也挺高，和CSS3一样也会有兼容性问题，至于Canvas和CSS3这两种方式在不同的情况下也会有不同的表现，总得来说Canvas方式更灵活，但是如果功力不够可能会导致开发效率低以及浏览器渲染效果不好。

### 像素差逐帧动画尝试

像素差逐帧动画主要做的就是剔除掉前后帧中相同或者相似的部分得到像素差，然后根据前一帧和像素差还原出后一帧，这样的话既不会丢失数据又能大幅减少加载所需的资源体积，主要涉及以下四个问题：

##### 1、怎么计算像素差?

主要是利用HTML5的新特性Canvas，Canvas提供了getImageData方法，可以得到类数组形式的图片的像素数据，通过计算前后帧图片的类数组形式像素数据的差异就可以得到类数组形式的像素差。

<img src="https://newbieyoung.github.io/images/frame-animation-compute.png">

##### 2、怎么存储像素差?

由于PNG格式是一种优秀的图片存储格式，为了保证得到的像素差数据存储之后不能大于原图，这里依然考虑用Canvas提供的putImageData方法，把像素差数据进行一些优化后存储为PNG格式并得到相关的位置信息，主要优化方法有如下几点：

+ 首先需要剔除掉像素差类数组中的连续无效像素点，并记录剔除点起始位置和剔除点位数得到相关位置信息，如下：

<img src="https://newbieyoung.github.io/images/frame-animation-position.png">

+ 然后由于现阶段浏览器对Canvas的实现出于性能的考虑，`在像素点的Alpha值小于255的时候，getImageData和putImageData之间存在数据丢失`，为了防止数据丢失对逐帧动画产生影响，这里需要在每个像素点的Alpha值前边手动插入255，从而让原Alpha值后退一位成为下一个像素点的R值；

+ 最后还需要在像素差类数组尾部增加一些连续无效像素点，从而让所有输出的像素差PNG图片具有相同的高度或宽度，方便最终精灵图片的合成。

以上优化操作主要是基于影响PNG图片大小的因素有，PNG图片的宽度和高度、PNG图片中颜色的分布规律、PNG图片中非完全透明像素点的个数。

##### 3、怎么通过像素差还原原始图片？

还原像素差操作其实就是存储像素差的反向操作，不过需要注意的是由于还原像素差操作是用户访问网页时在用户浏览器中进行的，所以和计算以及存储这两个操作不同的是，还原像素差操作需要考虑时间因素，必须尽可能的提高性能；

而浏览器中的JAVASCRIPT数组类型的插入和删除操作效率很低，不适合还原像素差这种需要大量计算的操作，所以这里需要实现一种自定义双向链表结构，同时还要优化该链表的查询效率（因为还原像素差操作也涉及到大量查询操作）。

使用流程图描述为：

<img src="https://newbieyoung.github.io/images/frame-animation-total.png">

简单举例为：

<img src="https://newbieyoung.github.io/images/frame-animation-data.png">

##### 4、怎么实现像素差逐帧动画？

由于通过像素差还原原始图片时涉及到大量计算，为了防止大量计算导致页面卡顿，这里还需要利用HTML5的新特性WebWorker，实现多线程编程。

另外就实现方式而言也分为两种：

+ 一种方式是根据数据还原出所有帧后再按照传统逐帧动画处理；

> 这种方式的优点是能精确控制每一帧之间的时间间隔，缺点是内存消耗过大；

`例子`:[富途三周年动画](https://newbieyoung.github.io/PxDiffFrameAnimation/examples/fututhreeyear.html)；

+ 还有一种方式是先展示第一帧，展示的同时还原下一帧，还原完成之后马上展示，以此类推；

> 这种方式的优点是内存消耗较好，缺点是不能精确控制每一帧的时间间隔；

`例子`:[富途牛牛屏幕演示动画](https://newbieyoung.github.io/PxDiffFrameAnimation/examples/macnnscreen.html)；

### 待改进

+ 相关数据较少，需要更多实践；

+ 制作困难，需要更完善的工具支持；

+ 压缩算法需要优化（减少误差，扩大适用范围，增大压缩比）。









