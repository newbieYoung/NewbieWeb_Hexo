---
title: 一次网页动画踩坑实录
date: 2016-05-23 14:38
tags: [网页动画]
categories: [前端开发]
author: Young
---

在富途从事前端开发以来，真正意义上就做过两次网页动画，但是每次的开发经历都很类似；刚开始感觉很简单，后来就不知不觉就掉坑里边了......为了下次能有所改观，我准备从头到尾仔细的总结一下。

需要注意的是以下所述只是我个人在开发过程中的感受和总结，有可能不完善甚至是错误的；另外补充了一些感觉有一定道理但是由于一些原因没有亲自去验证的观点，主要是为了记录下来，方便今后去验证。

<!--more-->

## 设计阶段

动画一般由专门的设计人员来设计，我以前以为这个阶段和开发人员没什么关系，现在发现过往的种种痛苦经历有很大一部分是因为这种想法导致的。

如果有下次，我一定在这个阶段就好好考虑一下几个问题：

### 1、有没有什么动画元素是可以去掉的？

从老虎机的基本功能上来说其实三竖行就足够了（不清楚实际生活中的老虎机一般有几个竖行，这里只是从基本功能角度来考虑）。

当前积分以及可玩次数后边的数字能否使用简单的网页字体并去掉跳动效果（手机设备一般比较小，就算这里实现了动画效果，对于用户来说基本也处于隐藏状态）。

<img src="https://newbieyoung.github.io/images/animation-optimize-0.png"/>

### 2、有没有什么动画效果是可以更换的？

有些动画效果会对浏览器产生严重的渲染负担（比如滚动），就抽奖而言是否可以换成更简单的动画而不一定采用老虎机的形式。

### 3、有没有什么动画元素用目前常用技术是很难实现的或者很难用目前的技术比较高效的实现？

下边按钮上有两个动效，一个是边框时暗时亮，然后就是按钮背景一直在向左侧移动，本来这两个动效可以用CSS3很简单很高效的实现，但是这个按钮不规则的形状导致很难通过CSS3构造，只能使用gif和图片，从而使资源显著增大，然后由于gif图片会出现留白、边框锯齿等情况也使实现效果不理想。

<img src="https://newbieyoung.github.io/images/animation-optimize-1.png"/>

### 4、有没有什么动画元素过于复杂（颜色、纹理、尺寸）需要简化的？

一般来说不建议动画元素的颜色、纹理过于复杂，同时也不建议动画元素的尺寸过于大，因为这会导致动画资源太大用户浏览时需要等待很长时间，同时增加浏览器的负担（具体原因在开发阶段解释）。

总而言之，在设计阶段，作为开发人员，其实是需要全程参与的，不断的进行`预开发`，从而提前发现一些不合理以及可能会踩坑的地方，而不是简单的认为设计成什么样子就实现成什么样子。

## 开发阶段

网页动画的实现和前端技术以及浏览器息息相关；那么要实现高性能的网页动画，就必须要熟悉浏览器机制以及各种动画方案以及其适用的场景，同时还需要了解简单的物理运动规律以及常用的缓动函数。

### 1、简单的物理运动与缓动函数

在现实中，事物在运动时可能加速或减速，如果加速度一定那么就是匀变速运动；如果加速度不固定，那么就是变加速运动；在做动画时，应该尽可能的遵循这些规律，从而产生更好的体验效果。

在CSS3中可以通过 animation-timing-function 和 transition-timing-function 来实现缓动；但是如果是其它没有提供相应功能的动画形式，比如 JS 动画，则需要采用其它第三方类库（其实也可以自己来实现）。
 
