---
title: 聊一聊全景图
date: 2017-10-12 11:24
tags: [ThreeJS,WebGL,全景图]
categories: [3D技术]
author: Young
---

前段时间学习了 ThreeJS 项目里边关于全景图的案例之后，自己动手练习了一下，实现了两个全景图的例子，分别如下：

[WebGLRender 球型全景图](https://newbieyoung.github.io/Html_learn/webgl/demo34.html)

[WebGLRender 立方体全景图](https://newbieyoung.github.io/Html_learn/webgl/demo35.html)

[CSS3DRender 立方体全景图](https://newbieyoung.github.io/Html_learn/webgl/demo37.html)

`WebGLRender 球型全景图`由于贴图的原因导致南北极存在扭曲的情况，通过把球型全景图转换成立方体全景图可以改善这个问题，但是不能完全避免；

网络不好的情况下第一次打开可能会比较慢，因为全景图资源比较大。

实现原理比较简单，只要把摄像机放在模型中心，然后把全景图渲染到模型内表面即可。

<!--more-->

不过还是需要注意以下两点：

一、有`两种`方法可以把全景图渲染到模型内表面：

- 一种是在创建材质时设置 `side` 参数为 `THREE.BackSide`；

<img src="https://newbieyoung.github.io/images/sphere-to-cube-panorama-1.jpg">

- 另一种则是对模型进行镜像变换；

<img src="https://newbieyoung.github.io/images/sphere-to-cube-panorama-2.jpg">

二、球型全景图和立方体全景图使用的图片资源是有区别的；

球型全景图如下：

<img src="https://newbieyoung.github.io/images/sphere-to-cube-panorama-3.jpg">

立方体全景图如下：

<img src="https://newbieyoung.github.io/images/sphere-to-cube-panorama-4.jpg">

虽然球型全景图更贴近人眼的构建模式，但是从模型上来说比立方体更复杂，而且出于兼容性考虑使用 CSS3DRender 时是无法构建球模型的，`因此立方体全景图具有更高的性能和更好的兼容性。`

根据自己搜到的相关知识并加以理解最终用 WebGL 实现了 [球型全景图转立方体全景图工具](https://newbieyoung.github.io/Html_learn/webgl/demo36.html)。

点击上述链接应该会看到该程序根据一张球型全景图生成了一张正方形图片即立方体全景图的一个面；

<img src="https://newbieyoung.github.io/images/sphere-to-cube-panorama-5.jpg">

该工具核心代码如下：

<img src="https://newbieyoung.github.io/images/sphere-to-cube-panorama-6-1.jpg">
<img src="https://newbieyoung.github.io/images/sphere-to-cube-panorama-6-2.jpg">

该工具每次只能得到立方体的一个面，图中 `y轴负平面` 没有被注释得到了执行，因此得到是 `y轴负平面` 也就是立方体的底面，要想得到其它平面，只需要执行相关平面代码即可。

最终得到的六个面后，对应相关命名代入 `立方体全景图` 例子中的图片数组即可得到立方体全景图了。

<img src="https://newbieyoung.github.io/images/sphere-to-cube-panorama-7.jpg">

如果你还有兴趣了解该工具的实现，可以接着往下看。

虽然上边的核心代码很简单每个面大概三四行代码的样子，其实球型全景图到立方体全景图的转换涉及到好几个坐标系的相互映射，稍不小心就会出错，如下：

<img src="https://newbieyoung.github.io/images/sphere-to-cube-panorama-8.jpg">

坐标系说明：

`xyz` 坐标系是全景场景中的坐标系；`st` 坐标系是立方体单个平面的纹理坐标系。

WebGL 中的纹理坐标系统是二维的，为了将纹理坐标和广泛使用的 `xy 坐标系` 区分开来，使用 `s` 和 `t` 命名，称之为 `st 坐标系`。

假设点 `P` 是从球面和 `Z` 轴的交点绕 `Y` 轴旋转 `theta`，然后在 `Y` 轴和其本身组成的平面上绕其过原点的法向量旋转 `phi` 得到，那么点 `P` 的坐标如下：

```text
P(x,y,z);

x = r*cos(phi)*sin(theta);
y = r*sin(phi);
z = r*cos(phi)*cos(theta);
```

因为立方体的六个面都和球面相切，那么假设上图中的 `OP` 直线和正方体的某个面相交于点 `Q`，那么 `OQ` 向量肯定等于 `m` 乘以 `OP` 向量，那么点 `Q` 的坐标如下：

```text
OQ = m*OP;

Q(x1,y1,z1);

x1 = m*x = m*r*cos(phi)*sin(theta);
y1 = m*y = m*r*sin(phi);
z1 = m*z = m*r*cos(phi)*cos(theta);
```

如果点 `Q` 在立方体的 `Z` 平面上；

```text
z1 = r = m*r*cos(phi)*cos(theta);

m*cos(phi)*cos(theta) = 1;
m*sin(phi) = 1/cos(theta)*tan(phi);
m*cos(phi)*sin(theta) = 1/tan(theta);

x1 = r/tan(theta);
y1 = r/cos(theta)*tan(phi);

Q(r/tan(theta),r/cos(theta)*tan(phi),r);
```

此时得到了点 `Q` 在 `xyz` 坐标系中的坐标，再假设点 `Q` 在 `st` 坐标系中的坐标为 `(s0,t0)`，那么就可以求得 `theta` 和 `phi` 的值。

```text
//WebGL纹理坐标最大为1，因此r = 0.5
float r = 0.5;

//xyz坐标系和st坐标系的映射关系
x = 0.5-s;
y = t-0.5;
```

通过上边的分析就不难看懂下述代码了：

<img src="https://newbieyoung.github.io/images/sphere-to-cube-panorama-9.jpg">

之所以求出 `theta` 和 `phi` 的值是因为可以通过这两个值把球型全景图转换为二维，示意图如下：

<img src="https://newbieyoung.github.io/images/sphere-to-cube-panorama-10.jpg">

```text
float s1 = theta/PI*0.5 + 0.5;
float t1 = phi/PI*2.0*0.5 + 0.5;
```

整个推导过程就可以理解为点 `Q(s0,t0)` 为立方体全景图 `z轴正平面` 的点，对应到二维球型全景图中的坐标为 `(s1,t1)`；

最后需要注意的是需要控制 `theta` 和 `phi` 的值不能让其超出范围，还有就是负平面的 `theta` 和 `phi` 可以根据正平面的 `theta` 或者 `phi` 加上一定的值得到。











