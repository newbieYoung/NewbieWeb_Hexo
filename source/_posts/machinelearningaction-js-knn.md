---
title: UI工程师的机器学习之旅（三）K近邻识别手写数字
date: 2018-08-16 21:58
tags: [机器学习,KNN]
categories: [机器学习]
author: Young
---

K 近邻算法基本原理：

存在一个样本数据集合，并且样本集中每个数据都存在标签，即我们知道样本集中每一数据项与所属分类的对应关系；

输入没有标签的新数据后，将新数据的每个特征与样本集中数据对应的特征进行比较，然后算法提取样本集中特征最相似数据的分类标签；

一般来说我们只选择样本数据集中前 K 个最相似的数据，这就是 K 近邻算法中 K 的出处，通常 K 是不大于 20 的整数；

最后前 K 个最相似数据中出现次数最多的分类，即为新数据的分类。

<!--more-->

举例来说：

有人曾统计过很多电影的打斗和接吻镜头，然后以此为分类依据，数据如下：

|    电影名称    | 打斗镜头 | 接吻镜头 | 电影类型 |
| ------------- | ------- | ------- | ------ |
|    California Man    | 3 | 104 | 爱情片 |
|    He is Not Really into Dudes    | 2 | 100 | 爱情片 |
|    Beautiful Woman    | 1 | 81 | 爱情片 |
|    Kevin Longblade    | 101 | 10 | 动作片 |
|    Robo Slayer 300    | 99 | 5 | 动作片 |
|    Amped II    | 98 | 2 | 动作片 |
|    ?    | 18 | 90 | ? |

现在有部未知分类的电影，统计出其打斗镜头为 18，接吻镜头为 90，根据 K 近邻算法，目前已有 5 个样本数据，需要计算新数据和样本集中所有数据的距离；

我们以打斗镜头数为横坐标、接吻镜头数为纵坐标构建一个二维坐标系，然后把所有数据放入其中即可计算距离了，如下：

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-knn-1.jpg">

|    电影名称    | 与未知电影的距离 |
| ------------- | ------- |
|    California Man    | 20.5 |
|    He is Not Really into Dudes    | 18.7 |
|    Beautiful Woman    | 19.2 |
|    Kevin Longblade    | 115.3 |
|    Robo Slayer 300    | 117.4 |
|    Amped II    | 118.9 |

根据 K 近邻算法，如果 K=4，则前四个最靠近的电影依次为 He is Not Really into Dudes、Beautiful Woman、California Man、Kevin Longblade；这四部电影中有 3 部爱情片、1 部动作片；因此新电影的类型为爱情片。

另外再说一下在电影分类的例子里边，可不能简单的认为接吻镜头比打斗镜头多的为爱情片，打斗镜头比接吻镜头多的为动作片哈，要不然如果未知电影的接吻镜头和打斗镜头相同的话，你就只能把这部电影归类为爱情动作片了...

到此《机器学习实战》中的内容复制完毕，接来下聊聊自己的实践所得。

