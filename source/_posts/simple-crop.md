---
title: 全平台（Vue、React、微信小程序）任意角度旋转 图片裁剪组件
date: 2019-05-16 17:14
tags: [图片裁剪, React, Vue, 微信小程序]
categories: [工具/组件/工程化]
author: Young
---

`SimpleCrop` 目前是`全网唯一`支持裁剪图片任意角度旋转、交互体验`媲美原生客户端`的`全平台`图片裁剪组件。

项目地址：[https://github.com/newbieYoung/Simple-Crop](https://github.com/newbieYoung/Simple-Crop)。

## 特性及优势

和目前流行的图片裁剪组件相比，其优势在于以下几点：

- 裁剪图片支持任意角度旋转；
- 支持 Script 标签、微信小程序、React、Vue 等多种开发模式；
- 支持移动和 PC 设备；
- 支持边界判断、当裁剪框里出现空白时，图片自动吸附至完全填满裁剪框；
- 移动端缩放以双指中心为基准点；
- 交互体验媲美原生客户端。

<!--more-->

## 示例

### 微信小程序示例

![微信小程序示例](https://newbieyoung.github.io/images/simple-crop-16.jpg)

### 移动端示例

![移动端示例](https://newbieyoung.github.io/images/simple-crop-0.jpg)

> 左侧是 IOS 系统相册中原生的图片裁剪功能，右侧为 SimpleCrop 移动端示例。

可以扫描二维码体验：

![移动端示例二维码](https://newbieyoung.github.io/images/simple-crop-1.png)

或者访问以下链接：

[https://newbieyoung.github.io/Simple-Crop/examples/test-2.html](https://newbieyoung.github.io/Simple-Crop/examples/test-2.html)

### PC 示例

![PC 示例](https://newbieyoung.github.io/images/simple-crop-11.jpg)

链接如下：

[https://newbieyoung.github.io/Simple-Crop/examples/test-1.html](https://newbieyoung.github.io/Simple-Crop/examples/test-1.html)

## 关键实现

要实现`任意角度旋转`、`双指中心缩放`、`边界判断`、`自动吸附`等功能，关键点如下：

### 1、屏幕坐标系和变换基准点

在裁剪图片场景中，存在两个坐标系，其一是裁剪图片所代表的实际尺寸坐标系，其二是裁剪框显示到屏幕上所代表的屏幕坐标系；后续进行 transform 变换计算和位置判断时，为了计算方便，需要把裁剪图片的尺寸以及位置从实际坐标系转换为屏幕坐标系。另外当对裁剪图片进行 transform 变换时，变换基准点默认为其中心点，对应 CSS 的 transform-origin 为 50% 50%。

### 2、获取实时坐标

首先需要实时获取裁剪图片进行 CSS Transform 变换后的新坐标，只有在实时获取变换后的新坐标的前提下才能结合裁剪框坐标进行越界、吸附等判断；

![获取实时坐标](https://newbieyoung.github.io/images/simple-crop-10.jpg)

在计算 CSS Transform 变换后的新坐标时需要注意选取的屏幕坐标系和 CSS Transform 坐标系的差别，比如示例中以黑色边框中心为坐标原点，水平向左为 X 轴正方向，垂直向上为 Y 轴正方向；但是 CSS Transform 的坐标系垂直向下为 Y 轴正方向和上述规定的坐标系 Y 轴正方向是相反的，因此在获取 CSS Transform 变换矩阵之后求实时坐标时还需要进行镜像变换。

详细计算过程可以查看 [CSS3 2D Transform Matrix](https://newbieweb.lione.me/2019/04/29/css-2d-transform-matrix/)。

### 3、旋转适配缩放

裁剪图片任意角度旋转时需要进行适当的放大才能保证裁剪框不超出，因此就需要先计算裁剪框哪些点超出，然后根据超出的点计算刚好包含的放大倍数。

![旋转适配放大](https://newbieyoung.github.io/images/simple-crop-5.gif)

当两个矩形位置关系任意变换时计算相互之间有哪些点超出有两种方案：

其一：

![判断夹角和](https://newbieyoung.github.io/images/simple-crop-3.jpg)

> 图中左侧红色矩形代表裁剪图片，黑色矩形代表裁剪框，如图所示裁剪框顶点 A 超出了裁剪图片。

连接矩形四个顶点和判断点，然后计算四条连线之间的夹角，如果夹角之和小于 360 度，那么该判断点在矩形外；反之如果夹角之和等于 360 度，那么该判断点在矩形内。

```
a1 + a2 + a3 + a4 < 360
b1 + b2 + b3 + b4 = 360
```

其二：

![计算投影](https://newbieyoung.github.io/images/simple-crop-4.jpg)

> 图中黑色矩形表示裁剪图片，点 A 表示裁剪框中超出裁剪图片的某个顶点。

连接矩形中心点和判断点，然后计算中心点和判断点向量在矩形边框向量上的投影长度（L1、L2），只要两个投影长度中有任意投影长度大于其投影边框长度（H1、H2）的一半即说明该点在矩形外。

另外还可以根据投影长度和其投影边框长度的比例计算出矩形恰好包含该点的放大系数，也就是示例图中的 S 变量。

最后旋转图片时除了要进行适当的放大，保证裁剪框不超出以外，还可以在裁剪图片中心点没有变动时进行适当的缩小，去掉多余间隙，进一步提升交互体验。

![旋转适配缩小](https://newbieyoung.github.io/images/simple-crop-12.gif)

缩小系数的计算原理和放大系数的计算原理类似，均是连接判断点和中心点，然后根据边框投影长度计算。

![计算缩小倍数](https://newbieyoung.github.io/images/simple-crop-13.jpg)

> 大矩形为裁剪图片，小矩形表示裁剪框，O 表示裁剪图片中心点。

### 4、双指中心位移

由于默认裁剪图片的变换基准点为其中心点，这么处理虽然计算方便，但是会对双指缩放造成一定的困难；因为双指操作时双指中心并不一定是裁剪图片中心。

解决方案需要先求出两个不同基准点的位移差，然后在进行缩放变换之后再进行位移变换。

### 5、缩放适配变换

在旋转裁剪图片时可以对其进行适当得放大和缩小从而保证裁剪框不会超出裁剪图片；但是在双指操作缩放裁剪图片却不能这么做，因为适配缩放会和用户的操作缩放冲突，因此需要采用移动裁剪图片的方式保证裁剪框不超出裁剪图片。

![缩放适配变换](https://newbieyoung.github.io/images/simple-crop-6.gif)

当裁剪图片进行位移变换之后可以包含裁剪框，就只需要计算位移向量；

![缩放适配情形一](https://newbieyoung.github.io/images/simple-crop-7.jpg)

> 红色矩形为裁剪图片，黑色矩形为裁剪框。

但是还有一种情况即裁剪图片进行位移变换之后不能包含裁剪框，如下：

![缩放适配情形二](https://newbieyoung.github.io/images/simple-crop-8.jpg)

> 红色实线矩形为裁剪图片，黑色矩形为裁剪框，红色虚线矩形为进行放大之后恰好包含裁剪框的裁剪图片。

此时说明用户的操作缩放超出了组件的合法限制范围，可以加入适配缩放了；这时候就需要先计算裁剪图片恰好包含裁剪框的放大系数，然后再进行位移变换。

### 6、其它

在整个实现过程中还涉及到大量 Canvas API 操作，需要注意以下几点；

- 使用微信小程序 type 2d Canvas API 时遇到了一些坑。比如：在 Android 中 Canvas 不支持 transform style，而在 IOS 中 Canvas 设置 width 和 height 属性之前需要先设置其 style 的 width 和 height 否则不会生效；还有 wx.chooseImage 选择原图时，如果图片过大会 Crash，在 IOS 中易现；

- Canvas 元素的尺寸在浏览器中是有最大限制的，超出后浏览器会报警告canvas area exceeds the maximum limit 而且绘制的结果为空白；

- 使用 Image 标签展示图片时浏览器会忽视图片的方向角数据，但是使用 Canvas API 绘制时又会考虑方向角；在处理图片时需要注意不同标签间的差异。建议的方案是使用 [Exif.js](https://github.com/exif-js/exif-js) 获取图片元数据，然后借鉴 [JavaScript-Load-Image](https://github.com/blueimp/JavaScript-Load-Image) 中的处理方法即可。

![处理图片方向角](https://newbieyoung.github.io/images/simple-crop-15.jpg)