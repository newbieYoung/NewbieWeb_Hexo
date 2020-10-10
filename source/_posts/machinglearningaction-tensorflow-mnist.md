---
title: UI工程师的机器学习之旅（四）Tensorflow MNIST 数字识别
date: 2019-06-30 16:19
tags: [机器学习,Tensorflow,MNIST]
categories: [机器学习]
author: Young
---

Tensorflow 是谷歌开源的一个计算框架，该计算框架可以很好地实现各种深度学习算法，网络上相关学习资料挺多，就不过多复制粘贴了；

这里只是记录一下自己的学习小结。

<!--more-->

## 基础程序结构

首先来看 Tensorflow 的基础程序结构；

以实现`两个数相加`为例：

使用 Python 实现，代码如下：

```
v_1 = 2.0
v_2 = 3.0

def add_n(a,b):
    return a+b

print(add_n(v_1, v_2))
```

使用 Tensorflow 实现，代码如下：

```
import tensorflow as tf

v_1 = tf.constant([1,2,3,4], name='input1')
v_2 = tf.constant([2,3,4,5], name='input2')

v_add = tf.add_n([v_1, v_2], name='add')

with tf.Session() as sess:
	print(sess.run(v_add))
```

Tensorflow 的基础程序结构 和 普通程序结构类似，都分为`定义`和`执行`两部分；

区别在于 Tensorflow 程序中需要使用`会话（Session）`来`执行`定义好的计算，计算完成之后需要关闭会话来帮助系统回收资源，否则可能出现资源泄漏的问题；

`两个数相加`例子中，先定义好两个常量`v_1`和`v_2`以及 两个常量的相加计算`v_add`，然后在会话中使用`sess.run(v_add)`执行；

由于把所有计算都放在了`with`内部，通过 Python 上下文管理器机制，计算完成之后会自动释放资源，也就不需要在显示执行`Session.close`函数来关闭会话了。

## 简单线性回归

