---
title: Chrome Network 和 Puppeteer 禁止 HTTP2
date: 2019-02-23 15:47
tags: [Chrome,Puppeteer,HTTP2]
categories: [网络协议]
author: Young
---

以前做过一个 [精灵图在 Lottie Web 动画中的应用](https://newbieweb.lione.me/2018/12/13/lottie-web-sprite/)的方案；因为要写PPT，所以需要举证这种方案的优点；

但是当我在 Chrome 浏览器的 Network 中查看优化前的 waterfall 的时候震惊了；

<img src="https://newbieyoung.github.io/images/chrome-network-http2-0.jpg">

不是说好并发请求数有限制的吗？怎么会有一百个请求同时发送？

<!--more-->

然后再去查看 Connection ID 竟然是同一个请求！

<img src="https://newbieyoung.github.io/images/chrome-network-http2-1.jpg">

瞬间就想到了很久没碰过的网络协议中有个叫 HTTP/2 的家伙。

<img src="https://newbieyoung.github.io/images/chrome-network-http2-2.jpg">

HTTP/2 协议支持多路复用，可以用同一个连接传输多个资源。

精力有限，暂时就不过多了解了，我只想好好的写个 PPT 而已！

有没有办法强制 Chrome 使用 HTTP/1.1 协议呢？

据我在网上搜索的结果来看，是可以在操作系统中通过命令行来设置让 Chrome 使用 HTTP/1.1 协议的，但是个人感觉略麻烦。

然后就想到了 Puppeteer，以前总是通过关闭沙盒模式和各种安全策略来让自己对一些 H5 游戏作弊，那改个协议的版本也应该是支持的。

<img src="https://newbieyoung.github.io/images/chrome-network-http2-3.jpg">

强制使用 HTTP/1.1 协议，然后就可以看到一百多个请求的酸爽效果了...够慢...截个图放 PPT 里边...美滋滋。

<img src="https://newbieyoung.github.io/images/chrome-network-http2-4.jpg">



