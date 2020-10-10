---
title: 聊一聊 ThreeJS 反锯齿
date: 2019-03-05 17:20
tags: [ThreeJS,WebGL,抗锯齿]
categories: [3D技术]
author: Young
---

之所以会折腾这个问题主要是因为自己的魔方微信小游戏在`Android`机型中存在莫名其妙的`锯齿`问题；

<img src="https://newbieyoung.github.io/images/threejs-antialiasing-0.jpg">

那怕是已经在创建`WebGLRenderer`渲染器时设置了反锯齿参数。

<img src="https://newbieyoung.github.io/images/threejs-antialiasing-1.jpg">

曾经有人问我遇到锯齿问题怎么办？

其实除了设置反锯齿参数之外，我也不知道该怎么办...

因此就趁着这个机会稍微了解了一下。

<!--more-->

## 为什么会有锯齿？

锯齿的出现和`光栅器`的工作方式有关，顶点坐标理论上可以取任意值，但是`片元`不行，因为它们受限于显示器的分辨率，所以光栅器必须以某种方式来决定每个片元最终的屏幕坐标；

<img src="https://newbieyoung.github.io/images/threejs-antialiasing-2.png">

在上图中每个像素中心包含一个采样点，它会被用来决定这个三角形是否覆盖了某个像素点，图中红色的采样点被三角形所覆盖，在每个被覆盖的像素处都会生成一个片元；虽然三角形边缘的一些部分也遮住了某些屏幕像素，但是这些像素的采样点并没有被三角形内部所覆盖，所以它们不会被片元着色器影响，最终渲染出来的效果如下：

<img src="https://newbieyoung.github.io/images/threejs-antialiasing-3.png">

由于显示器的分辨率限制，有些边缘的像素能被渲染出来，有些则不会，造成的结果就是渲染的图像具有不平滑的边缘，这也就是锯齿出现的原因了。

## 超采样抗锯齿（SSAA）

最开始大家使用一种叫`超采样抗锯齿（Super Sample Anti-aliasing）`的技术来解决这个问题，它会使用比正常分辨率更高的分辨率来渲染场景，当图像输出在帧缓冲中更新时，分辨率会被下采样（Downsample）至正常的分辨率，这些额外的分辨率会被用来防止锯齿的产生，但是因为要绘制更多的片元，因此会带来很大的性能开销。

在 ThreeJS 框架中有对应的例子实现[webgl_postprocessing_ssaa](https://threejs.org/examples/?q=ssaa#webgl_postprocessing_ssaa)。

<img src="https://newbieyoung.github.io/images/threejs-antialiasing-4.jpg">

<img src="https://newbieyoung.github.io/images/threejs-antialiasing-5.jpg">

不过这个例子中并不是采用的使用更高分辨率的方式来实现的，而是通过多次渲染场景，每次渲染时对摄像机进行轻微的抖动偏移，从而得到额外的颜色信息，最终会根据这些额外的颜色信息来进行防锯齿处理。

## 多重采样抗锯齿（MSAA）

在超采样抗锯齿技术基础上诞生了更为现代的技术，即`多重采样抗锯齿（Multisample Anti-aliasing）`。

<img src="https://newbieyoung.github.io/images/threejs-antialiasing-6.png">

如图所示多重采样所做的正是将单一的采样点变为多个采样点，我们不再使用像素中心的单一采样点，取而代之的是以特定图案排列的4个采样点，我们将用这些子采样点来决定像素的覆盖度；当然这也意味着颜色缓存区的大小会随着子采样点的增加而增加；

采样点的数量并不是固定的，更多的采样点能带来更精确的覆盖率，而最终的像素颜色将由片元本身的颜色和覆盖率共同决定。

<img src="https://newbieyoung.github.io/images/threejs-antialiasing-7.png">

多重采样需要硬件的支持而且多采样点也会带来一定的性能问题，另外还不能和`延迟渲染`很好的配合。

## 后处理抗锯齿（PPAA）

鉴于多重采样的不足，各种后处理抗锯齿技术（Post Process Antialiasing）发展了起来，比如`FXAA`和`SMAA`。

`快速近似抗锯齿（FXAA）`是基于边缘检测的抗锯齿算法，依赖于像素信息的变化，可以参考 [mattdesl/glsl-fxaa](https://github.com/mattdesl/glsl-fxaa) 的实现：

- 先是根据片元位置获取其周围片元的位置；

<img src="https://newbieyoung.github.io/images/threejs-antialiasing-8.png">

示意图如下：

<img src="https://newbieyoung.github.io/images/threejs-antialiasing-9.jpg">

- 然后根据位置获取颜色值；

<img src="https://newbieyoung.github.io/images/threejs-antialiasing-10.jpg">

- 然后根据颜色值计算亮度；

<img src="https://newbieyoung.github.io/images/threejs-antialiasing-11.jpg">

同时计算最大亮度和最小亮度，边缘变化越明显则局部差异越大（`luma`是标准灰度向量，亮度通过颜色向量和标准灰度向量的点乘计算得到）。

- 最后就是很玄学的抗锯齿处理的颜色计算了。

<img src="https://newbieyoung.github.io/images/threejs-antialiasing-12.jpg">

作者提供了 [在线DEMO](http://mattdesl.github.io/glsl-fxaa/demo/) 对比优化前后的效果。

在 ThreeJS 框架中对于`FXAA`和`SMAA`都是对应的例子实现：

- [webgl_postprocessing_fxaa](https://threejs.org/examples/?q=fxaa#webgl_postprocessing_fxaa)；

<img src="https://newbieyoung.github.io/images/threejs-antialiasing-13.jpg">

- [webgl_postprocessing_smaa](https://threejs.org/examples/?q=fxaa#webgl_postprocessing_fxaa)；

<img src="https://newbieyoung.github.io/images/threejs-antialiasing-14.jpg">

但是当我依葫芦画瓢在微信小游戏中使用`FXAA`和`SMAA`时却出现了很尴尬的情况：

<img src="https://newbieyoung.github.io/images/threejs-antialiasing-15.jpg">

虽然解决了魔方外部边缘的细小锯齿，但是却给魔方内部边缘带来了更严重的锯齿。

这里我只能甩锅给微信小游戏的渲染引擎了！

## 参考资料

- [LearnOpenGL CN 抗锯齿](https://learnopengl-cn.github.io/04%20Advanced%20OpenGL/11%20Anti%20Aliasing/)
- [抗锯齿技术之FXAA](http://www.selfgleam.com/fxaa.html)
- [现代抗锯齿技术——PPAA中的新星SMAA](https://www.qiujiawei.com/antialiasing/)
- [Fast Approximate Anti-Aliasing](https://www.geeks3d.com/20110405/fxaa-fast-approximate-anti-aliasing-demo-glsl-opengl-test-radeon-geforce/)









