---
title: NodeJS Imagemin
date: 2019-11-06 21:21
tags: [NodeJS, imagemin, pngquant, mozjpeg]
categories: [NodeJS]
author: Young
---

`imagemin` 是目前比较常用的压缩图片的 NodeJS 库，Github 仓库地址为 [https://github.com/imagemin/imagemin](https://github.com/imagemin/imagemin)；它支持很多插件可以用有损压缩或者无损压缩的方式压缩不同类型的图片，比如：

<img src="https://newbieyoung.github.io/images/nodejs-imagemin-0.jpg">

其用法很简单：

```javascript
const imagemin = require("imagemin");
const imageminPngquant = require("imagemin-pngquant");
const imageminMozjpeg = require("imagemin-mozjpeg")(async () => {
  const files = await imagemin(["images/*.{jpg,png}"], {
    destination: "build/images",
    plugins: [
      imageminJpegtran(),
      imageminPngquant({
        quality: [0.6, 0.8]
      })
    ]
  });
})();
```

本应该没啥好记录的才对，但是当你执行 `npm install` 开始安装的时候麻烦事就来了（以 CentOS6 为例）。

<!--more-->

首先安装 imagemin 自身，一般不会出啥问题；

```bash
[root@4d694e7e1fd5 test]# npm install imagemin --save
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN test@1.0.0 No description
npm WARN test@1.0.0 No repository field.

+ imagemin@7.0.0
added 45 packages in 6.236s
```

接着安装压缩 png 图片的插件 `imagemin-pngquant`；

```bash
[root@4d694e7e1fd5 test]# npm install imagemin-pngquant --save

> pngquant-bin@5.0.2 postinstall /document/test/node_modules/pngquant-bin
> node lib/install.js

  ⚠ tunneling socket could not be established, cause=write EPROTO 139648169346848:error:140770FC:SSL routines:SSL23_GET_SERVER_HELLO:unknown protocol:../deps/openssl/openssl/ssl/s23_clnt.c:797:

  ⚠ pngquant pre-build test failed
  ℹ compiling from source
  ✖ Error: pngquant failed to build, make sure that libpng-dev is installed
```

开始报错了，imagemin-pngquant 依赖底层读写 png 图片的跨平台库 `libpng-dev`；

在 CentOS 上可以通过 `rpm -qa |grep libpng` 命令来检查是否有安装，如果没有则需要执行 `yum install libpng-devel`；

```bash
[root@4d694e7e1fd5 test]# rpm -qa |grep libpng
libpng-1.2.49-2.el6_7.x86_64
libpng-devel-1.2.49-2.el6_7.x86_64
```

libpng-dev 安装好了之后重新安装 imagemin-pngquant，结果还是报错 `Error: pngquant failed to build, make sure that libpng-dev is installed`；折腾了很久之后注意到报错之前的日志输出：

```bash
pngquant-bin@5.0.2 postinstall /document/test/node_modules/pngquant-bin
node lib/install.js
pngquant pre-build test failed
compiling from source
```

再结合 `pngquant-bin` [https://github.com/imagemin/pngquant-bin](https://github.com/imagemin/pngquant-bin) 的源代码：

<img src="https://newbieyoung.github.io/images/nodejs-imagemin-1.jpg">

可以清楚看到其安装的详细流程，先执行 `./vendor/pngquant --version` 如果执行失败则说明 pngquant-bin 提供的 pngquant 不能在当前环境中执行，然后就只能从 `source` 目录中的源码压缩包重新进行编译安装；而编译安装需要操作系统满足很多条件，比如需要有 libpng-devel 等；上面明明已经安装了 libpng-devel 还报错，则有很大可能是通过 yum 安装的版本比较低并不兼容 pngquant-bin 中提供源码的版本。

这种情况有两种解决办法，第一种是去升级 libpng-devel 的版本；但并不建议这么做，因为有非常大的可能 libpng-devel 的升级依赖操作系统的 GLIBC 版本；而 GLIBC 是 Linux 操作系统中最底层的 API 了，几乎所有运行库都会依赖它，它自身也提供了很多必要功能的实现，以我的经验去升级这种东西绝壁是害人害己的操作。

第二种办法则是自己手动安装好 `pngquant`( pngquant 是一个用 C 语言编写的 PNG 压缩开源库 )，在 CentOS 中通过 yum 安装 pngquant 时要先安装 `epel-release` 如下：

```
yum install epel-release
yum install pngquant
```

安装完成之后执行 `pngquant --version` 可以查看其版本：

```
[root@4d694e7e1fd5 node_modules]# pngquant --version
2.5.2 (October 2015)
```

再然后修改 pngquant-bin 的安装代码，去掉编译安装失败终止进程的逻辑，起个新名字 `pngquant-bin-no-exit` [https://www.npmjs.com/package/pngquant-bin-no-exit](https://www.npmjs.com/package/pngquant-bin-no-exit) 发布到 `npm`。

<img src="https://newbieyoung.github.io/images/nodejs-imagemin-2.jpg">

接着修改 imagemin-pngquant 用 pngquant-bin-no-exit 代替 pngquant-bin，同样起个新名字 `imagemin-pngquant-no-exit` [https://www.npmjs.com/package/imagemin-pngquant-no-exit](https://www.npmjs.com/package/imagemin-pngquant-no-exit) 发布到 `npm`。

<img src="https://newbieyoung.github.io/images/nodejs-imagemin-3.jpg">
<img src="https://newbieyoung.github.io/images/nodejs-imagemin-4.jpg">

最后不用再安装 imagemin-pngquant 了；改为安装 imagemin-pngquant-no-exit 。

```
[root@4d694e7e1fd5 test]# npm install imagemin-pngquant-no-exit --save

> pngquant-bin-no-exit@5.0.3 postinstall /document/test/node_modules/pngquant-bin-no-exit
> node lib/install.js

  ⚠ tunneling socket could not be established, cause=write EPROTO 140379423373088:error:140770FC:SSL routines:SSL23_GET_SERVER_HELLO:unknown protocol:../deps/openssl/openssl/ssl/s23_clnt.c:797:

  ⚠ pngquant pre-build test failed
  ℹ compiling from source
  ✖ Error: pngquant failed to build, make sure that libpng-dev is installed
npm WARN test@1.0.0 No description
npm WARN test@1.0.0 No repository field.

+ imagemin-pngquant-no-exit@8.0.0
added 237 packages in 14.783s
```

虽然还是报错，但至少进程没有被终止，`node_modules`里除了 vendor 目录中的 pngquant 可执行二进制文件并不存在之外，其它都已经 OK 了。

```
[root@4d694e7e1fd5 node_modules]# cd pngquant-bin-no-exit/
[root@4d694e7e1fd5 pngquant-bin-no-exit]# ls
cli.js    lib      node_modules  readme.md
index.js  license  package.json  vendor
[root@4d694e7e1fd5 pngquant-bin-no-exit]# cd vendor/
[root@4d694e7e1fd5 vendor]# ls
source
```

最后把手动安装在 `/usr/bin` 目录下的 pngquant 的复制到 `node_modules/pngquant-bin-no-exit/vendor` 目录下即可。

```
[root@4d694e7e1fd5 node_modules]# cd pngquant-bin-no-exit/
[root@4d694e7e1fd5 pngquant-bin-no-exit]# ls
cli.js    lib      node_modules  readme.md
index.js  license  package.json  vendor
[root@4d694e7e1fd5 pngquant-bin-no-exit]# cd vendor/
[root@4d694e7e1fd5 vendor]# ls
source
[root@4d694e7e1fd5 vendor]# pwd
/document/test/node_modules/pngquant-bin-no-exit/vendor
[root@4d694e7e1fd5 vendor]# cp /usr/bin/pngquant /document/test/node_modules/pngquant-bin-no-exit/vendor
[root@4d694e7e1fd5 vendor]# ls
pngquant  source
```

至此虽然我们还是没有安装成功，但其实整个程序已经可以正常运行了，只需要在引入稍作改动即可。

```
const imageminPngquant = require('imagemin-pngquant-no-exit')
```

对于 imagemin 的其它插件安装失败时也可以用同样的方法解决，比如压缩 JPG 的 imagemin-mozjpeg 插件；同样是依赖一个 C 语言编写的图片压缩库 `mozjpeg` [https://github.com/mozilla/mozjpeg](https://github.com/mozilla/mozjpeg)，和 pngquant 的区别在于一个压缩 PNG 图片，一个压缩 JPG 图片；另外 mozjpeg 并不能通过 yum 安装，需要下载源代码进行编译安装。

```bash
yum install autoconf automake libtool nasm make
wget https://github.com/mozilla/mozjpeg/archive/v3.1.tar.gz
tar -xvf v3.1.tar.gz
cd mozjpeg-3.1
autoreconf -fiv
mkdir build && cd build
sh ../configure
sudo make install
```

编译完成之后可以在 `/opt/mozjpeg/bin` 看到对应的二进制可执行文件。
