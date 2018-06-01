---
title: 基于Three.js的简单物理游戏 ——《Wed World》开发笔记
date: 2017-05-31 11:30:02
tags:
- JavaScript
- WebGL
- Three.js
---

## 前言
年前中了知乎的毒，打算认真写个项目作为毕业设计，刚好自己也对游戏开发和3D开发一直都很感兴趣，想尝试一下，所以就打算基于WebGL写一个3D游戏，因为WebGL的API都比较原始，开发起来非常费劲，项目里用的是目前比较流行的封装库[Three.js](https://threejs.org/)。这个游戏的灵感来自于某年百度前端技术学院一个叫做html world的题目，大意是用页面的html结构生成地图，然后操控一个小球在里面进行运动，我对这个题目做了一点扩展，增加游戏的玩法和内容，运动的物理实现也更加真实，不过这个游戏如果真的全部完成可以说有好几个毕业设计的工作量，所以来来回回写了三四个月最后只是做了一个简单的demo版本（花了不少时间赶论文……）。下面我简单讲讲这个游戏的开发思路。源代码：[点击此处](https://github.com/Orange-C/web-world)。演示地址：[点击此处](https://orange-c.github.io/web-world/)

游戏运行截图如下：

![1](/blog/images/2017053111.jpeg)

![2](/blog/images/2017053122.jpeg)

![3](/blog/images/2017053133.jpeg)

<!-- more -->

## 游戏基础设计

### 游戏规则

操控一个小球在特定地图上运动。地图上有一定数量的goal方块，在固定时间内找到所有goal方块即可通关，掉落出地图或者超时则失败

### 物理环境

主要操控角色是一个3D小球，可以在地图上进行自由的物理运动，包括多方向的加减速、跳跃。为了模拟真实的物理环境，游戏的物理实现应包含以下规则：
1.	小球具有固定的质量
2.	小球速度应通过力学模型来改变而不是直接改变
3.	游戏环境应有一个固定的重力
4.	游戏的平面部分应有一个固定的摩擦力
5.	小球碰撞时具有一定弹性

### 地图生成

基础模式有一张默认地图。可以本地启用[web-world-analyze](https://github.com/Orange-C/web-world-analyze)打开游戏分析页面入口服务器，开启游戏的分析功能。输入任意网址，根据网址获得的html结构作为种子生成地图。

## 游戏初始化
基本的设计如上，交互设计，单/双人模式等等就不细讲了，下面逐步开始介绍具体程序设计。游戏初始化做了不少工作，包括dom事件的绑定，全局变量的处理等等。这里主要讲讲和3D游戏开发有关的部分初始化。

### 贴图预加载

贴图这个概念想必游戏玩家都不陌生，这个项目里也用到了部分贴图，都是我从minecraft素材包里面手动截出来的（因为本来想和minecraft和terraria一样做个随机生成的像素风游戏）。贴图资源需要预先加载到内存中，这里在游戏启动时调用了`THREE.TextureLoader`进行贴图的预加载。

### 初始化Renderer、Scene、Camera

在`Three.js`中，核心的三个对象就是`Renderer`（渲染器）、`Scene`（场景）、`Camera`（镜头）。
* `Renderer`用于和canvas对象绑定
* `Scene`代表渲染场景，用来存储所有需要渲染的对象
* `Camera`代表镜头，用于选择投影区域和投影方式，即场景中的物体如何呈现在屏幕上

`Renderer`和`Scene`都较为模式化，谈谈`Camera`的初始化，因为是真实环境，所以肯定是采用透视投影而不是正交投影，而且视角是也是斜视视角，需要计算相应角度，初始之后还需要手动调整`Camera`的朝向，将镜头对准中心，也就是游戏开始的地方。完成之后，需要将`Camera`添加到`Scene`中，只有在`Scene`中的对象才会被渲染。具体代码如下：
```
function initCamera() {
    // 初始化config.camera
    config.camera = new THREE.PerspectiveCamera(45, WIDTH/HEIGHT, 1, 100*config.focalDistance);
    config.camera.position.set(4*config.focalDistance, 3*config.focalDistance, 5*config.focalDistance);
    config.camera.lookAt(new THREE.Vector3(0, 0, 0));
    config.scene.add(config.camera);
}
```

### 初始化光源和阴影

真实环境当然不能缺了光与影，光影的实现也是非常消耗计算资源的一部分。3D场景中光源分为几种类型，包括环境光、平行光、点光源、聚光灯等，在不同场合下组合利用，将能达到很好的光照效果。这个项目中使用了环境光`AmbientLight`和平行光`DirectionalLight`。阴影方面，只需要将光源的`castShadow`属性设为`true`即可开启，为了性能考虑用了一个相对较低的配置。如果用的是点光源，`Three.js`还提供了`cameraHelper`来帮助进行光源的调试。

## 动画循环

游戏中有一个概念叫做帧数，代表了每秒钟会刷新几次画面。这就意味着有一个循环函数，每隔一段时间执行一遍用来重绘画面，这就是游戏的动画循环函数。

最初分析需求时想到的一般都是Web前端编程中常用的`setInterval`函数，每隔一段时间执行回调函数，但是在深入了解浏览器重绘原理后，这种方案并不令人满意。`setInterval`的间隔时间并不是标准的，会根据浏览器自身的重绘有一些提前或稍后，在间隔时间较大时没有明显影响，但在常用的60fps下，时间间隔为16.7ms，这个影响会比较明显，帧数也不太稳定。最后用了新的动画API`requestAnimationFrame`，它不需要使用者指定循环间隔时间，浏览器会基于当前页面是否可见、CPU的负荷情况等来自行决定最佳的渲染时间，从而更合理地使用CPU。因为浏览器自身刷新页面时的帧数一般都在60左右，所以使用这个接口后，基本能保持主循环的帧数在60左右。
```
 function animate() {
    // 计算处理

    config.renderer.render(config.scene, config.camera);
    config.id = requestAnimationFrame(animate);
}
```

## 键盘响应

游戏中主要的操控方式都是通过键盘，熟悉js开发的同学都知道常用的事件响应基本都是回调，但是在游戏中，回调形式的响应并不适合，主要原因有：
1. 键盘响应的回调间隔远大于16.7ms，也就是说一直按键并不会在每一帧都有响应
2. 游戏的主要逻辑都是在动画循环中处理，这时候加入一个外部回调会使内部计算依赖外部回调，难以开发

所以这里的键盘响应采用了轮询的形式，浏览器提供的键盘事件只有回调，这个没办法，但是在这个回调中只是按下把相应的key值设为true，松开设为false，然后在主循环中判断相应的key值，如果为true则在主循环中进行相应操作，这样的话，整个逻辑都在主循环中处理，比较清晰，也不会有帧数问题。具体代码如下：
```
// 绑定键盘事件
function createEvents() {
    document.addEventListener('keydown', (e) => {
        e.preventDefault();
        keyboard[e.keyCode] = true;
    });
    document.addEventListener('keyup', (e) => {
        e.preventDefault();        
        keyboard[e.keyCode] = false;
    });
}

// 轮询监听，例：镜头旋转
function animate() {
    if(keyboard[79]) {
        config.camera.rotateY(Math.PI/180);
    }
    if(keyboard[80]) {
        config.camera.rotateY(Math.PI/180, true);        
    }

    config.renderer.render(config.scene, config.camera);
    config.id = requestAnimationFrame(animate);
}
```