在[UI工程师的机器学习之旅（一）线性回归和梯度下降](https://newbieweb.lione.me/2017/11/23/machinelearningaction-js-linear-regression/)中曾经用 JS 简单地实现过线性回归，初步了解 Tensorflow 后也可以尝试着用 Tensorflow 来实现；

先`假定线性模型（ y = 0.1 * x + 0.3 ）`，然后`使用随机生成的数据模拟训练数据`：

```
x_data = np.random.rand(100).astype(np.float32)
y_data = x_data * 0.1 + 0.3
```

拟合之前先`初始化待拟合线性模型的参数`：

```
Weights = tf.Variable(tf.random_uniform([1],-1.0,1.0))
biases = tf.Variable(tf.zeros([1]))
```

Tensorflow 中使用`tf.Variable`来定义变量，可以把它当成普通程序的变量来理解，只不过在使用时需要先使用`tf.global_variables_initializer`函数定义`初始化变量操作`，然后在会话中执行。

接着根据初始化的模型参数计算预测值：

```
y = Weights * x_data + biases
```

线性回归的误差为`真实值和预测值差的平方和`：

```
loss = tf.reduce_mean(tf.square(y-y_data))
```

最后根据误差对模型参数进行梯度优化，在 Tensorflow 框架中提供了多种优化算法实现，这里使用`tf.train.GradientDescentOptimizer(0.5).minimize(loss)`，传入`学习速率`和`误差计算`即可。

从会话中输出的线性模型参数可以看到，随着训练次数的增加两个线性模型在不断拟合。

<img src="https://newbieyoung.github.io/images/machinelearningaction-tensorflow-mnist-0.jpg">

完整代码如下：

```
import tensorflow as tf
import numpy as np

# 训练数据
x_data = np.random.rand(100).astype(np.float32)
y_data = x_data * 0.1 + 0.3

# 初始化线性模型参数
Weights = tf.Variable(tf.random_uniform([1],-1.0,1.0))
biases = tf.Variable(tf.zeros([1]))

# 线性模型计算结果
y = Weights * x_data + biases

# 误差
loss = tf.reduce_mean(tf.square(y-y_data))

# 梯度下降
train = tf.train.GradientDescentOptimizer(0.5).minimize(loss)

# 初始化变量
init = tf.global_variables_initializer()

with tf.Session() as sess:
	sess.run(init)
	for step in range(201):
		sess.run(train)
		if step % 20 == 0:
			print(step,sess.run(Weights),sess.run(biases))
```

## 单层神经网络

MNIST 数字识别应该算是 Tensorflow 的 Hello World 程序了；

首先加载数据，Tensorflow 已经帮我们封装好了一个类来处理，这个类会自动下载并转化 MNIST 数据的格式，将数据从原始的数据包中解析成训练数据和测试神经网络时使用的格式：

```
import tensorflow as tf
from tensorflow.examples.tutorials.mnist import input_data

# 载入 MNIST 数据集， 如果指定地址下没有已经下载好的数据，那么 Tensorflow 会自动下载
mnist = input_data.read_data_sets('/Users/young/Documents/MNIST_data/', one_hot=True)

# 输出 MNIST 数据集信息
#print('training data size : ', mnist.train.num_examples)
#print('testing data size : ', mnist.test.num_examples)
#print('example traning data :', mnist.train.images[0]) # 数字图片像素数据
#print('example traning data :', mnist.train.labels[0]) # 数字图片类别数据
print('--- mnist data ready! ---')
```

`mnist.train`为训练数据集，`mnist.test`为测试数据集，`mnist.train.images`为训练数据集的数字图片像素数据，`mnist.train.labels`为训练数据集的数字图片类别数据；

数字图片数据如下图所示：

<img src="https://newbieyoung.github.io/images/machinelearningaction-tensorflow-mnist-1.png">

然后定义 MNIST 数据集常量；

```
# MNIST 数据集相关的常数
INPUT_NODE = 784 # 输入层节点数（28 * 28 共 784 个像素）
OUTPUT_NODE = 10 # 输出层节点数（类别数目，因为要区分 0-9 这10个数字，因此这里的输出层节点数为10）

BATCH_SIZE = 100 # 单次训练数据量（小批量）
TRAINING_STEPS = 1000 # 训练轮数
LEARNING_RATE_BASE = 0.01 # 基础学习速率
```

数字图片尺寸为 28*28 把所有像素数据都输入到神经网络，那么输入层节点数为 784，而要区分 0-9 这 10 个数字，因此输出层节点数为 10；

每一轮迭代在全部训练数据上计算损失函数会非常消耗时间（批量梯度下降），而如果每一轮迭代只在随机一条数据上计算损失函数时间消耗大大减少，但拟合效果会不好（随机梯度下降）；因此一般会采用折中的办法，每一轮迭代在一小部分数据上计算损失函数，这样既不会消耗太多时间，拟合效果也不会太差（小批量梯度下降）；

例子中定义这一小部分数据的数据量为`BATCH_SIZE = 100`，训练轮数为`TRAINING_STEPS = 1000`，基础学习速速率为`LEARNING_RATE_BASE = 0.01`。

接着就可以开始定义训练模型了：

```
# 单层神经网络模型
def train_model():
    # 输入
    x_i = tf.placeholder(tf.float32, shape=(None,INPUT_NODE), name='x-input')
    y_i = tf.placeholder(tf.float32, shape=(None,OUTPUT_NODE), name='y-input')

    # 权重值 和 偏置量
    W = tf.Variable(tf.truncated_normal([INPUT_NODE, OUTPUT_NODE], stddev=0.1))  # truncated_normal 正态分布产生函数
    b = tf.Variable(tf.constant(0.1, shape=[OUTPUT_NODE]))

    # 输出
    y = tf.nn.softmax(tf.matmul(x_i,W) + b) # softmax 将神经网络向前传播得到的结果转换为概率分布

    # 损失函数
    cross_entropy = -tf.reduce_sum(y_i * tf.log(y)) # 交叉熵

    # 优化方法
    train_step = tf.train.GradientDescentOptimizer(LEARNING_RATE_BASE).minimize(cross_entropy)

    # 模型评估
    correct_prediction = tf.equal(tf.argmax(y,1), tf.argmax(y_i,1))
    accuracy = tf.reduce_mean(tf.cast(correct_prediction, 'float'))

    # 变量初始化
    init_op = tf.global_variables_initializer()

    with tf.Session() as sess :
        sess.run(init_op)

        # 设定训练的轮数
        for i in range(TRAINING_STEPS) :

            # 每次选取 batch_size 个样本进行训练
            start = (i * BATCH_SIZE) % mnist.train.num_examples
            end = min(start + BATCH_SIZE, mnist.train.num_examples)

            # 通过选取的样本训练神经网络并更新参数
            sess.run(train_step, feed_dict={x_i: mnist.train.images[start:end], y_i: mnist.train.labels[start:end]})

            # 每隔一段时间计算在所有训练数据上的交叉熵并输出
            # if i % 200 == 0 :
            #    total_cross_entropy = sess.run(cross_entropy, feed_dict={x_i:mnist.train.images,y_i:mnist.train.labels})
            #    print('After %d training steps, cross entropy on all data is %g' % (i, total_cross_entropy))

        # 正确率
        print(sess.run(accuracy, feed_dict={x_i: mnist.test.images, y_i: mnist.test.labels}))
```

流程大致和线性回归差不多，除了以下几点：

- 占位符机制实现小批量梯度下降

```
# 输入
x_i = tf.placeholder(tf.float32, shape=(None,INPUT_NODE), name='x-input')
y_i = tf.placeholder(tf.float32, shape=(None,OUTPUT_NODE), name='y-input')
```

在线性回归中使用的批量梯度下降，每一轮迭代都是使用全部训练数据，但是在上述例子中使用的是小批量梯度下降，每一轮迭代只使用一小部分训练数据；

因此在定义输入`x_i`和`y_i`时使用了占位符`tf.placeholder`，`shape=(None,INPUT_NODE)`则定义该`占位符`的数据格式为二维向量；`None`表示该二维向量的第一维可以是任意长度。

```
# 每次选取 batch_size 个样本进行训练
start = (i * BATCH_SIZE) % mnist.train.num_examples
end = min(start + BATCH_SIZE, mnist.train.num_examples)

# 通过选取的样本训练神经网络并更新参数
sess.run(train_step, feed_dict={x_i: mnist.train.images[start:end], y_i: mnist.train.labels[start:end]})
```

然后在每一轮训练时依次取一部分训练数据并填入到占位符中进行训练。

- 交叉熵计算误差

```
# 损失函数
cross_entropy = -tf.reduce_sum(y_i * tf.log(y)) # 交叉熵
```

线性回归的损失函数为`真实值和预测值差的平方和`，但是在分类问题中一般使用`交叉熵`；想要快速了解其含义可以查看知乎上的一篇回答[如何通俗的解释交叉熵与相对熵?](https://www.zhihu.com/question/41252833/answer/195901726)；

- Softmax 回归把前向传播结果转换为概率分布

```
# 输出
y = tf.nn.softmax(tf.matmul(x_i,W) + b) # softmax 将神经网络向前传播得到的结果转换为概率分布
```

`交叉熵`刻画的是两个概率分布之间的距离，然而神经网络的输出却不一定是一个概率分布，因此在用交叉熵计算误差之前，还需要对神经网络的输出进行 Softmax 回归转换，从而把神经网络前向传播得到的结果变成概率分布。

- 使用测试数据评估模型

```
# 模型评估
correct_prediction = tf.equal(tf.argmax(y,1), tf.argmax(y_i,1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction, 'float'))
```

`tf.argmax`是返回向量中最大值的索引号，神经网络的输出为长度为 10 的向量，如果最大值序号为 1 则表示预测为数字 1 的概率最大；

`tf.cast`是会把结果比较向量转换为浮点数向量，因为预测正确则为 1，预测错误则为 0，通过计算平均值即可得到整体的正确率了。


完整代码在[https://github.com/newbieYoung/tensorflow_learn/blob/master/demo8.py](https://github.com/newbieYoung/tensorflow_learn/blob/master/demo8.py)。

## 小结

<img src="https://newbieyoung.github.io/images/machinelearningaction-tensorflow-mnist-3.jpg">

上述单层神经网络模型正确率大致在`0.91`左右，不是很好，需要采用一些手段进行优化，且慢慢折腾着...





