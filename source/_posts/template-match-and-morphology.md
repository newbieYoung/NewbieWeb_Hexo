---
title: OpenCV 图像模板匹配、形态学变换以及卷积运算
date: 2019-09-02 20:59
tags: [OpenCV, 模板匹配, 形态学变换]
categories: [数字图像处理]
author: Young
---

简单记录下几个知识点：`卷积运算`、`模版匹配`、`形态学变换`。

<!--more-->

## 图像卷积运算和形态学变换

随便摘录一段维基百科对`数学形态学`的定义：

`数学形态学（Mathematical morphology）是一门建立在格论和拓扑学基础之上的图像分析学科，是数学形态学图像处理的基本理论。其基本的运算包括：腐蚀和膨胀、开运算和闭运算、骨架抽取、极限腐蚀、击中击不中变换、形态学梯度、Top-hat变换、颗粒分析、流域变换等。`

怎么说呢，反正我看完之后感觉比不看还要差，就好像每个字都在劝我放弃...

一直以来自学养成的习惯是先去找例子，因为形态学变换是基于图像卷积运算的，所以先看卷积运算。

`在我理解图像卷积运算和数字加减乘除运算是同一种东西，区别在于操作对象不一样而且操作流程复杂一点。`

如图：

<img src="https://newbieyoung.github.io/images/template-match-and-morphology-0.png">

卷积运算操作的是两个矩阵，分别为卷积核和待处理矩阵；

卷积运算的操作流程是先把卷积核`旋转 180 度`，然后从待处理矩阵的左上角第一位开始运算，每次运算时需要把旋转之后的`卷积核中心和当前运算位对齐`，然后把`卷积核覆盖的待处理矩阵元素一一对应相乘，待处理矩阵中不存在的元素一般使用常量 0 代替`，最终的`结果之和`即为当前位的计算结果。

也就是说图中矩阵第一位的卷积运算结果为`9*0+8*0+7*0+6*0+5*1+4*2+3*0+2*5+1*4 = 27`，然后以此类推即可。

具体实现如下：

```
/**
 * 卷积
 * data     卷积核+矩阵数据
 * kerRows  卷积核行数
 * kerCols  卷积核列数
 * matRows  矩阵行数
 * matCols  矩阵列数
 * ----
 * 对于图片来说行数即高度、列数即宽度
 */
function convolution(data, kerRows, kerCols, matRows, matCols) {
  let len = 4;
  //拆分矩阵数据
  let kerLen = kerRows * kerCols * len;
  let kerMat = data.slice(0, kerLen);

  let matLen = matRows * matCols * len;
  let matMat = data.slice(kerLen, matLen + kerLen);

  //暂不考虑非奇数尺寸卷积核
  if (kerRows % 2 != 1 || kerCols % 2 != 1) {
    return [];
  }

  //一般而言卷积核所有元素之和等于1；否则大于1时生成的图片亮度会增加，小于1时生成的图片亮度会降低
  let eles = [];
  let sums = [0.0, 0.0, 0.0, 0.0];
  for (let z = 0; z < len; z++) {
    for (let i = 0; i < kerRows; i++) {
      for (let j = 0; j < kerCols; j++) {
        let no = (i * kerCols + j) * len;
        sums[z] += kerMat[no + z];
        eles[no + z] = kerMat[no + z];
      }
    }
  }
  for (let z = 0; z < len; z++) {
    for (let i = 0; i < kerRows; i++) {
      for (let j = 0; j < kerCols; j++) {
        let no = (i * kerCols + j) * len;
        eles[no + z] = eles[no + z] / sums[z];
      }
    }
  }

  //卷积核中心
  let cRow = (kerRows - 1) / 2;
  let cCol = (kerCols - 1) / 2;

  //卷积计算
  let result = new Array(matLen);
  for (let r = 0; r < matRows; r++) {
    let rs = r - cRow;
    for (let c = 0; c < matCols; c++) {
      let cs = c - cCol;
      let noCur = (r * matCols + c) * len;
      for (let z = 0; z < len; z++) {
        let sum = 0;
        if (z == len - 1) {
          //透明通道不参与计算
          sum = matMat[noCur + z];
        } else {
          for (let i = 0; i < kerRows; i++) {
            let r1 = rs + i;
            for (let j = 0; j < kerCols; j++) {
              let c1 = cs + j;

              let n1 = (r1 * matCols + c1) * len;
              let v1 = 0; //超出范围使用常量0代替
              if (r1 >= 0 && r1 < matRows && c1 >= 0 && c1 < matCols) {
                v1 = matMat[n1 + z];
              }

              let r2 = kerRows - 1 - i; //卷积核旋转180度
              let c2 = kerCols - 1 - j;

              let n2 = (r2 * kerCols + c2) * len;
              let v2 = eles[n2 + z];

              sum += v1 * v2;
            }
          }
        }
        result[noCur + z] = sum;
      }
    }
  }

  return result;
}
```

需要注意的地方在于当在图像上进行卷积运算时如果通道值大于通道的最大值或者小于通道的最小值（一般为 255 和 0）时会被截断处理导致计算结果异常，解决办法是对卷积核进行处理让其所有元素之和等于 1，这样最终的结果就不会超出范围了。

