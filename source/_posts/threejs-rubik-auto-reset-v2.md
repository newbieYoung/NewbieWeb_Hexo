---
title: 使用WebAssembly实现前端运行Kociemba算法自动解魔方
date: 2018-08-31 16:49
tags: [WebAssembly,魔方]
categories: [其它]
author: Young
---

`Kociemba算法`，又称为`Two-Phase算法`或者`二阶段算法`；本质上是利用搜索算法来还原魔方；

当然这里并不准备仔细聊这个算法的实现，毕竟我也不是很清楚！

接下来的内容是怎么使用`WebAssembly技术`让`C语言`实现的`Kociemba算法`在浏览器环境中也能运行起来。

<!--more-->

如果不是很清楚什么是`WebAssembly`，可以去看这篇文章[https://www.zhihu.com/question/31415286/answer/58022648](https://www.zhihu.com/question/31415286/answer/58022648)；

另外还需要一个工具`Emscripten`，如果对这个也不是很清楚可以去看这篇文章[http://www.ruanyifeng.com/blog/2017/09/asmjs_emscripten.html](http://www.ruanyifeng.com/blog/2017/09/asmjs_emscripten.html)；

接下需要一个`Kociemba算法`的实现，在`GITHUB`上边有很多，比如这个[https://github.com/muodov/kociemba](https://github.com/muodov/kociemba)；

这个库有`Python`的实现版本，同时也有`C`的实现版本：

<img src="https://newbieyoung.github.io/images/threejs-rubik-auto-reset-v2-0.jpg">

`README`对`Python`实现版本的介绍很详细；我们来看看`C`的实现版本：

<img src="https://newbieyoung.github.io/images/threejs-rubik-auto-reset-v2-1.jpg">

进入源代码目录即可看到`Makefile`文件已经有了，那么接下来只需要直接编译就好了：

<img src="https://newbieyoung.github.io/images/threejs-rubik-auto-reset-v2-2.jpg">

编译时会创建`bin`目录，并把编译好的文件放入其中；

编译完成之后我们就可以直接执行了：

<img src="https://newbieyoung.github.io/images/threejs-rubik-auto-reset-v2-3.jpg">

传入特定的`魔方序列`即可得到解法了。

至于`魔方序列`是怎么回事？`README`里边解释的很清楚：

<img src="https://newbieyoung.github.io/images/threejs-rubik-auto-reset-v2-4.jpg">

简单来说就是不同面用不同字母表示，比如`U`表示顶面、`F`表示正面等；正面的意思是你用手拿着魔方时正对着你的那个面；然后按照固定的规律依次排列得到的字符串序列。

比如初始魔方序列为：

```
UUUUUUUUURRRRRRRRRFFFFFFFFFDDDDDDDDDLLLLLLLLLBBBBBBBBB
```

其实我们要使用这个算法有很多种方式，比如：用`Python`或者`Node`搭建后台，然后封装接口给前台调用即可；

但是`秉着折腾的原则`偏要浏览器里边运行算法，那这时候就需要用到`Emscripten`了，安装完成之后，在`C`的实现版本源代码目录执行以下命令：

```
emcc coordcube.c cubiecube.c facecube.c prunetable_helpers.c search.c solve.c -s WASM=1 -s EXTRA_EXPORTED_RUNTIME_METHODS='["UTF8ToString"]' -s ALLOW_MEMORY_GROWTH=1 -s EXPORTED_FUNCTIONS="['_solve']" -o kociemba.js
```

首先如果需要编译多个文件需要把它们依次列出：

```
coordcube.c cubiecube.c facecube.c prunetable_helpers.c search.c solve.c
```

然后需要设置`ALLOW_MEMORY_GROWTH=1`表示运行内存自动增长，如果不设置运行时会因为内存不够报错；

```
EXTRA_EXPORTED_RUNTIME_METHODS='["UTF8ToString"]'
EXPORTED_FUNCTIONS="['_solve']"
```

这两点也很重要，后边再说！

最后需要设置编译得到文件的文件名，`WebAssembly`文件的后缀名为`.wasm`，这里之所以设置为`.js`，主要是因为如果设置为`.wasm`，那么仅仅只能得到`WebAssembly`文件，之后你还需要自己实现加载`WebAssembly`文件，内存设置等一系列工作；

这里边坑很多，比如如果`WebAssembly`文件超过`4K`就不能使用`WebAssembly.Instance`来初始化了，而应该使用`WebAssembly.instantiate`等等...

<img src="https://newbieyoung.github.io/images/threejs-rubik-auto-reset-v2-5.jpg">

但是设置为`.js`文件，则不光会生成`WebAssembly`文件，还会顺便帮你生成一份`JS`文件，里边会做一些默认的基本操作，比如加载`WebAssembly`文件，内存设置等。

执行命令之前，如果你不对源代码做任何改动，你就会发现如下错误：

<img src="https://newbieyoung.github.io/images/threejs-rubik-auto-reset-v2-6.jpg">

大致意思是找不到`头文件`，解决办法是把源代码`include`目录里边`头文件`移出来和`源文件`同级即可。

如果不报任何错，同时会得到一份`.js`文件以及一份`.wasm`文件，那么编译过程基本就成功了。

接下来的问题，我们怎么使用呢？

首先只需要加载编译得到的`JS`文件即可：

<img src="https://newbieyoung.github.io/images/threejs-rubik-auto-reset-v2-7.jpg">

接着需要对算法源代码进行一定的修改，新增一个`solve`方法，让其暴漏出来供`JS`调用：

因此需要设置`EXPORTED_FUNCTIONS="['_solve']`，方法名默认需要加上横杆才能正确匹配。

<img src="https://newbieyoung.github.io/images/threejs-rubik-auto-reset-v2-8.jpg">

因为传入的是魔方序列字符串因此方法参数为`char * rubik`表示`char`类型的指针，同时返回的`解法`也是字符串，因此返回值类型为`char *`；

至于`solve`函数的内容则是来自源代码中的`main`函数，`int argc`默认表示参数个数，`char **argv`则表示参数数组，当我们执行`./kociemba DRLUUBFBRBLURRLRUBLRDDFDLFUFUFFDBRDUBRUFLLFDDBFLUBLRBD`时，参数只有一个，因此可以推测出`main`函数中的执行语句，复制到`solve`函数中即可。

修改完成之后别忘了重新编译。

具体执行过程如下：

<img src="https://newbieyoung.github.io/images/threejs-rubik-auto-reset-v2-9.jpg">

`Module['onRuntimeInitialized']`表示初始化完成，`_solve`方法已经被挂载到`Module`对象上了，直接执行即可；

但是在传递以及接收字符串时，需要额外处理；

比如传递字符串需要用`allocate(intArrayFromString(rubik), 'i8', ALLOC_NORMAL)`转换成内存地址；

接收时直接得到也是内存地址，要取得真正的字符串需要调用`UTF8ToString`方法，为了能直接调用该方法，需要在执行编译命令时设置`EXTRA_EXPORTED_RUNTIME_METHODS='["UTF8ToString"]'`。

各内置方法详情可以去`Emscripten官网`[http://kripken.github.io/emscripten-site/index.html](http://kripken.github.io/emscripten-site/index.html)查看。

<img src="https://newbieyoung.github.io/images/threejs-rubik-auto-reset-v2-11.jpg">
<img src="https://newbieyoung.github.io/images/threejs-rubik-auto-reset-v2-10.jpg">

在浏览器中运行结果如下：

<img src="https://newbieyoung.github.io/images/threejs-rubik-auto-reset-v2-12.jpg">

示例代码在[https://newbieyoung.github.io/Html_learn/webassembly/demo3.html](https://newbieyoung.github.io/Html_learn/webassembly/demo3.html)。

至此折腾结束！

最后剩下的工作就是把已有的魔方模型转换成`Kociemba算法`所需要的魔方序列即可。

`小魔方第六步：二阶段算法自动还原魔方`示例代码在[https://newbieyoung.github.io/Threejs_rubik/step6.html](https://newbieyoung.github.io/Threejs_rubik/step6.html);

演示如下：

<img src="https://newbieyoung.github.io/images/threejs-rubik-auto-reset-v2-13.gif">

可以明显看到`Kociemba算法`的还原效率比`层先法`高了很多。























