---
title: 运维相关杂记
date: 2017-12-21 18:00
tags: [Nodejs,Docker,Centos,Nginx,PHP-FPM]
categories: [其它]
author: Young
---

折腾服务器的踩坑流水账......

<!--more-->

## Nginx 关闭与重启

```
-s signal     : send signal to a master process: stop, quit, reopen, reload
```

执行`nginx -h`查看帮助说明时可以看到执行`nginx -s`+`对应命令`可以关闭或者启动nginx；

然后我们实际操作时却发现并不是这么简单：

<img src="https://newbieyoung.github.io/images/nginx-php-fpm-0.jpg">

从上图可以很容易看到关闭之后再启动会报错，大致意思是`nginx.pid`这个文件不存在；

`nginx.pid`是系统用来存储 Nginx 的主进程相关信息的文件（比如进程号）；

```
pid  logs/nginx.pid;

events {
    use epoll;
    worker_connections  2048;
}
```

可以通过`nginx.conf`文件中`pid`来指定；

<img src="https://newbieyoung.github.io/images/nginx-php-fpm-1.jpg">

启动之后就可以在指定的目录中看到该文件了；

<img src="https://newbieyoung.github.io/images/nginx-php-fpm-2.jpg">

但是关闭 Nginx 之后会发现该文件消失了，然后`reopen`和`reload`命令需要通过`nginx.pid`获取进程，如果不存在就会报错了。

```
sudo nginx -c /usr/local/etc/nginx/nginx.conf
```

解决办法是通过`nginx -c`指定配置文件启动。

## PHP-FPM 关闭与重启

启动`PHP-FPM`很简单，只需要执行以下命令即可：

```
sudo php-fpm
```

需要注意的是`PHP-FPM`默认使用的`9000`端口，如果该端口已经被其它程序使用，那么启动就会报错，这时需要关闭占用端口的程序；

```
lsof -i :9000
```

先用`lsof`命令查出占用端口的`PID`，然后执行`KILL`命令即可；

除了上述方法外，我们还可以更改`PHP-FPM`的配置，来解决问题或者进行一些优化，配置文件一般名为`php-fpm.conf`；

部分属性列举如下：

- `pid`和`nginx`类似均是主进程相关信息文件路径；
- `error_log`错误日志；
- `listen`监听端口。

关闭`PHP-FPM`则需要手动杀掉相关进程了；

```
ps -ef | grep php-fpm
```

上述命令可以查询出所有的`PHP-FPM`进程，然后执行`KILL`命令即可。

## session_start() failed: Permission denied

这种情况很有可能是写入 session 文件时有权限问题，因此首先我们需要找到 session 文件的保存路径；

在`php.ini`文件中存在相关定义：

<img src="https://newbieyoung.github.io/images/nginx-php-fpm-7.jpg">

但是注意这里的定义被注释了，也就是不起作用；

那么还可以去`PHP_FPM`的`www.conf`文件中去找：

<img src="https://newbieyoung.github.io/images/nginx-php-fpm-8.jpg">

找到 session 文件的路径后就可以查看用户、用户组以及其它信息了；

<img src="https://newbieyoung.github.io/images/nginx-php-fpm-9.jpg">

需要注意的是 session 文件的用户和用户组需要和执行`PHP-FPM`的用户和用户组相同，否则就会出现上述错误。

查询`PHP-FPM`进程相关信息可以通过以下命令：

```
ps -ef | grep php-fpm.conf
```

<img src="https://newbieyoung.github.io/images/nginx-php-fpm-10.jpg">

红框中的 UID 即为该进程用户，其中 + 号表示省略的意思。

更改 session 文件的用户和用户组则可以使用以下命令：

```
chown -R -v newbieweb:newbieweb session
```

## 查找 Nginx 配置文件

Nginx 的配置文件一般在`nginx.conf`文件中，因此可以使用查找文件命令`locate`去查找；

```
locate nginx.conf
```

