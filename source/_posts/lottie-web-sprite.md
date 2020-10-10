---
title: 精灵图在 Lottie Web 动画中的应用
date: 2018-12-13 21:43
tags: [Lottie,网页动画]
categories: [前端开发]
author: Young
---

`Lottie`是一套跨平台的平面动画解决方案；设计师使用`AE`设计动画，然后通过插件将动画导出成定义好的`json`文件；之后开发人员只需要依赖这个 json 文件和对应平台的`Lottie`动画库就可以很方便的在相应平台中实现动画了。

## 普通示例

<img src="https://newbieyoung.github.io/images/lottie-web-sprite-0.gif">

普通示例在线预览链接为 [https://newbieyoung.github.io/lottie-web-sprite/normal.html](https://newbieyoung.github.io/lottie-web-sprite/normal.html)；

源代码在 [lottie-web-sprite](https://github.com/newbieYoung/lottie-web-sprite/blob/master/normal.html) 项目中。

<!--more-->

在 Web 平台则需要使用 [Lottie Web](https://github.com/airbnb/lottie-web)；

可以去 [https://cdnjs.cloudflare.com/ajax/libs/bodymovin/5.4.2/lottie.min.js](https://cdnjs.cloudflare.com/ajax/libs/bodymovin/5.4.2/lottie.min.js) 下载压缩后的源代码，或者去 [https://cdnjs.cloudflare.com/ajax/libs/bodymovin/5.4.2/lottie.js](https://cdnjs.cloudflare.com/ajax/libs/bodymovin/5.4.2/lottie.js) 下载未压缩源代码;

设计师使用 AE 导出如下资源：

<img src="https://newbieyoung.github.io/images/lottie-web-sprite-1.jpg">

格式化其 json 文件内容可以看到：

<img src="https://newbieyoung.github.io/images/lottie-web-sprite-2.jpg">

其中需要注意`w`、`h`、`assets`；

- `w`表示动画画布宽度；
- `h`表示动画画布高度；
- `assets`表示`images`目录中的图片资源信息；

知道动画画布宽度和高度之后，我们就可以设置页面中的动画容器宽度和高度了；

```CSS
div{
	width:280px;
	height:280px;
}
```

```HTML
<div id="animation"></div>
```

动画容器为`DIV`元素，其`ID`属性为`animation`，其宽度和高度均为`280`；

之后需要引入框架源代码，这里使用未压缩版本，方便调试；

```HTML
<script src="./lib/lottie.js"></script>
```

引入代码之后只需要调用`lottie.loadAnimation()`就可以实现动画了；

```JS
var animation = document.querySelector('#animation');
lottie.loadAnimation({
    container: animation,
    renderer: 'canvas',
    loop: true,
    autoplay: true,
    path: './animation/data.json'
});
```

- `container`表示动画容器；
- `renderer`表示渲染器，目前支持的渲染器有`html`、`svg`、`canvas`，较常用的是`svg`和`canvas`；
- `loop`表示动画循环播放；
- `autoplay`表示动画自动播放；
- `path`则表示`json`文件路径。

到此整个动画实现就完成了。

初一看起来没有任何问题，但是当我们打开浏览器调试工具，查看资源加载情况时就会发现有点问题：

<img src="https://newbieyoung.github.io/images/lottie-web-sprite-3.jpg">

设计师通过 AE 导出的资源里边存在图片资源，而这些图片资源没有经过任何处理是单独加载的，这会导致加载的资源数量过多，特别是当一个页面存在多个独立动画时。

这个问题可以通过把图片资源整合到 json 文件中的方式解决，最终设计师导出的资源就只有一个 json 文件，不过这种方式需要设计师在导出资源时进行处理，而且需要先把图片转换成矢量图。

对于我来说这种方式会显得有点麻烦，首先我并不清楚怎么把图片资源转换成矢量图然后整合 json 文件，甚至连 AE 都没安装过；对于一个本身并不清楚而且也没有实际操作经验的问题，自然也就很难跟设计师讲清楚了。

那有没有其它我擅长的方式呢？

## 精灵图示例

<img src="https://newbieyoung.github.io/images/lottie-web-sprite-0.gif">

精灵图示例在线预览链接为 [https://newbieyoung.github.io/lottie-web-sprite/sprite.html](https://newbieyoung.github.io/lottie-web-sprite/sprite.html)；

源代码在 [lottie-web-sprite](https://github.com/newbieYoung/lottie-web-sprite/blob/master/sprite.html) 项目中。

<img src="https://newbieyoung.github.io/images/lottie-web-sprite-10.jpg">

打开浏览器调试工具，可以清楚的看到图片资源都被整合在一张精灵图中了。

### 源代码解析及思路

怎么实现呢？在没有思路的时候看看源代码好了。

<img src="https://newbieyoung.github.io/images/lottie-web-sprite-4.jpg">

<img src="https://newbieyoung.github.io/images/lottie-web-sprite-5.jpg">

<img src="https://newbieyoung.github.io/images/lottie-web-sprite-6.jpg">

从`loadAnimation`方法开始，到`animItem.setParams(params)`在设置并解析参数，最后在`assetLoader.load`方法中加载 json 文件，加载完成之后执行`this.configAnimation.bind(this)`配置动画回调函数；

<img src="https://newbieyoung.github.io/images/lottie-web-sprite-7.jpg">

其中`this.animationData`即是解析 json 文件内容得到动画数据，其`assets`属性为图片信息数组；忽略其它流程，重点关注`this.preloadImages()`，在这个方法中会通过`assets`加载对应图片资源；

<img src="https://newbieyoung.github.io/images/lottie-web-sprite-8.jpg">

<img src="https://newbieyoung.github.io/images/lottie-web-sprite-9.jpg">

`loadAssets`方法会根据图片信息数组发送异步请求加载图片资源，然后生成`img`元素，整合到`ImagePreloader`对象的`images`属性中。

到这里虽然并没有弄清楚`Lottie`是怎么根据`json`文件实现动画的，但是可以清楚看到它是怎么解析`json`文件然后加载相应图片素材的；

根据上述流程怎么在`Lottie`中应用精灵图的思路也就渐渐清晰了，我们只要保证通过精灵图得到和上述流程中一样的`images`数组数据就可以了。

### 生成精灵图

可以用 [lia](https://github.com/cupools/lia) 生成精灵图；

用法很简单：

1. 使用`npm`全局安装`lia`；

```
npm i -g lia
```

2. 创建精灵图配置文件`sprite_conf.js`；

<img src="https://newbieyoung.github.io/images/lottie-web-sprite-11.jpg">

> 使用`lia init`命令可以自动生成配置文件，只不过因为需要使用自定义模版，所以这里手动创建。

- `src`表示图片素材路径匹配规则；
- `image`表示生成的精灵图的路径；
- `style`表示生成的图片素材和精灵图的位置关系数据文件的路径；
- `tmpl`表示图片素材和精灵图的位置关系数据文件模版。

3. 创建图片素材和精灵图的位置关系数据模版文件`template.ejs`；

<img src="https://newbieyoung.github.io/images/lottie-web-sprite-12.jpg">

4. 执行`lia`命令。

```
lia
```

就得到了精灵图以及位置关系数据文件。

<img src="https://newbieyoung.github.io/images/lottie-web-sprite-13.jpg">

最后需要把位置关系数据文件中的绝对路径改为相对路径。

### 更改 json 文件

<img src="https://newbieyoung.github.io/images/lottie-web-sprite-14.jpg">

更改 json 文件很简单只需要先读取内容，然后新增自定义字段，最后保存即可。

### 更改源代码

数据都准备完成了，接下来需要对`Lottie`的源代码进行兼容处理。

兼容处理之后的完整源代码在[https://github.com/newbieYoung/lottie-web-sprite/blob/master/lib/lottie-sprite.js](https://github.com/newbieYoung/lottie-web-sprite/blob/master/lib/lottie-sprite.js)。

首先在`configAnimation`方法中，如果解析的 json 数据存在`_sprite`属性，则使用`preloadSprite`方法从精灵图中获取图片素材，否则按以前的流程使用`preloadImages`方法从`assets`数组中获取图片素材；不管是哪种获取图片素材的方式，后续流程都保持不变。

<img src="https://newbieyoung.github.io/images/lottie-web-sprite-15.jpg">

`preloadSprite`和`preloadImages`两个方法的逻辑基本类似，除了前者使用自定义方法`loadAssetsFromSprite`，后者使用原生方法`loadAssets`。

<img src="https://newbieyoung.github.io/images/lottie-web-sprite-16.jpg">

在`loadAssetsFromSprite`方法中，先根据精灵图路径异步请求精灵图，然后根据位置关系从精灵图中动态获取相关图片素材，并设置到`images`属性中即可。

<img src="https://newbieyoung.github.io/images/lottie-web-sprite-17.jpg">

至此就可以在`Lottie`中应用精灵图了。








































