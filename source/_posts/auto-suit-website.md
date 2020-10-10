---
title: 移动端网页绝对自适应方案总结
date: 2017-07-31 20:53
tags: [网页自适应]
categories: [前端开发]
author: Young
---

网页自适应是个涵盖很广的话题，这里我只简单总结下`移动端网页绝对自适应`，简单来说如果设计师提供给你的设计稿是750px宽，此时里边有一张图片宽高均为375px；那么在IPhone6 Plus手机里边屏幕宽为414px时该图片的渲染宽度应该为207px，这种`保证元素尺寸、字体大小在屏幕中所占比例大小始终不变`的方式我称之为`移动端网页绝对自适应`。

目前来说我总结了大概有如下五种方式来实现`移动端网页绝对自适应`。

<!--more-->

## 一、百分比

在CSS中很多属性都可以使用百分比值，正确使用百分比有两个关键点：

#### 1、找到当前元素的参照元素；

某个元素的参照元素不能简单的理解成父元素，还和其定位有关系，比如：

- `静态定位元素`、`相对定位元素`的参照元素一般是其父元素；
- `绝对定位元素`参照元素一般是`离它最近`的绝对定位、相对定位或者固定定位的父元素，如果不存在则为视口；
- `固定定位元素`的参照元素一般是视口。

#### 2、找到当前属性的参照属性；

在CSS中很多属性都可以被设置为百分比数值，但是其意义确会有很多不同；总结如下：

- `padding`百分比是相对于`参照元素的宽度`而言；
- `margin`百分比是相对于`参照元素的宽度`而言；
- `width`百分比是相对于`参照元素的宽度`而言；
- `left`、`right`百分比是相对于`参照元素的宽度`而言；
- `height`百分比是相对于`参照元素的高度`而言；
- `top`、`bottom`百分比是相对于`参照元素的高度`而言；
- `line-height`百分比是相当于`元素自身文字大小`而言；
- `background-size`百分比是相当于`元素自身的宽高`而言；
- `border-radius`百分比是相当于`元素自身的宽高`而言；
- `transform`百分比是相当于`元素自身的宽高`而言；
- `background-position`百分比和其它的百分比单位表现都不一样，具体可以用以下公式计算：

```
positionX = (容器的宽度-图片的宽度) * percentX;
positionY = (容器的高度-图片的高度) * percentY;
```

