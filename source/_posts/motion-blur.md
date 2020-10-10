---
title: 运动模糊滤镜
date: 2019-03-26 21:00
tags: [滤镜]
categories: [前端开发]
author: Young
---

运动模糊是指快速运动的物体造成明显的模糊拖动痕迹；

以简单的直线运动为例，运动模糊应该具有两个参数，分别是半径长度和角度；

比如在`Sketch`中如下：

<img src="https://newbieyoung.github.io/images/motion-blur-0.jpg">

但是 CSS、Canvas、SVG 的滤镜均不支持运动模糊。

那是不是意味着如果设计师只是对一个很简单的图形使用了`Sketch`中的运动模糊滤镜，我们在重构时就必须使用图片来实现呢？

<!--more-->

答案并不是。

SVG 的高斯模糊滤镜和 CSS 以及 Canvas 的并不太一样；

CSS 和 Canvas 高斯模糊滤镜都只支持一个参数，但是 SVG 中 `feGaussianBlur` 标签的 `stdDeviation` 属性却可以有两个参数，分别表示 X 和 Y 轴上的模糊半径；

```
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
    <defs>
        <filter id="blur2">
            <feGaussianBlur stdDeviation="20 0" />
        </filter>
        <filter id="blur3">
            <feGaussianBlur stdDeviation="11 0" />
        </filter>
    </defs>
</svg>
```

这就意味着如果把 Y 轴的模糊半径设置为 0，那么是可以模拟出水平直线运动的运动模糊效果的。

```
.demo0{
    width:165px;
    height:165px;
    border-radius:50%;
    opacity: 0.62;
    background-image: radial-gradient(circle at 50% 50%,#4185FC 35%, #0352DB 91%);
    filter: url("#blur3");
}

.demo1{
    width:256px;
    height:256px;
    background-image:url(../img/lena.jpg);
    background-size:100% 100%;
    background-repeat: no-repeat;
    filter: url("#blur2");
}
```

<img src="https://newbieyoung.github.io/images/motion-blur-1.jpg">

另外模拟任意角度的运动模糊效果也是可以的，不过需要进行分层处理，如下：

```
.layer-reset{
    width:256px;
    height:256px;
    transform: rotate(-45deg);
}
.layer-blur{
    width:100%;
    height:100%;
    filter: url("#blur2");
}
.layer-img{
    width:100%;
    height:100%;
    background-image:url(../img/lena.jpg);
    background-size:100% 100%;
    background-repeat: no-repeat;
    transform: rotate(45deg);
}
```

`feGaussianBlur`只能模拟水平或者竖直方向的运动模糊，要想模拟任意角度，就只能对内容进行旋转变换，然后再在最外层进行反向旋转变换还原。

<img src="https://newbieyoung.github.io/images/motion-blur-2.jpg">

在线示例地址为：[https://newbieyoung.github.io/SVG_Learn/filter/demo0.html](https://newbieyoung.github.io/SVG_Learn/filter/demo0.html)

在[常用滤镜算法以及WebGL实现](https://newbieweb.lione.me/2017/08/22/normal-filter-intro/)中曾简单介绍过一个基于 WebGL 的滤镜库 [WebGLImageFilter](https://github.com/phoboslab/WebGLImageFilter)；

<img src="https://newbieyoung.github.io/images/motion-blur-3.jpg">

作者在实现模糊滤镜也没考虑过运动模糊；

但是我们可以稍作改动来让其支持：

<img src="https://newbieyoung.github.io/images/motion-blur-4.jpg">

主要改动点为：

- 1、新增第二个参数旋转角度`deg`；

- 2、当存在旋转角度时参数时，只进行`Horizontal`的处理；

- 3、对待处理片元向量进行旋转变换；

- 4、迭代次数由 7 次改为 30 次，另外权重函数参考 [glfx.js triangleblur](https://github.com/evanw/glfx.js/blob/master/src/filters/blur/triangleblur.js) 的实现。

迭代次数太小会导致模糊半径较大时出现很明显的分块现象，[glfx.js](https://github.com/evanw/glfx.js) 是另外一个基于 WebGL 的滤镜库。

<img src="https://newbieyoung.github.io/images/motion-blur-5.jpg">

左侧是`Sketch`中`Motion Blur`的滤镜效果，右侧是改动后`WebGLImageFilter`的`blur`滤镜的效果，除了对边缘的处理不一样以外，基本没啥区别。

在线示例地址为：[https://newbieyoung.github.io/Html_learn/test/demo1.html](https://newbieyoung.github.io/Html_learn/test/demo1.html)。

关于运动模糊的内容远不止上边这些那么简单，还有一些很复杂的内容，比如3D场景、其它运动模式等，如果有机会再慢慢折腾吧！