介绍一种曾今使用过的类库 [bezier-easing](https://github.com/gre/bezier-easing)，在项目中可以通过 `npm install bezier-easing` 来安装，这个类库实现了在［0，1］的范围内，数值从［0，1］的缓动变化，在实际使用时需要根据实际情况处理变化区间。

### 2、浏览器机制

#### 浏览器渲染

<img src="https://newbieyoung.github.io/images/animation-optimize-2.png"/>
<img src="https://newbieyoung.github.io/images/animation-optimize-3.png"/>

关于浏览器渲染有很多细节，具体可以查看以下链接：

+ [渲染性能](https://developers.google.com/web/fundamentals/performance/rendering/) 
+ [无线性能优化：Composite](http://taobaofed.org/blog/2016/04/25/performance-composite/)
+ [Rendering: repaint, reflow/relayout, restyle](http://www.phpied.com/rendering-repaint-reflowrelayout-restyle/) 
+ [CSS Triggers](https://csstriggers.com/)

一般来说动画尽可能使用 transform、opacity 这两个属性。

#### setTimeout、setInterval、requestAnimationFrame

使用 requestAnimationFrame（回调函数会被传入一个 DOMHighResTimeStamp 参数，这个参数指示当前被 requestAnimationFrame 序列化的函数队列被触发的时间）时，一般推荐基于时间来制作动画，如果某些设备渲染性能很差，则可以根据情况改为基于帧数来制作动画。

### 3、动画方案

- JavaScript
- CSS3
- Canvas
- GIF
- Video
- WebGl
- SVG

## 调试阶段

其实我一直觉得编程就是发现问题解决问题，所以调试能力是很重要的一环，对前端来说最棒的调试工具莫过于Chrome浏览器的dev-tool了，虽然我没有找到系统详细介绍这款工具的教程，但是还是搜罗了一些文章，链接如下：

+ [How to Use the Timeline Tool](https://developers.google.com/web/tools/chrome-devtools/profile/evaluate-performance/timeline-tool) 
+ [Evaluating network performance](https://developer.chrome.com/devtools/docs/network) 

同时会在接下来通过一个简单的例子来说明我是如何通过 dev-tool 来发现问题并解决的。

<img src="https://newbieyoung.github.io/images/animation-optimize-4.jpg">

首先我通过发现很多 long frame 的 composite 是耗时最多的，接着在子线程中有大量的 rasterize paint 操作，这个操作主要是 GPU 加速时，CPU 往 GPU 转移数据；PC 上由于 Chrome 会开启多个子线程所以不会出现卡顿的现象，但是在移动端 Chrome 只会开启一个，这时候大量的 rasterize paint 操作挤在一起就会产生卡顿现象。

知道原因之后接下来需要定位元素了，通过 Paint profile 栏发现目标元素的尺寸是 26*26 ，同时 long frame 的时间间隔大致为 100 毫秒，就定位了老虎机头部灯。

<img src="https://newbieyoung.github.io/images/animation-optimize-5.jpg">

开发的时候我问设计要图片，但是她给的圆不是一个严格的圆，转动起来总有飘动的感觉，后来我就切出了两种灯的状态，然后一个个定位，然后就出现了大量的绝对定位元素，而且这些元素动画时透明度会不断变化，透明度的变化会被 Chrome 浏览器启动 GPU 加速，GPU 加速时会给当前创建一个单独的渲染层，导致渲染层增多而且会有大量转移操作，在子线程只有一个的移动端就会出现卡顿的现象了，改进措施就是按以前的方案，用严格圆替代单独小点。

<img src="https://newbieyoung.github.io/images/animation-optimize-6.jpg">

还做了一个优化措施，把动画元素和非动画元素分开了，以前动画元素和非动画元素的父元素是同一个 dom 元素，这个 dom 元素有一个很大的背景，我发现每一帧都会绘制这个背景，所以就把它们分开了，但是效果不太明显。

<img src="https://newbieyoung.github.io/images/animation-optimize-7.jpg">

不断优化直到 iphone4 没有卡顿现象后，timeline 概况如下，longframe 明显减少，另外之所以启动阶段有大量 longframe 时因为启动时大量动画同时开启而且还是执行 JS 脚本处理事件等；

<img src="https://newbieyoung.github.io/images/animation-optimize-8.jpg">

最后之所以还有一些 long frame 是因为代码里边有使用定时器，但是影响不大也就没有再改了。