[示例](https://newbieyoung.github.io/CSS_learn/percent.html)

以上只总结了较为常见的属性，实际还存在很多其它属性可以被设置为百分比，并可能存在你意想不到的意义。

vw和vh应该就是百分比的一种变种，只不过所有元素的参照元素都是视口，参照属性都是视口的宽高而已；

另外还要注意在一些Android设备中软键盘弹出会影响视口的尺寸（具体是高度变小），因此会对以百分比或者vw、vh为单位的属性造成影响；而且并不能通过强制设置视口尺寸来解决这个问题，因为在一些Android设备中会对脚本代码设置的视口尺寸再进行一次校验来保证视口尺寸比例不发生变化，具体情况请查看[示例](https://newbieyoung.github.io/SomeBugs/bug-about-vh-vw-in-android/demo0.html)。

#### 小结

百分比方式在处理文字大小的自适应上会比较麻烦，另外在处理图片的自适应也有问题，只能使用img标签的方式来实现，这样就会导致不能把图片合成精灵图（就目前来说把小图片合成精灵图还是有必要的）；

另外设计稿一般都是固定数值的，所以重构页面时你得自己计算出百分比数值，虽然计算量不大，其实也是有那么一点点麻烦的。

### 二、媒体查询

媒体查询方式的问题在于只能处理特定状态，而“绝对自适应”其实可以理解成有无限多的状态。

### 三、REM

REM方式我一般会在页面里边加入如下脚本；

	(function (doc, win) {
		var $body = document.querySelector('body');
        var docEl = doc.documentElement,
                resizeEvt = 'orientationchange' in window ? 'orientationchange' : 'resize',
                recalc = function () {
                    var clientWidth = docEl.clientWidth;
                    if (!clientWidth) return;
                    clientWidth = clientWidth>750?750:clientWidth;//防止被放大保证宽屏效果
                    //以宽750px为例子，并扩大100倍，尽可能的保证精度
                    docEl.style.fontSize = 100 * (clientWidth / 750) + 'px';
                    //缩放之后再显示页面，防止缩放过程被观察到，影响体验
                    $body.style.visibility = 'visible';
                };
        if (!doc.addEventListener) return;
        win.addEventListener(resizeEvt, recalc, false);
        doc.addEventListener('DOMContentLoaded', recalc, false);
    })(document, window);

需要注意的是脚本中`750`表示的是设计稿的宽度，后述脚本中的`640`等也是相同的意义。

其中关于扩大`100`倍的主要原因是如果这里没有扩大`100`倍，那么假设屏幕宽度是`400px`，那么html标签的font-size是`0.5333...px`，在某些浏览器只保留两位小数，此时实际有效值就变成了`0.53px`，如果CSS样式里边的值为`46rem`，最终结果就是`0.53*46=24.38px`；

但是如果扩大`100`倍，屏幕宽度还是`400px`，那么html标签的font-size是`53.33...px`，还是在这个保留两位小数的浏览器里边，此时为了保证整体大小不变，CSS样式里边的值缩小`100`倍为`0.46rem`，此时最终结果就是`53.33*0.46=24.53px`；

显然`24.53px`会比`24.38px`更精确一些。

REM方式理论上来说已经能解决问题了，但是实际上还是会有一些问题，最核心的问题在于数值的精度问题（不同浏览器处理浮点数时所保留的精度不一样），比如：

- 1、较小数值可能会因为精度问题导致实际渲染大小为0，比如边框等；
- 2、在Android系统里边有可能出现即使设置了height=line-height时文字依然不能完全垂直居中的情况，如果使用REM，font-size、height、line-height均是小数，精度问题会导致问题更明显。

### 四、meta inital-scale

上述方式归根到底都属于一种间接缩放的方法，需要注意的WebView早就提供一个直接缩放整个页面的方法，那就是meta标签中inital-scale属性，也就说可以控制这个属性来缩放页面，来达到绝对自适应；

控制脚本和REM控制脚本类似：

	(function (doc, win) {
	    var docEl = doc.documentElement,
	            resizeEvt = 'orientationchange' in window ? 'orientationchange' : 'resize',
	            recalc = function () {
	            	var clientWidth = docEl.clientWidth;
	                if (!clientWidth) return;
                    clientWidth = clientWidth>750?750:clientWidth;//防止被放大保证宽屏效果
	                document.querySelector('meta[name=viewport]').setAttribute('content','width=750, initial-scale='+clientWidth/750+', maximum-scale=1.0, user-scalable=0');
            		document.querySelector('body').setAttribute('style','visibility:visible;');
	            };
	    if (!doc.addEventListener) return;
	    doc.addEventListener('DOMContentLoaded', recalc, false);
	})(document, window);

这种方式异常的简单方便（谁用谁知道），如果REM方式是`无脑自适应`的话，那么这种方式可以被称为`脑残自适应`；这种方式基本不存在REM的精度问题，移动端细边框等问题完全不需要考虑；唯一需要你做的就是认认真真的把设计稿里边的值复制到页面中，然后再加上上述脚本；

起初我以为找到了完美的解决方案，但是后来实践的时候发现这个方式的缺点也很明显；

- 1、存在极少数的设备不支持meta标签的initial-scale属性（少到可以忽略不计的那种）；
- 2、脚本中所涉及meta标签中的那些属性不能再被其它脚本所处理（貌似普通网页也不会随便操作mata标签）；
- 3、由于这个方式是缩放整个页面的方式来实现自适应，那么页面里边所有的样式都得遵循这种方式（就实际情况来看局限性很大），另外渲染性能也偏低（如果页面存在大量动画，不建议采用这种方式）。

### 五、transform scale

受上一种方式启发，缩放除了meta标签的initial-scale方式，CSS3的tranform也可以做到，而且meta标签的initial-scale是缩放整个页面，导致会和其它样式冲突，那可不可以专门定义一块区域缩放，其它区域不缩放呢？

	(function (doc, win) {
        var docEl = doc.documentElement,
                resizeEvt = 'orientationchange' in window ? 'orientationchange' : 'resize',
                recalc = function () {
                    var clientWidth = docEl.clientWidth;
                    if (!clientWidth) return;
                    var $bodyContent = document.querySelector('.body-content');
                    var $body = document.querySelector('body');
                    var percent = clientWidth/750;
                    percent = percent>1?1:percent;//防止被放大保证宽屏效果
                    var othersHeight = $body.clientHeight-$bodyContent.clientHeight;//计算其它元素的高度
                    $bodyContent.setAttribute('style','width:750px;transform:scale('+percent+');transform-origin:0 0;-webkit-transform:scale('+percent+');-webkit-transform-origin:0 0;margin:auto;');
                    var lastStyle = $bodyContent.getAttribute('style');
                    var articleHeight = $bodyContent.clientHeight*percent;
                    $body.setAttribute('style','visibility:visible;height:'+(articleHeight+othersHeight)+'px;');
                };
        if (!doc.addEventListener) return;
        doc.addEventListener('DOMContentLoaded', recalc, false);
    })(document, window);

上述脚本就实现了上述的设想，解决了meta标签的initial-scale方式的第一个和第二个缺陷；

需要注意的是脚本中还额外处理了页面的高度，因为CSS3的tranform方式只影响了元素视觉效果，不会影响元素的占位尺寸，比如如果元素本身`1000px`高，缩放一半后看起来只有`500px`高，但是高度占位还是`1000px`，导致如果页面出现滚动条，缩放之后滚动高度还是不变，此时需要设置页面高度为缩放之后的高度。

除了高度问题，这种方式还需要注意一些问题：

- 1、tansform会导致其内部元素的fixed定位失效，~~变为absolute定位~~；

[示例](https://newbieyoung.github.io/SomeBugs/bug-about-transform-fixed/demo0.html)；

fixed定位失效的真正原因在于transform影响了fixed定位元素的堆叠上下文，简单来说fixed定位的元素，如果其祖先元素存在非none的transform值，那么该元素将相对于设定了transform的祖先元素定位，而不是原本相对于视口。

所谓堆叠上下文这个概念可以看看这篇文章[层叠顺序（stacking level）与堆叠上下文（stacking context）知多少？](http://www.cnblogs.com/coco1s/p/5899089.html)。

fixed定位失效的问题可以通过把元素单独抽离出来并在外部多添加一层，外部容器层使用fixed定位，内部元素使用absolute定位来解决。

- 2、缩放区域得清除浮动，否则会导致高度计算异常；

- 3、全屏布局不适用这种方法，具体原因如下：

`全屏布局`是指页面刚好在一个屏幕的范围内，这种布局需要既需要自适应宽度的变化，还需要自适应高度的变化，一般方法为让页面主内容区域在屏幕中居中，内容元素自适应宽度，元素之间的间隙自适应高度；
某些时候甚至需要使用`media query`针对不同的情况设置不同的值。

- 4、嵌入iframe时，需要在父页面中设置缩放比；

因为IOS中嵌入在iframe中的页面的clientWidth是由页面本身的内容宽度决定的和父页面的iframe的宽度没有关系，IOS的这种机制会导致缩放比计算异常。

[示例](https://newbieyoung.github.io/SomeBugs/bug-about-iframe-clientwidth-in-ios/container.html)；

<img src="https://newbieyoung.github.io/images/auto-suit-website-0.png">

### 总结

上述这些方式仅仅我在坚持开发所谓的`绝对自适应`网页过程中慢慢得到的一些经验总结，就目前看来没有一种十全十美的方式，具体情况具体分析咯，毕竟方法是死的，人是活的。