但是往往服务器会存在多个 nginx.conf 文件，要从多个文件中找出实际调用的配置文件，可以参考如下流程：

首先找出 Nginx 的路径

```
ps aux|grep nginx
```

<img src="https://newbieyoung.github.io/images/nginx-php-fpm-11.jpg">

然后使用 Nginx 的 -t 参数进行配置检查，即可知道实际调用的配置文件路径了。

<img src="https://newbieyoung.github.io/images/nginx-php-fpm-12.jpg">

## 根据错误日志排查问题

在构建开发环境时我们会遇到各种各样的问题，处理这些的一般方法是分析日志文件；

- `Nginx`的错误日志文件，定义在`nginx.conf`配置文件中；

<img src="https://newbieyoung.github.io/images/nginx-php-fpm-3.jpg">

- Web 服务的错误日志文件，定义在 Web 服务的相关配置信息中；

<img src="https://newbieyoung.github.io/images/nginx-php-fpm-4.jpg">

- `PHP-FPM`的错误日志文件，定义在`php-fpm.conf`配置文件中；

<img src="https://newbieyoung.github.io/images/nginx-php-fpm-5.jpg">

- `PHP`的错误日志文件，定义在`php.ini`文件中。

<img src="https://newbieyoung.github.io/images/nginx-php-fpm-6.jpg">

## Mac OS X 使用 sz/rz 命令下载/上传文件配置

一般管理远程Linux服务器时，经常需要与本地交互文件；可以有很多种方式，比如FTP、SSH等；通常SSH支持的文本传输协议主要有ASCII、Xmodem、Zmodem等；而rz、sz则是进行Zmodem文件传输的命令行工具。

