---
title: 你可能不需要知道的 IOS 的一些坑
date: 2018-07-05 01:43
tags: [IOS,Bug]
categories: [前端开发]
author: Young
---

重要不是记住有哪些坑，而是锻炼从坑里边爬出来的能力......

<!--more-->

### 在 IOS 中 iframe 中嵌入页面的 clientWidth 属性是由嵌入页面内容以及外部 iframe 的宽度共同决定的

具体来说在 IOS 中 iframe 中嵌入页面的内容宽度大于 iframe 容器宽度时，iframe 嵌入页面的 clientWidth 属性等于页面的内容宽度；否则等于 iframe 容器宽度；

例子如下：
[https://newbieYoung.github.io/SomeBugs/bug-about-iframe-clientwidth-in-ios/container.html](https://newbieYoung.github.io/SomeBugs/bug-about-iframe-clientwidth-in-ios/container.html)；

<img src="https://newbieyoung.github.io/images/ios-pits-0.jpg">

使用 IPhone 手机访问链接则会看到上述图片左侧部分，使用 Android 手机访问链接则会看到上述图片右侧部分；

灰色部分为宽度不同的 iframe 元素，分别嵌入不同的页面；

<img src="https://newbieyoung.github.io/images/ios-pits-1.jpg">

灰色部分中的数字为嵌入页面的 clientWidth 属性；

<img src="https://newbieyoung.github.io/images/ios-pits-2.jpg">

其它颜色部分为嵌入页面中的元素，里边的数字为其宽度；

<img src="https://newbieyoung.github.io/images/ios-pits-3.jpg">

从上述例子我们不难看出 Android 中 iframe 中嵌入页面的 clientWidth 由 iframe 的宽度决定的，但是在 IOS 中则略有不同。

某些页面如果是根据 clientWidth 动态计算缩放比来实现自适应的话，在嵌入到 iframe 中时需要注意这个问题。

### 在 IOS 中软键盘弹出会改变页面滚动条位置，软键盘收起时并不会复原

例子如下：
[https://newbieyoung.github.io/SomeBugs/bug-about-keyboard-in-ios/demo0.html](https://newbieyoung.github.io/SomeBugs/bug-about-keyboard-in-ios/demo0.html)

在例子中使用脚本修复了这个问题：

<img src="https://newbieyoung.github.io/images/ios-pits-8.jpg">

页面表现正常：

<video style="height:500px;" controls="controls" preload="auto" webkit-playsinline="true" playsinline="true" src="https://newbieyoung.github.io/images/ios-pits-9.mp4"></video>

如果去掉修复脚本，就会出现异常，具体表现是`input`输入框位置并不会复原。

<video style="height:500px" controls="controls" preload="auto" webkit-playsinline="true" playsinline="true" src="https://newbieyoung.github.io/images/ios-pits-10.mp4"></video>

### 在 IOS 中某些情况下滚动页面时会出现图片渲染异常

<video style="height:500px;" controls="controls" preload="auto" webkit-playsinline="true" playsinline="true" src="https://newbieyoung.github.io/images/ios-pits-4.mp4"></video>

例子如下：
[https://newbieYoung.github.io/SomeBugs/bug-about-img-render/demo.html](https://newbieYoung.github.io/SomeBugs/bug-about-img-render/demo.html)；

使用 IPhone 手机访问链接然后向下滚动页面之后再向上滚动页面就会发现背景图片消失了；

解决这个问题有以下几种方案：

+ 方案一：去掉`body.html`中的`overflow-x:hidden`；

<img src="https://newbieyoung.github.io/images/ios-pits-6.jpg">

[https://newbieYoung.github.io/SomeBugs/bug-about-img-render/demo1.html](https://newbieYoung.github.io/SomeBugs/bug-about-img-render/demo1.html)；

+ 方案二：去掉`body.html`中的`height:100%`由局部滚动改为全局滚动；

<img src="https://newbieyoung.github.io/images/ios-pits-5.jpg">

[https://newbieYoung.github.io/SomeBugs/bug-about-img-render/demo2.html](https://newbieYoung.github.io/SomeBugs/bug-about-img-render/demo2.html)；

+ 方案三：去掉`.bg`中的`position:absolute`为了使页面看起来类似，需要设置`margin-top:-800px`让`.content`向上移动；

<img src="https://newbieyoung.github.io/images/ios-pits-7.jpg">

[https://newbieYoung.github.io/SomeBugs/bug-about-img-render/demo3.html](https://newbieYoung.github.io/SomeBugs/bug-about-img-render/demo3.html)；

由此可见这个异常情况是由局部滚动、overflow 以及背景元素和滚动元素的定位关系共同决定的，因此解决办法并不局限于上述三种，还有诸如把背景设置在 body 或者把背景元素直接设置在滚动元素上等方法；

突然感觉能遇到要连踩这么多坑的异常也是件挺幸运的事情...虽然几个小时就这么过去了...至于这种渲染异常最终的原因暂时就不太清楚了!