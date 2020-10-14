---
title: 3D图形学线代基础
date: 2020-10-14 18:00
tags: [线性代数]
categories: [线性代数]
author: Young
---

如标题所言都是些很基础但是异常重要的数学知识，如果不能彻底掌握它们，在 3D 的世界中你将寸步难行。

## 一、向量

`向量`是一个基础且重要的数学工具，`从几何意义上来说主要用来描述事物间的位移以及指示方向`，如下：

![图 1.2.3-4.png](./images/chapter1/1.2.3-4.png)

上图 XY 坐标系中 A 点坐标为 （x1，y1），B 点坐标系为 （x2，y2），从 B 点移动到 A 点时，位置的变化即为`向量 BA`。

简单来说`向量是既有大小又有方向的线段`，从 B 点移动到 A 点的方向也就是图中箭头所示的方向即为向量 BA 的方向，线段 BA 的长度即为向量 BA 的大小。

从数学表现形式上来看向量就是一个数字列表，列表中的每个数表示在不同维度上的有向位移，还是以向量 BA 为例：

![图 1.2.3-5.png](./images/chapter1/1.2.3-5.png)

从 B 点移动到 A 点，在 X 轴上位置的变化为 `x1-x2`，在 Y 轴上位置的变化为 `y1-y2`，把这两个维度上的位置的变化组合在一起最终形成了二维向量 BA；图中 BA 上方的箭头表示向量的方向是从 B 点移动到 A，有些时候不标记出来则默认前面是起始点后边是终止点。

![图 1.2.3-6.png](./images/chapter1/1.2.3-6.png)

向量数字列表有两种组织方式，水平组织的为`行向量`，垂直组织的为`列向量`；行向量和列向量的区别体现在和矩阵的乘法中，因为涉及到矩阵，这里就不过多展开了；向量运算和矩阵相关知识将会在后续内容中进行详细讲解。

回到示例图，当从 O 点移动到 B 点时，可以用向量 OB 来表示；因为 O 点为坐标系原点，其坐标值为 （0，0），分别计算向量 OB 在 X 轴和 Y 轴的有向位移，写成行向量的形式如下：

![图 1.2.3-7.png](./images/chapter1/1.2.3-7.png)

向量 OB 恰好和 B 点坐标是一致的，因此可以理解向量和点在概念上完全不同，但是在数学形式上却是等价的；这也就是为什么在 ThreeJS 框架中 `Vector3` 既可以用来表示三维向量又可以用来表示三维坐标系中的点，其实际含义和具体的使用场景有关。