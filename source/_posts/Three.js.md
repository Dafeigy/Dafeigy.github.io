---
title: Three.js 学习笔记（一）
mathjax: true
tags:
- Three.js
categories:
- 学习记录
- 前端
---

<p align="center">
    <img src="https://s2.loli.net/2023/04/19/N5CW4f9JBb3TYrv.webp" alt="Three.js logo"/>
</p>

本篇是`Three.js`的入门笔记~

<!-- more -->

## Three.js 的安装

在新建一个项目文件夹后，使用`npm`安装：

```bash
npm install three
```

然后新建一个`.html`文件写入如下内容：

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title>My first three.js app</title>
		<style>
			body { margin: 0; }
		</style>
	</head>
	<body>
		<script type="module">
			import * as THREE from 'https://unpkg.com/three/build/three.module.js';

			// Our Javascript will go here.
		</script>
	</body>
</html>
```

## Three.js 的应用结构

先看代码：

```javascript
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );

const renderer = new THREE.WebGLRenderer();
renderer.setSize( window.innerWidth, window.innerHeight );
document.body.appendChild( renderer.domElement );
```

一个Three.js应用需要有如下三个东西：

* 场景（scene）：场景相当于是画布，用来容纳整个`Three.js`的相关资源。

* 相机（camera）：three.js里有几种不同的相机，在这里，我们使用的是**PerspectiveCamera**——透视相机。

  * 第一个参数是**视野角度（FOV）**。视野角度就是无论在什么时候，你所能在显示器上看到的场景的范围，它的单位是角度(与弧度区分开)。

  * 第二个参数是**长宽比（aspect ratio）**。 也就是你用一个物体的宽除以它的高的值。比如说，当你在一个宽屏电视上播放老电影时，可以看到图像仿佛是被压扁的。

  * 接下来的两个参数是**近截面**（near）和**远截面**（far）。 当物体某些部分比摄像机的**远截面**远或者比**近截面**近的时候，该这些部分将不会被渲染到场景中。或许现在你不用担心这个值的影响，但未来为了获得更好的渲染性能，你将可以在你的应用程序里去设置它。

* 渲染器（renderer）：

  除了我们在这里用到的WebGLRenderer渲染器之外，Three.js同时提供了其他几种渲染器，当用户所使用的浏览器过于老旧，或者由于其他原因不支持WebGL时，可以使用这几种渲染器进行降级。

  除了创建一个渲染器的实例之外，我们还需要在我们的应用程序里设置一个渲染器的尺寸。比如说，我们可以使用所需要的渲染区域的宽高，来让渲染器渲染出的场景填充满我们的应用程序。因此，我们可以将渲染器宽高设置为浏览器窗口宽高。对于性能比较敏感的应用程序来说，你可以使用**setSize**传入一个较小的值，例如**window.innerWidth/2**和**window.innerHeight/2**，这将使得应用程序在渲染时，以一半的长宽尺寸渲染场景。

  如果你希望保持你的应用程序的尺寸，但是以较低的分辨率来渲染，你可以在调用**setSize**时，将**updateStyle**（第三个参数）设为false。例如，假设你的<canvas> 标签现在已经具有了100%的宽和高，调用**setSize(window.innerWidth/2, window.innerHeight/2, false)**将使得你的应用程序以一半的分辨率来进行渲染。

  **最后一步很重要，**我们将**renderer**（渲染器）的dom元素（renderer.domElement）添加到我们的HTML文档中。这就是渲染器用来显示场景给我们看的<canvas>元素。

## Three.js 的渲染

首先先添加物品内容，这里以一个立方体为例：

```js
const geometry = new THREE.BoxGeometry( 1, 1, 1 );
const material = new THREE.MeshBasicMaterial( { color: 0x00ff00 } );
const cube = new THREE.Mesh( geometry, material );
scene.add( cube );

camera.position.z = 5;
```

要创建一个立方体，我们需要一个**BoxGeometry**（立方体）对象. 这个对象包含了一个立方体中所有的顶点（**vertices**）和面（**faces**）。未来我们将在这方面进行更多的探索。

接下来，对于这个立方体，我们需要给它一个材质，来让它有颜色。Three.js自带了几种材质，在这里我们使用的是**MeshBasicMaterial**。所有的材质都存有应用于他们的属性的对象。在这里为了简单起见，我们只设置一个color属性，值为**0x00ff00**，也就是绿色。这里所做的事情，和在CSS或者Photoshop中使用十六进制(**hex colors**)颜色格式来设置颜色的方式一致。

第三步，我们需要一个**Mesh**（网格）。 网格包含一个几何体以及作用在此几何体上的材质，我们可以直接将网格对象放入到我们的场景中，并让它在场景中自由移动。

默认情况下，当我们调用**scene.add()**的时候，物体将会被添加到**(0,0,0)**坐标。但将使得摄像机和立方体彼此在一起。为了防止这种情况的发生，我们只需要将摄像机稍微向外移动一些即可。

现在，如果将之前写好的代码复制到HTML文件中，你不会在页面中看到任何东西。这是因为我们还没有对它进行真正的渲染。为此，我们需要使用一个被叫做“**渲染循环**”（render loop）或者“**动画循环**”（animate loop）的东西，代码如下：

```js
function animate() {
	requestAnimationFrame( animate );
    cube.rotation.x += 0.01;
    cube.rotation.y += 0.01;
	renderer.render( scene, camera );
}
animate();
```

这段代码每帧都会执行（正常情况下是60次/秒），这就让立方体有了一个看起来很不错的旋转动画。基本上来说，当应用程序运行时，如果你想要移动或者改变任何场景中的东西，都必须要经过这个动画循环。当然，你可以在这个动画循环里调用别的函数，这样你就不会写出有上百行代码的**animate**函数。
