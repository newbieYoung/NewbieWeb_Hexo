---
title: 不建议使用 CSS3 keyframe transform 实现逐帧动画
date: 2018-12-26 21:43
tags: [逐帧动画]
categories: [前端开发]
author: Young
---

一般在使用 CSS3 keyframe transform 实现逐帧动画时，往往会先把逐帧动画图片合成精灵图，然后再使用 transform 改变其位置，这个方法会消耗大量显存（帧数越多，合成的精灵图越大，显存消耗越多），容易出现闪烁的问题。

直接看例子（以下截图都来自于 OPPO FindX Chrome 远程调试）：

<img src="https://newbieyoung.github.io/images/css-frame-animation-0.gif">

链接为 [https://newbieyoung.github.io/Html_learn/animation/demo1.html](https://newbieyoung.github.io/Html_learn/animation/demo1.html)；

<!--more-->

<img src="https://newbieyoung.github.io/images/css-frame-animation-1.jpg">

页面中有50个使用 CSS3 keyfrmae 结合 transform 实现的逐帧动画；

为什么会闪烁，大概是因为 transform 会导致浏览器开启硬件加速，每个动画区域都是一个独立的渲染层；

在调试工具的 Layers 中可以看到：

<img src="https://newbieyoung.github.io/images/css-frame-animation-3.jpg">

另外还要注意每个独立的渲染层并不仅仅只是显示出来的那一小块区域，而是整个逐帧动画精灵图内容区域，这应该就是为什么例子中的逐帧动画会消耗那么多显存的原因了。

<img src="https://newbieyoung.github.io/images/css-frame-animation-2.jpg">

当选中 Rendering 中的 FPS meter 就可以看到，总消耗显存 130M 左右。

<img src="https://newbieyoung.github.io/images/css-frame-animation-4.jpg">

接着再看 Performance：

<img src="https://newbieyoung.github.io/images/css-frame-animation-5.jpg">
<img src="https://newbieyoung.github.io/images/css-frame-animation-6.jpg">

可以看到因为渲染层太多，光栅化会消耗较长时间，此时页面空白，较长时间的空白就会造成闪烁。

怎么解决这个问题呢？

### Canvas逐帧动画

<img src="https://newbieyoung.github.io/images/css-frame-animation-7.gif">

链接为 [https://newbieyoung.github.io/Html_learn/animation/demo2.html](https://newbieyoung.github.io/Html_learn/animation/demo2.html)；

普通的2D Canvas并不是独立的渲染层，渲染区域也仅仅只是显示的那一块区域，并不会消耗太多显存；

<img src="https://newbieyoung.github.io/images/css-frame-animation-8.jpg">

<img src="https://newbieyoung.github.io/images/css-frame-animation-9.jpg">

<img src="https://newbieyoung.github.io/images/css-frame-animation-10.jpg">

最后上述 Canvas逐帧动画是可以再优化一下的；

<img src="https://newbieyoung.github.io/images/css-frame-animation-11.jpg">

每次绘制时都是从整个精灵图中选取一部分绘制，这其实是比较耗时的操作；

<img src="https://newbieyoung.github.io/images/css-frame-animation-12.jpg">
<img src="https://newbieyoung.github.io/images/css-frame-animation-13.jpg">

正确的做法应该在精灵图加载完成之后，先从精灵图中拆出每一帧的图片，然后绘制时直接绘制特定帧的图片即可。

这样优化之后，页面中 Paint 阶段的耗时就可以从 10毫秒 减少为 5 毫秒了。

<img src="https://newbieyoung.github.io/images/css-frame-animation-14.jpg">

优化后的链接为 [https://newbieyoung.github.io/Html_learn/animation/demo3.html](https://newbieyoung.github.io/Html_learn/animation/demo3.html)；

### 总结

- 硬件加速并不是银弹，当满足某些条件时，甚至有可能是毒药；
- 当页面中存在CSS动画时最好使用调试工具查看下 Layers，可能会因为一些用法不当的原因导致出现渲染层过多的情况。

### 附录

- [渲染层爆炸例子](http://taobaofed.github.io/demo/performance-composite-demo/memory/layer-explode.html)
- [LayoutObjects、PaintLayers、GraphicsLayers、assumedOverlap](http://taobaofed.org/blog/2016/04/25/performance-composite/)







