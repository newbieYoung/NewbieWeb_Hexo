---
title: CSS3 2D Transform Matrix
date: 2019-04-29 17:17
tags: [CSS3, 图形学, 矩阵变换]
categories: [前端开发]
author: Young
---

给自己出了一道题如下：

在某个大矩形中心有一个黄色的矩形，对该黄色矩形进行一系列 transform 变换得到灰色矩形；

以大矩形中心为`坐标原点`，屏幕水平向左为`X 轴正方向`，屏幕垂直向上为`Y 轴正方向`，黄色矩形初始位置中心在坐标原点，根据其宽高可以得到其初始位置四个顶点的坐标`initPoints`；

那么求黄色矩形经过一系列变换后新的顶点坐标。

<img src="https://newbieyoung.github.io/images/css-2d-transform-matrix-0.jpg">

示例链接如下：[https://newbieyoung.github.io/CSS_learn/transform3.html](https://newbieyoung.github.io/CSS_learn/transform3.html)

<!--more-->

## 解答

<img src="https://newbieyoung.github.io/images/css-2d-transform-matrix-1.jpg">

其实不管 CSS3 transform 属性有多复杂，都是可以通过 `getComputedStyle` 直接获得最终变换矩阵的，具体方法实现如下：

```javascript
//获得transform属性对应的矩阵形式
function getTransformMatrix(transform){
    var $div = document.createElement('div');
    $div.style.visibility = 'hidden';
    $div.style.position = 'fixed';

    //处理transform属性的兼容性
    var transformProperty = 'transform';
    if('transform' in $div.style){
        transformProperty='transform'
    } else if( 'WebkitTransform' in $div.style ){
        transformProperty='webkitTransform'
    } else if('MozTransform' in $div.style){
        transformProperty='MozTransform'
    } else if('OTransform' in $div.style){
        transformProperty='OTransform'
    }

    $div.style[transformProperty] = transform;
    document.body.appendChild($div);

    var style = window.getComputedStyle($div);
    var matrix = style[transformProperty];

    document.body.removeChild($div);

    return matrix;
}
```

通过 `getTransformMatrix` 函数可以获得以下形式的变换矩阵：

```
matrix(a, b, c, d, e, f);

matrix(0.430963, 1.01542, -0.234879, 1.76697, 5, 40)
```

转换为三阶矩阵：

```
[a, c, e,
 b, d, f,
 0, 0, 1];
```

不知道大家想过没有，明明是 `2D` 变换，转换为`三阶`矩阵干啥？

其实这里是引入了一个`齐次坐标`的概念，用 `N+1` 维的向量来表示 `N` 维向量；比如在 2D 坐标系中某个点 `(x, y)` 可以在逻辑上表示为 `(x*w, y*w, w)`，`w` 即为新增的那个量。

引入齐次坐标的好处在于可以把`缩放`、`旋转`、`平移`等变换都统一转换成矩阵乘法的形式，这样不管进行多少次变换，都可以表示成矩阵连乘的形式了。

如果某个 2D 坐标系中点为 `(x, y)`，转换为齐次坐标 `(x, y, 1)`，那么经过上述变换后的新坐标为`(nx, ny, nz)`：

```
nx = a * x + c * y + 1 * e;
ny = b * x + d * y + 1 * f;
nz = 0 + 0 + 1;
```

再把齐次坐标还原，最终坐标为 `(nx, ny)`。

具体方法实现如下：

```javascript
//计算矩阵变换后的坐标点
function getMatrixPoints(p,matrix){
    var mat = matrixAnalyze(matrix);

    // var mat3 = [mat[0],mat[2],mat[4],
    //             mat[1],mat[3],mat[5],
    //             0,0,1];

    // var mat3 = [a,c,e,
    //             b,d,f,
    //             0,0,1];

    // var newX = a * x + c * y + 1 * e;
    // var newY = b * x + d * y + 1 * f;
    // var newZ = 0 + 0 +1;

    //计算变换后点坐标
    var newX = mat[0] * p.x + mat[2] * p.y + 1 * mat[4];
    var newY = mat[1] * p.x + mat[3] * p.y + 1 * mat[5];
    var newZ = 0 + 0 +1;

    return {x:newX/newZ,y:newY/newZ};
}
```

已知初始坐标 `initPoints` 和变换矩阵 `matrix` 相乘即可得到 `matrixPoints`，到这里我以为这个题目已经解完了。

直到我尝试着在大矩形中用`红色`直线把计算得到的 matrixPoints 连起来才发现，新坐标连接得到的图形和 CSS3 transform 变换得到灰色图形并不重合。

这也就意味着`上述计算过程有问题`！

在走了不少弯路之后我才意识到一个问题：`CSS3 2D transform 坐标系和题目中设定的坐标系的 Y 轴正方向是相反的`（题目中设置的坐标系 Y 轴正方向是屏幕垂直向上而 CSS3 2D transform 坐标系的 Y 轴正方向是屏幕垂直向下）。

因此在计算新坐标之前，需要对 getComputedStyle 得到的变换矩阵进行`镜像变换`，从而转换为题目中设定坐标系的变换矩阵，也就是示例中 `fixedMatrix`，最后计算的新坐标为 `fixedPoints`；用`蓝色`直线绘制出来就可以看到和灰色图形完全重合了。

## 总结

在进行图形学计算时，不管是 2D 还是 3D 都要格外注意不同坐标系之间的区别！





















