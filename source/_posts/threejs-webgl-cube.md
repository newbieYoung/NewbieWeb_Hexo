---
title: ThreeJS四步制作一个简易魔方
date: 2017-02-28 14:00
tags: [ThreeJS,WebGL,魔方]
categories: [3D技术]
author: Young
---

去年之所以再次兴起了学习 WebGL 的念头，主要是有两个原因：

- 其一是想制作一个魔方玩；
- 其二是想用 Web 技术还原一些经典电影的镜头，比如《Cast Away》又译《荒岛余生》中电影快结束时主人公站在十字路口的场景。

<img src="https://newbieyoung.github.io/images/threejs-webgl-cube-0.jpg">

现在看来我想第一个目的已经达成了，有点可惜的是在此之前已经有很多人做过同样的事了，比如：

+ [https://github.com/miniwangdali/SimpleRubiksCube](https://github.com/miniwangdali/SimpleRubiksCube)
+ [https://www.google.com/logos/2014/rubiks/iframe/index.html](https://www.google.com/logos/2014/rubiks/iframe/index.html)

站在前人的肩膀上整个事情简单了很多，但是解决问题所带来的成就感也相对减少了很多，这也是没有办法的事情了。

<!--more-->

## 前言

首先我假设你是一名前端工程师而且已经初步了解 WebGL 和 ThreeJS 的基础知识，比如坐标系、相机、光线、矩阵、弧度等；

## 编码

### 第一步：搭架子

从我短暂的 ThreeJS 编程经验来看，有个通用的的架构能处理大部分情况，如下：

```
//开始
function threeStart() {
    initThree();
    initCamera();
    initScene();
    initLight();
    initObject();
    render();
}
function initThree() {
	//...
}
var camera;
function initCamera() {
	//...
}
var scene;
function initScene() {
	//...
}
var light;
function initLight() {
	//...
}
var cubes
function initObject() {
	//...
}
function render(){
	//...
}
```

第一步完整代码如下：

[https://newbieyoung.github.io/Threejs_rubik/step1.html](https://newbieyoung.github.io/Threejs_rubik/step1.html)

### 第二步：画外型

魔方的外型很简单，就是由一些小正方体组成的一个大正方体而已。

用一个方法封装起来：

```
/**
 * 简易魔方
 * x、y、z 魔方中心点坐标
 * num 魔方阶数
 * len 小方块宽高
 * colors 魔方六面体颜色
 */
function SimpleCube(x, y, z, num, len, colors) {
    //魔方左上角坐标
    var leftUpX = x - num / 2 * len;
    var leftUpY = y + num / 2 * len;
    var leftUpZ = z + num / 2 * len;
    //根据颜色生成材质
    var materialArr = [];
    for (var i = 0; i < colors.length; i++) {
        var texture = new THREE.Texture(faces(colors[i]));
        texture.needsUpdate = true;
        var material = new THREE.MeshLambertMaterial({ map: texture });
        materialArr.push(material);
    }
    var cubes = [];
    for (var i = 0; i < num; i++) {
        for (var j = 0; j < num * num; j++) {
            var cubegeo = new THREE.BoxGeometry(len, len, len);
            var cube = new THREE.Mesh(cubegeo, materialArr);
            //依次计算各个小方块中心点坐标
            cube.position.x = (leftUpX + len / 2) + (j % num) * len;
            cube.position.y = (leftUpY - len / 2) - parseInt(j / num) * len;
            cube.position.z = (leftUpZ - len / 2) - i * len;
            cubes.push(cube)
        }
    }
    return cubes;
}

```

基本都是些 ThreeJS 对象的简单运用，比如盒子对象`BoxGeometry`、纹理`Texture`、材质`MeshLambertMaterial`等，纹理主要是用来描述物体表面静态属性的对象，材质主要是用来描述物体表面动态属性的对象，比如处理光照等。

其中`faces`方法主要是生成一块黑色边框的大正方形其内部是某种颜色填充的圆角小正方形的 Canvas 画布，用来充当纹理渲染魔方中小正方体的某个面。

```
//生成canvas素材
function faces(rgbaColor) {
    var canvas = document.createElement('canvas');
    canvas.width = 256;
    canvas.height = 256;
    var context = canvas.getContext('2d');
    if (context) {
    	//画一个宽高都是256的黑色正方形
        context.fillStyle = 'rgba(0,0,0,1)';
        context.fillRect(0, 0, 256, 256);
        //在内部用某颜色的16px宽的线再画一个宽高为224的圆角正方形并用改颜色填充
        context.rect(16, 16, 224, 224);
        context.lineJoin = 'round';
        context.lineWidth = 16;
        context.fillStyle = rgbaColor;
        context.strokeStyle = rgbaColor;
        context.stroke();
        context.fill();
    } else {
        alert('您的浏览器不支持Canvas无法预览.\n');
    }
    return canvas;
}
```

如果把这个 Canvas 画布渲染出来，大致是下边这样的：

<img src="https://newbieyoung.github.io/images/threejs-webgl-cube-5.png">

另外基于魔方中心在坐标系原点从而推算出所有小正方体中心点坐标可以画图理解如下：

<img src="https://newbieyoung.github.io/images/threejs-webgl-cube-4.jpg">

最后需要把生成的魔方加入到场景中才会被渲染出来：

```
//创建展示场景所需的各种元素
var cubes
function initObject() {
	//生成魔方小正方体
	cubes = SimpleCube(cubeParams.x,cubeParams.y,cubeParams.z,cubeParams.num,cubeParams.len,cubeParams.colors);
	for(var i=0;i<cubes.length;i++){
		var item = cubes[i];
		scene.add(cubes[i]);//并依次加入到场景中
	}
}
```

第二步完整代码如下：

[https://newbieyoung.github.io/Threejs_rubik/step2.html](https://newbieyoung.github.io/Threejs_rubik/step2.html)

此时在浏览器中运行第二步完整代码应该是下边这个样子的：

<img src="https://newbieyoung.github.io/images/threejs-webgl-cube-7.png">

相比于第一步一片空白的页面而言，页面中多了一个类似九宫格的正方形，有人可能会说大兄弟要画这么个玩意用得着 ThreeJS 吗，DIV + CSS 分分钟搞定......

其实之所以会这样是因为我们设置的`相机`的位置是在坐标系的Z轴，魔方的中心在坐标系原点，它们刚好处于同一条直线上，导致显示出来的是魔方的正视图。

### 第三步：操控魔方视角

第二步完成之后有个很严重的问题，我们只能看到魔方的正面，为了解决这个问题我们需要让`相机`随着鼠标或者触摸点的移动而移动；

在 ThreeJS 中作者提供了很多种视角控制类库，比如：

+ 轨迹球控件`TrackballControls`(最常用的控件,用鼠标控制相机移动和转动)；

+ 飞行控件`FlyControls`(飞行模拟器控件,用键盘和鼠标控制相机移动和旋转)；

+ 翻滚控件`RollControls`(翻滚控件是飞行控件的简化版,控制相机绕Z轴旋转)；

+ 第一人称控件`FirstPersonControls`(类似于第一人称视角的相机控件)；

+ 轨道空间`OrbitControls`(类似于轨道中的卫星,控制鼠标和键盘在场景中游走)；

+ 路径控件`PathControls`(控制相机在预定义的轨道上移动和旋转)；

在这里我使用 OrbitControls 控制器，具体用法很简单如下：

首先下载代码并引入：

<img src="https://newbieyoung.github.io/images/threejs-webgl-cube-25.jpg">

```
<script type="text/javascript" src="./threejs/controls/OrbitControls.js"></script>
```

然后根据相机以及画布初始化即可：

```
//创建相机，并设置正方向和中心点
var camera;
var controller;//视角控制器
function initCamera() {
    camera = new THREE.PerspectiveCamera(45, width / height, 1, 1000);
    camera.position.set(0, 0, 600);
    camera.up.set(0, 1, 0);//正方向
    camera.lookAt(origPoint);
    //视角控制
    controller = new THREE.OrbitControls(camera, renderer.domElement);
    controller.target = origPoint;//设置控制点
}
```

第三步完整代码如下：

[https://newbieyoung.github.io/Threejs_rubik/step3.html](https://newbieyoung.github.io/Threejs_rubik/step3.html)

此时在浏览器中运行第三步完整代码应该是下边这个样子的：

<img src="https://newbieyoung.github.io/images/threejs-webgl-cube-10.gif">

### 第四步：转动魔方

经过前三步在视觉方面简易魔方已经完成了差不多了，但是依然欠缺很重要的东西，没办法转动连最基本的可玩性都没有；

要想转动魔方需要解决以下几个问题：

#### 1、首先得确定触摸点

也就是说必须得在代码里边判断出魔方的哪个部位被触摸了，Canvas 编程是没办法像 DOM 编程那样有完备的事件机制支持的；所以这个问题需要其它解决办法，比如在 2D Canvas 我们可以根据坐标来判断当前鼠标或者触摸点在哪个元素上，从而假定该元素获得了焦点；但是在 WebGL 中存在一个平面 2D 坐标映射为 3D 坐标的问题，万幸 ThreeJS 也提供了对应的解决方案`Raycaster`。

简单来说就是模拟一道光从屏幕点击或者触摸的位置上开始，以相机朝向为方向，然后检测光线与物体的碰撞，可以得知距离、碰撞点以及哪些物体先碰撞哪些物体慢碰撞。

首先得知道在页面的 2D 坐标，这里可以通过监听鼠标事件或者触摸事件来完成；

```
//监听鼠标事件
renderer.domElement.addEventListener('mousedown', startCube, false);
renderer.domElement.addEventListener('mousemove', moveCube, false );
renderer.domElement.addEventListener('mouseup', stopCube,false);
//监听触摸事件
renderer.domElement.addEventListener('touchstart', startCube, false);
renderer.domElement.addEventListener('touchmove', moveCube, false);
renderer.domElement.addEventListener('touchend', stopCube, false);
```

`Raycaster`的调用也很简单，但是需要注意的是当刚开始的接触点在魔方上且魔方没有转动时操作为转动魔方，此时屏蔽控制器转动
（控制器的`enabled`属性设置为 false 即可）；反之操作为控制器转动（控制器的`enabled`属性设置为 true ）；

魔方是否正在转动，这里用`isRotating`变量控制，开始一次转动时设置为 true，转动结束之后才还原为 false；而且一下转动操作必须等当前转动动画结束之后才可以被触发。

```
//开始操作魔方
function startCube(event){
    getIntersects(event);
    //魔方没有处于转动过程中且存在碰撞物体
    if(!isRotating&&intersect){
        startPoint = intersect.point;//开始转动，设置起始点
        controller.enabled = false;//当刚开始的接触点在魔方上时操作为转动魔方，屏蔽控制器转动
    }else{
        controller.enabled = true;//当刚开始的接触点没有在魔方上或者在魔方上但是魔方正在转动时操作转动控制器
    }
}
//获取操作焦点以及该焦点所在平面的法向量
function getIntersects(event){
	//触摸事件和鼠标事件获得坐标的方式有点区别
    if(event.touches){
        var touch = event.touches[0];
        mouse.x = (touch.clientX / width)*2 - 1;
        mouse.y = -(touch.clientY / height)*2 + 1;
    }else{
        mouse.x = (event.clientX / width)*2 - 1;
        mouse.y = -(event.clientY / height)*2 + 1;
    }
    raycaster.setFromCamera(mouse, camera);
    //Raycaster方式定位选取元素，可能会选取多个，以第一个为准
    var intersects = raycaster.intersectObjects(scene.children);
    if(intersects.length){
        try{
            if(intersects[0].object.cubeType==='coverCube'){
                intersect = intersects[1];
                normalize = intersects[0].face.normal;
            }else{
                intersect = intersects[0];
                normalize = intersects[1].face.normal;
            }
        }catch(err){
            //nothing
        }
    }
}
```

#### 2、然后得确定转动方向

转动魔方时应该是存在有六个方向的，分别是`X轴正方向`、 `X轴负方向`、`Y轴正方向`、`Y轴负方向`、`Z轴正方向`、`Z轴负方向`；

```
//魔方转动的六个方向
var xLine = new THREE.Vector3( 1, 0, 0 );//X轴正方向
var xLineAd = new THREE.Vector3( -1, 0, 0 );//X轴负方向
var yLine = new THREE.Vector3( 0, 1, 0 );//Y轴正方向
var yLineAd = new THREE.Vector3( 0, -1, 0 );//Y轴负方向
var zLine = new THREE.Vector3( 0, 0, 1 );//Z轴正方向
var zLineAd = new THREE.Vector3( 0, 0, -1 );//Z轴负方向
```

先根据滑动时的两点确定转动向量，然后判断转动向量和这六个方向向量夹角最小的方向即为转动方向；

```
//获得旋转方向
function getDirection(vector3){
    var direction;
    //判断差向量和x、y、z轴的夹角
    var xAngle = vector3.angleTo(xLine);
    var xAngleAd = vector3.angleTo(xLineAd);
    var yAngle = vector3.angleTo(yLine);
    var yAngleAd = vector3.angleTo(yLineAd);
    var zAngle = vector3.angleTo(zLine);
    var zAngleAd = vector3.angleTo(zLineAd);
    var minAngle = min([xAngle,xAngleAd,yAngle,yAngleAd,zAngle,zAngleAd]);//最小夹角
    switch(minAngle){
        case xAngle:
            direction = 0;//向x轴正方向旋转90度（还要区分是绕z轴还是绕y轴）
            //...
            break;
        case xAngleAd:
            direction = 1;//向x轴反方向旋转90度
            //...
            break;
        case yAngle:
            direction = 2;//向y轴正方向旋转90度
            //...
            break;
        case yAngleAd:
            direction = 3;//向y轴反方向旋转90度
            //...
            break;
        case zAngle:
            direction = 4;//向z轴正方向旋转90度
            //...
            break;
        case zAngleAd:
            direction = 5;//向z轴反方向旋转90度
            //...
            break;
        default:
            break;
    }
    return direction;
}
```

但是光知道方向其实还是不能够转动魔方的，比如下图中从点`E`滑动到点`F`和从点`G`滑动到点`H`，滑动方向都是`X轴的正方向`，而且还有其它情况滑动方向是`X轴正方向`的；对魔方来说这完全是两种不同的情形，所以我们还需要知道是在哪个平面滑动的。

<img src="https://newbieyoung.github.io/images/threejs-webgl-cube-15.jpg">

判断是在哪个平面，我们可以通过该平面的法向量和哪个坐标轴平行来判断，比如如果滑动平面的法向量平行于坐标系的Y轴且等于Y轴正方向的单位向量，那么该滑动平面肯定是魔方的`上平面`，以此类推；上边那个判断转动方向的方法可以优化为如下这个样子：

```
//获得旋转方向
function getDirection(vector3){
    var direction;
    //判断差向量和x、y、z轴的夹角
    var xAngle = vector3.angleTo(xLine);
    var xAngleAd = vector3.angleTo(xLineAd);
    var yAngle = vector3.angleTo(yLine);
    var yAngleAd = vector3.angleTo(yLineAd);
    var zAngle = vector3.angleTo(zLine);
    var zAngleAd = vector3.angleTo(zLineAd);
    var minAngle = min([xAngle,xAngleAd,yAngle,yAngleAd,zAngle,zAngleAd]);//最小夹角
    switch(minAngle){
        case xAngle:
            direction = 0;//向x轴正方向旋转90度（还要区分是绕z轴还是绕y轴）
            if(normalize.equals(yLine)){
                direction = direction+0.1;//绕z轴顺时针
            }else if(normalize.equals(yLineAd)){
                direction = direction+0.2;//绕z轴逆时针
            }else if(normalize.equals(zLine)){
                direction = direction+0.3;//绕y轴逆时针
            }else{
                direction = direction+0.4;//绕y轴顺时针
            }
            break;
    //...
```

那么接下来的问题就是怎么获得滑动平面的法向量了，所幸 ThreeJS 的光线碰撞检测机制除了能得到碰撞物体、碰撞点，还能得到碰撞平面；已知平面那么就可以获得平面法向量了。

但是 ThreeJS 中有个问题需要我们注意，在 ThreeJS 中存在物体自身坐标系和世界坐标系的区分，在初始化时物体的坐标和世界坐标系一致，但是当物体发生变化之后它自身的坐标系也是会发生变化的；比如说刚开始某个物体`上平面`的法向量就是其自身坐标系Y轴正方向的单位向量，同时也是世界坐标系Y轴正方向的单位向量，如果该物体旋转 90 度之后，其`上平面`的法向量还是其自身坐标系Y轴正方向的单位向量，但是却是世界坐标系Z轴正方向的单位向量了，如图：

<img src="https://newbieyoung.github.io/images/threejs-webgl-cube-17.jpg">

所以不能使用魔方中小正方体的碰撞平面，因为小正方体的坐标系是会随着小正方体的变化而变化的，此时需要再加入一个和魔方整体大小一样的透明正方体，然后根据该透明正方体的碰撞平面的法向量来判断。

```
var cubes
function initObject() {
	//...
	//透明正方体
    var cubegeo = new THREE.BoxGeometry(150,150,150);
    var hex = 0x000000;
    for ( var i = 0; i < cubegeo.faces.length; i += 2 ) {
        cubegeo.faces[ i ].color.setHex( hex );
        cubegeo.faces[ i + 1 ].color.setHex( hex );
    }
    var cubemat = new THREE.MeshBasicMaterial({vertexColors: THREE.FaceColors,opacity: 0, transparent: true});
    var cube = new THREE.Mesh( cubegeo, cubemat );
    cube.cubeType = 'coverCube';
    scene.add( cube );
}
```

#### 3、再然后我们得根据转动方向、触发点获取转动物体

比如上图中从点 G 滑动到点 H，转动物体是魔方`上平面`的所有小正方体；

至于怎么判断，有两种方法，第一种可以根据小正方体的中心点来判断，比如如果转动的是魔方`上平面`的正方体，那么已知触发点所在正方体的中心点，根据其Z轴大小就可以确定其它小正方体了；

还有一种办法则是根据小正方体初始化时的编号规律来判断，转动之后更新编号，保证其规律不发生变化，后续判断依旧即可，从下边的简图很容易就能看出其编号规律。

<img src="https://newbieyoung.github.io/images/threejs-webgl-cube-19.jpg">

比如：

- `A/9==0` A 层小方块序号整除 9 都等于 0；
- `B/9==1` B 层小方块序号整除 9 都等于 1；
- `C/9==2` C 层小方块序号整除 9 都等于 2；
- `D%3==2` D 层小方块序号取余 3 都等于 2；
- `E%3==1` E 层小方块序号取余 3 都等于 1；
- `F%3==0` F 层小方块序号取余 3 都等于 0；
- `H%9/3==0` H 层小方块序号取余 9 然后再整除 3 都等于 0；
- `I%9/3==1` I 层小方块序号取余 9 然后再整除 3 都等于 1；
- `J%9/3==2` J 层小方块序号取余 9 然后再整除 3 都等于 2。

#### 4、最后是制作转动动画

制作转动动画的过程中使用 RequestAnimationFrame，这没什么要说的；唯一要注意的地方还是关于物体自身坐标系和世界坐标系的问题，举例来说，绕世界坐标系Y轴旋转的方法应该是如下图所示：

```
//绕着世界坐标系的某个轴旋转
function rotateAroundWorldY(obj,rad){
    var x0 = obj.position.x;
    var z0 = obj.position.z;
    var q = new THREE.Quaternion();
    q.setFromAxisAngle( new THREE.Vector3( 0, 1, 0 ), rad );
    obj.quaternion.premultiply( q );
    //obj.rotateY(rad);
    obj.position.x = Math.cos(rad)*x0+Math.sin(rad)*z0;
    obj.position.z = Math.cos(rad)*z0-Math.sin(rad)*x0;
}
```

#### 5、另外还要注意的是这里的转动魔方会对转动视角有影响

这和`OrbitControls`控制器的实现有关系；虽然该控制器为了兼容移动端和PC端同时支持触摸事件和鼠标事件，但是大体逻辑差不多，我们以移动端的触摸事件为例子来简单说明一下：

首先这个控制器会记录下刚开始触摸时的位置为触摸起点；

```
function onTouchStart( event ) {
	if ( scope.enabled === false ) return;
	event.preventDefault();
	switch ( event.touches.length ) {
		case 1:	// one-fingered touch: rotate
			if ( scope.enableRotate === false ) return;
			handleTouchStartRotate( event );
			state = STATE.TOUCH_ROTATE;
			break;
		//...
```

```
function handleTouchStartRotate( event ) {
	//console.log( 'handleTouchStartRotate' );
	rotateStart.set( event.touches[ 0 ].pageX, event.touches[ 0 ].pageY );
}
```

然后通过这个触摸起点和后续的触摸点计算位移数据；这逻辑初一看起来很完美，但是该控制器还提供了`enabled`属性用来对其本身起开关的作用；放到当前例子中来说，转动魔方前控制器为开启状态，我们准备转动魔方时刚一接触屏幕控制器会记录下触摸起点，然后控制器被设置为关闭状态，`由于我并没有在完成魔方转动动画后立即恢复控制器为开启状态，而是在下一次触摸开始时重新进行状态判断`导致如果重新进行状态判断的逻辑晚于控制器记录触摸起点执行就会出现`先转动魔方然后转动视角，视角会突然变化，这是因为控制器记录的触摸起点还为转动魔方时的触摸起点`。

<img src="https://newbieyoung.github.io/images/threejs-webgl-cube-24.gif">

解决办法为`让控制器的事件监听晚于魔方的事件监听执行`，另外还得注意得让魔方的事件监听挂在画布上，因为控制器的事件监听是挂在画布上的，如果魔方的事件监听挂在全局window对象上，那么就算初始化控制器的代码写在后边，依然会是魔方的事件监听先执行，因为事件监听都被设置为在事件捕获阶段执行。

```
//开始
function threeStart() {
    initThree();
    initCamera();
    initScene();
    initLight();
    initObject();
    render();
    //监听鼠标事件
    renderer.domElement.addEventListener('mousedown', startCube, false);
    renderer.domElement.addEventListener('mousemove', moveCube, false );
    renderer.domElement.addEventListener('mouseup', stopCube,false);
    //监听触摸事件
    renderer.domElement.addEventListener('touchstart', startCube, false);
    renderer.domElement.addEventListener('touchmove', moveCube, false);
    renderer.domElement.addEventListener('touchend', stopCube, false);
    //视角控制
    controller = new THREE.OrbitControls(camera, renderer.domElement);
    controller.target = new THREE.Vector3(0, 0, 0);//设置控制点
}
```

> 还有一种办法是在魔方转动动画完成后立即恢复控制器状态为开启状态。

第四步完整代码如下：

[https://newbieyoung.github.io/Threejs_rubik/step4.html](https://newbieyoung.github.io/Threejs_rubik/step4.html)

## 拓展

至此一个简易魔方完成了，是时候开下脑洞了；

+ 淘宝上应该只能买到低阶魔方，但是对于这个例子稍加拓展，你甚至可以玩 100 阶魔方；

+ 这个例子可以和微信小游戏结合，比如：

<img src="https://newbieyoung.github.io/images/threejs-webgl-cube-26.jpg">

+ 这个例子稍加扩展结合摄像头和自动还原算法，应该是可以做到扫描现实中的魔方，然后根据自动还原算法还原，得到一步步还原魔方的动画演示例子的。