根据卷积核的不同，图像的卷积运算也会起到不同的作用，比较常用的是锐化和边缘提取，以及接下来要说的两种基础的形态学变换，`膨胀和腐蚀`。

`膨胀`顾名思义就是会让图像中亮的区域扩大范围也就是膨胀了，比如：

<img src="https://newbieyoung.github.io/images/template-match-and-morphology-1.jpg">

将结构 B 在结构 A 上进行膨胀操作，结构 B 的中心会在结构 A 的红色元素进行移动；在移动过程中如果结构 A 上其它被覆盖的元素没有结构 B 对应的元素亮则会被结构 B 上的元素替换，最后会形成第三幅图的样子。

膨胀操作的实质其实是卷积运算之后取局部最大值，上图中结构 B 是卷积核，结构 A 则是待处理矩阵。

具体实现如下：

```
/**
 * 膨胀
 */
function dilate(data, kerRows, kerCols, matRows, matCols ) {
  let len = 4;
  let kerLen = kerRows * kerCols * len;
  let matLen = matRows * matCols * len;
  let matMat = data.slice(kerLen, matLen + kerLen);

  let conv = convolution(data, kerRows, kerCols, matRows, matCols); //卷积

  for (let i = 0; i < conv.length; i++) {
    if (conv[i] < matMat[i]) {
      //取局部最大值
      conv[i] = matMat[i];
    }
  }

  return conv;
}
```

图像卷积运算之后会把卷积结果图像和原图进行比较，如果卷积结果图像通道值小于原图通道值则用原图通道值代替，使用自己实现的卷积运算方法和膨胀变换方法实际效果如下：

<img src="https://newbieyoung.github.io/images/template-match-and-morphology-2.jpg">

`腐蚀`和膨胀相反会让图像中亮的区域缩小范围，比如：

<img src="https://newbieyoung.github.io/images/template-match-and-morphology-3.jpg">

将结构 B 在结构 A 上进行腐蚀操作，结构 B 的中心会在结构 A 的红色元素进行移动；在移动过程中如果结构 A 上红色元素被结构 B 的绿色元素覆盖的越多，则其最终颜色越接近绿色否则越接近白色，最终会形成第三幅图的样子。

腐蚀操作的实质其实是卷积运算之后取局部最小值，和膨胀的示例一样结构 B 是卷积核，结构 A 则是待处理矩阵。

具体实现如下：

```
function erode(data, kerRows, kerCols, matRows, matCols) {
  let len = 4;
  let kerLen = kerRows * kerCols * len;
  let matLen = matRows * matCols * len;
  let matMat = data.slice(kerLen, matLen + kerLen);

  let conv = convolution(data, kerRows, kerCols, matRows, matCols); //卷积

  for (let i = 0; i < conv.length; i++) {
    if (conv[i] > matMat[i]) {
      //取局部最小值
      conv[i] = matMat[i];
    }
  }

  return conv;
}
```

图像卷积运算之后会把卷积结果图像和原图进行比较，如果卷积结果图像通道值大于原图通道值则用原图通道值代替，使用自己实现的卷积运算方法和腐蚀变换方法实际效果如下：

<img src="https://newbieyoung.github.io/images/template-match-and-morphology-4.jpg">

上述 卷积、膨胀、腐蚀 三种操作的简单实现例子在 [https://newbieyoung.github.io/OpenCV_learn/demo/demo9.html](https://newbieyoung.github.io/OpenCV_learn/demo/demo9.html)，从效果上看貌似没啥问题。

## OpenCV 图像模板匹配

<img src="https://newbieyoung.github.io/images/template-match-and-morphology-5.jpg">

模板匹配如图所示会从大图中匹配到小图的准确位置，看起来好像是个很高大上的功能；其实原理很简单，简单来说就是拿着小图从大图左上角开始移动，每移动一个像素就进行一次相似度计算；把所有计算结果整合起来就是右下角那张图片，结果图片中最亮的地方就是相似度最高的地方；

另外定位的时候还需要注意结果图片的宽高分别为大图的宽高减去小图的宽高。

关于图像相似度的计算方法有很多，比如在[UI 工程师的机器学习之旅（三）K 近邻识别手写数字](https://newbieweb.lione.me/2018/08/16/machinelearningaction-js-knn/)用到的`感知哈希`算法；

此外 OpenCV 中则有六种：

<img src="https://newbieyoung.github.io/images/template-match-and-morphology-6.jpg">

第一种`TM_SQDIFF`很好理解，`T`表示模板像素，`I`表示目标像素，简单来说就是像素差的平方和，相似度越高计算结果越小；

第二种`TM_SQDIFF_NORMED`和第一种类似，区别在于进行了标准化操作。

其它算法我觉得有点像是在计算两张图片像素值的协方差，如果两张图片某个位置的变化趋势一致，也就是说如果其中一个大于自身的平均值，另外一个也大于自身的平均值，那么它们之间相乘的结果是正值；反之如果变化趋势相反，即其中一个大于自身的平均值，另外一个却小于自身的平均值，那么它们之间相乘的结果就是是负值；也就是说最终计算结果越大则相似度越高。

之所以会这么说是因为图中第五种`TM_CCOFF`和协方差计算公式简直一模一样。

<img src="https://newbieyoung.github.io/images/template-match-and-morphology-7.jpg">