但是在MAC操作系统中，默认终端无法直接通过Zmodem来上传和下载文件，我们需要使用iTerm，具体操作请查看[Mac OS X 使用 sz/rz 命令下载/上传文件配置](http://www.xheldon.com/how-to-use-rz-sz-on-mac.html)。

如果按照上述文章的指引操作之后还是有问题，请仔细检查`文件权限`和`文件路径`是否匹配正常。

## 源代码安装Nodejs

+ 去[Nodejs官网](http://nodejs.cn/download/releases/)下载你需要的版本的源代码；

+ 本地和服务器进行文件交互，上传Nodejs源代码；

+ 解压下载文件（假设源代码文件为node-v7.9.0.tar.gz）：
```
tar -xzvf node-v7.9.0.tar.gz //解压 tar.gz
tar -xvf node-v7.9.0.tar //解压 tar
```

+ 切换工作目录至源代码解压后的目录；
```
cd node-v7.9.0
```

+ 编译码代码；
```
make
```

+ 查看`Nodejs`版本，确定是否安装成功。
```
node -v
```

+ 安装
```
make install
```

## 二进制免编译安装Nodejs

因为编译安装会受系统其它因素的影响，比如`gcc版本`等，实际操作时有很大可能会出现异常，因此可以采用更简单的二进制免编译安装的方式；

+ 去[Nodejs官网](http://nodejs.cn/download/releases/)下载你需要的版本的编译好的二进制数据包；

+ 本地和服务器进行文件交互，上传编译好的二进制Nodejs数据包；

+ 解压下载文件（假设编译好的二进制数据包文件为node-v7.10.0-linux-x64.tar.xz）；
```
tar.gz 文件格式有两层压缩，外层是xz压缩，内层是tar压缩，所以需要分两步实现解压
xz -d node-v7.10.0-linux-x64.tar.xz
tar -xvf node-v7.10.0-linux-x64.tar
```

+ 创建软链使node和npm命令全局有效：

首先你得用`whereis node`查询系统之前的Nodejs所在的目录；

如果查询结果有两个那么你得创建两个软链；
```
ln -sf /yourpath/node-v7.10.0-linux-x64/bin/node /usr/bin/node
ln -sf /yourpath/node-v7.10.0-linux-x64/bin/node /usr/local/bin/node
ln -sf /yourpath/node-v7.10.0-linux-x64/bin/npm  /usr/bin/npm
ln -sf /yourpath/node-v7.10.0-linux-x64/bin/npm  /usr/local/bin/npm
```

创建的过程中会报错`ln: creating symbolic link : File exists`；这时你需要先删除以前的node程序：
```
rm -rf /usr/bin/node
rm -rf /usr/local/bin/node
rm -rf /usr/bin/npm
rm -rf /usr/local/bin/npm
```

+ 查看`Nodejs`版本，确定是否安装成功。
```
node -v
```

## Nodejs项目部署发布相关

虽然Nodejs是一个开放源代码、跨平台的JavaScript语言运行环境，但并不意味着你在MAC操作系统上运行正常的项目，简单打包上传到其它不同操作系统的机器就可以正常运行。

特别是服务器不能联网的情况下，就不能执行npm install；另外 Nodejs 中除了有 javascript 模块，还有一些功能使用 javascript 可能达不到性能要求 ，这时候就需要 c++ 模块。

这些 c++ 模块的编译安装需要依赖本地系统的环境，导致了上述问题。

面对这种情况，有如下建议：

#### 1、本地使用Docker搭建和内网机器一样的环境，然后在Docker容器中执行npm install安装依赖；

比如内网机器操作系统为centos；

+ 首先你可以去[docker hub](https://hub.docker.com/)找到centos镜像，下载到本地；
```
docker pull centos
docker images
```

+ 然后启动centos镜像；
```
docker run -i -t centos /bin/bash
```

+ 使用yum或者其它方式安装必备软件，比如git、vim、node、zip等；
```
yum install git
```

+ 有些软件安装之后可能需要重启容器；
```
docker ps
docker restart container_id
```

+ 重启之后再次进入容器终端；
```
docker attach container_id
```

+ 通过git把项目拷贝到容器中；
```
git clone project_url project_name
```

+ 安装相关依赖；
```
npm install
```

+ 打包安装依赖之后的项目；
```
zip -r project_name.zip priject_path
```

+ 把打包之后的文件，从容器中拷贝到本地机器；
```
docker cp container_id:container_file_path local_file_path // 容器拷贝至本地
docker cp local_file_path container_id:container_file_path // 本地拷贝至容器
```

+ 为了避免频繁上传的操作需要先自测完成，这时候需要在Docker容器里边部署web服务，然后本地测试，要想本地访问容器中的web服务需要映射容器的端口为本地端口；
```
docker run -idt -p 8080:8888 image-name /bin/bash
```
8080 对应 docker 容器中 web 服务监听的端口，8888 对应本地访问的端口；映射完端口并启动镜像后进入容器启动服务即可。

+ 当你对该容器做了一些自定义工作后希望保存下来方便下次直接使用，可以使用commit命令；
```
docker commit container-id  image-name[:version]
```

+ 最后把本地得到的项目文件上传到服务器，解压即可运行。

#### 2、离线安装或者更新相关类库及其依赖；

上述`建议1`其实只能解决操作系统之间的环境差异，有可能得到的项目还是不能运行，因为即使操作系统相同，还是有可能存在比如gcc版本不一致导致编译的c++模块运行错误的问题，这时候就需要在没有网络的情况下升级服务器的相关类库（如果这种操作被允许的话...）。

比如内网机器操作系统为`centos`；

+ 在Docker容器中使用yum安装`downloadonly`插件；
```
(RHEL5)
yum install yum-downloadonly
(RHEL6)
yum install yum-plugin-downloadonly
```

+ 使用downloadonly插件只下载安装某个插件所依赖的所有文件；
```
yum install --downloadonly --downloaddir=dir-path package-name
```

+ 把相关文件统一压缩；
```
zip -r rpms.zip rpms_path
```

+ 把统一压缩的文件从容器中复制到本地机器；
```
docker cp container_id:container_file_path local_file_path 
```

+ 把统一压缩的文件从本地上传到内网机器，解压之后采用本地安装的方式安装；
```
yum localinstall *
```