K 近邻算法很简单也很容易理解，我们用它来进行手写数字的识别，完整例子如下：
[https://newbieyoung.github.io/machinelearninginaction-js/Ch02/demo0.html](https://newbieyoung.github.io/machinelearninginaction-js/Ch02/demo0.html)

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-knn-0.jpg">

在白色画布中用鼠标写一个数字，然后点击识别，即可在 `console` 中看到识别结果；

需要注意的是识别之前，需要先加载并解析样本集；

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-knn-2.jpg">

样本集中包含 0-9 的数字图片，尺寸为 `32x32`，每个数字均有 5 个不同样本，`console` 中的 `404` 是由于加载机制导致的，因为加载方法考虑了样本数量不确定的情况，因此加载时如果报错则说明当前数字样本已经加载完毕，开始加载下一个数字；

等到输出 `load end` 和 `parse end` 时则说明例子可以正常运行了。

手绘功能使用的是 [signature_pad](https://github.com/szimek/signature_pad)，这个组件的原理应该很简单，监听鼠标事件，获得移动点，然后连线即可；难点在于移动速度控制笔画粗细的函数不好确定，容易出现不同速度之间的笔画不平滑导致笔迹凹凸不平。

另外样本图片尺寸之所以为 `32x32`，则涉及到图片相似度算法。

计算图片相似度的算法有很多，出于简单易实现以及精度方面的考虑，我采用了`感知哈希`中的 `pHash` 算法；

主要过程以及实现如下：

1、缩小尺寸，一般缩小为 32x32，这也是样本图片尺寸为 32x32 的主要原因，因为可以省去算法的第一步；

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-knn-3.jpg">

2、简化色彩，将图片转化为灰度图片，进一步简化计算量；

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-knn-4.jpg">

3、对灰度图片的像素值进行`离散余弦变换`

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-knn-5.jpg">

这时就有疑惑了，`离散余弦变换`是什么鬼？

查查维基百科：

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-knn-7.jpg">

刚看第一句，`傅里叶变换`又是什么鬼？

一般遇到这种尴尬的情况，会选择放弃治疗，不管了直接看公式：

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-knn-8.jpg">

`最常用`！OJBK！搞定！

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-knn-6.jpg">

当然，如果对某个概念并不是很理解的话，至少得知道这个东西有什么用吧！

请看`测试离散余弦变换`例子：
[https://newbieyoung.github.io/machinelearninginaction-js/test/test0.html](https://newbieyoung.github.io/machinelearninginaction-js/test/test0.html)

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-knn-9.jpg">

分别是原图、灰度图以及灰度之后进行`离散余弦变换`然后再把变换后的值当成像素值的新图；

离散变换值转换为像素值我采用的方法是取的最小值和最大值，然后获得差值，最后把所有的值减去最小值除以差值获得对应比例，最后把该比例换算到 0-255 的区间即可。

粗略看新图只会发现原图的内容不存在了，仔细看则会发现新图左上角有一块很小的图案，放大后会稍微明显一点；

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-knn-10.jpg">

因此可以简单的理解`离散余弦变换`会把图片内容转换为左上角这种类似指纹一样的图案，这也就不难理解第四步了。

4、缩小 DCT，虽然 DCT 的结果是 32x32 矩阵，但是我们只保留左上角 8x8 的部分，这部分呈现了图片中的最低频率；

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-knn-11.jpg">

5、计算缩小 DCT 的平均值；

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-knn-12.jpg">

6、计算 hash 值，在缩小的 DCT 矩阵中，如果某一位的值大于或等于平均值则设置为 1，小于则设置为 0，按照某一顺序得到一个由 0、1 组成的 64 位的字符串。

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-knn-13.jpg">

至此就是 `pHash` 算法的全部了。

最后我们把不同图片之间 `pHash` 算法得到的 64 位`指纹`数据的不同位数的个数当成`距离`即可（这个距离也被称为`汉明距离`）。

另外 K 近邻算法还有几点需要注意：

- 上述 K 近邻算法实现采用的是`暴力计算`的方式，这种方式在样本集很小的时候效果很好；但当样本集增大之后就会很低效，为了解决这个问题，诞生了大量基于树结构的改进算法，比如：`K-D 树`、`球树`等。

- 错误率；

虽然我并没有做这部分工作，但是实际应用时需要对错误率进行测试。

放到这个例子中，你在运行时会有较大概率发现识别错误，这和样本数量太少有一定关系；

识别错误时，你可以在 `console` 中看到：

<img src="https://newbieyoung.github.io/images/machinelearningaction-js-knn-14.jpg">

距离最小的也超过了 `10`，在`pHash`算法中如果不同数据位不超过 `5` 则说明两张图片类似；如果超过 `10`，就说明差别很大！

同时也说明 `pHash` 算法得到哈希值只是内容特征值，并没有反应结构特征，毕竟我们人识别数字并不是根据图片中某些位置的像素值是多少来判断的。

- 例子中实现的离散余弦变换是完全按公式实现的，计算量较大；其实是有优化版本的；

- 样本数据特征如果单位不一致，还需要进行归一化处理，防止计算距离时某些值比重过高；

- K 近邻算法是分类数据最简单最有效的算法，但也存在着很多缺点，比如计算复杂度高、空间复杂度高、无法给出数据的内在含义等。










