---
title: Three.js相机控件OrbitControls
date: 2025-12-17 13:45:17
tags:
  - three
categories:
  - 教程
keywords: "Three.js, OrbitControls, 3D, 相机控件, WebGL"
---
## Three.js相机控件OrbitControls

通过Three.js的相机控件OrbitControls.js可以对Threejs的三维场景进行缩放、平移、旋转操作，本质上改变的并不是场景，而是相机的参数，通过前面的学习应该知道，相机的位置角度不同，同一个场景的渲染效果是不一样，比如一个相机绕着一个场景旋转，就像场景旋转一样。

如果你想深入了解相机控制器OrbitControls的每一个功能，OrbitControls是如何对Three.js正投影相机和透视投影相机对象进行封装的，可以阅读Three.js`\examples\js\controls`目录下的`OrbitControls.js`文件。

调用OrbitControls的时候需要引入OrbitControls.js文件。

```html
<!-- 引入three.js相机控件OrbitControls.js -->
<script src="../../three.js-master/examples/js/controls/OrbitControls.js"></script>
```

#### 旋转缩放平移

如果你想实现一个场景旋转缩放平移的效果，直接把相机对象`camera`作为`OrbitControls`构造函数的参数就可以。

鼠标操作：通过拖动鼠标左键可以720旋转展示三维场景，通过拖动鼠标右键可以平移三维场景，通过上下滚动鼠标中键可以缩放三维场景。

```javascript
//创建控件对象  相机对象camera作为参数   
// 控件可以监听鼠标的变化，改变相机对象的属性
var controls = new THREE.OrbitControls(camera);
```

#### 禁止旋转平移缩放(`.enablePan`属性)

比如一个展示一个三维场景，比如一辆轿车产品，你不希望鼠标右键拖动会产生一个平移效果。可以通过设置相机空间对象OrbitControls的`.enablePan`属性，查看OrbitControls源码可以看到`.enablePan`属性的默认值是true。

```javascript
controls.enablePan = false; //禁止右键拖拽
```

通过`.enableZoom`属性可以控制是否允许鼠标中键缩放场景，`.enableZoom`属性默认值true。

```JavaScript
controls.enableZoom = false;//禁止缩放
```

通过`.enableRotate`属性可以控制是否允许鼠标左键旋转场景，`.enableRotate`属性默认值true。

```javascript
controls.enableRotate = false; //禁止旋转
```

#### 设置缩放范围

在实际应用中，你想控制一个产品或一个房间户型的缩放范围，使用的正投影相机对象`OrthographicCamera`，可以通过相机空间OrbitControls的`.minZoom`和`.maxZoom`属性实现

```javascript
// 缩放范围
controls.minZoom = 0.5;
controls.maxZoom = 2;
```

对于透视投影相机情况，控制缩放通过相机空间OrbitControls的`.minDistance`和`.maxDistance`属性实现，控制缩放范围的原理就是限制相机距离目标观察点的距离来实现。

```javascript
//透视投影相机：相机距离目标观察点距离越远显示越小，距离越近显示越大

//相机距离观察目标点极小距离——模型最大状态
controls.minDistance = 200;
//相机距离观察目标点极大距离——模型最小状态
controls.maxDistance = 500;
```

#### 设置旋转范围

展示一个三维场景，你想控制360度旋转范围，比如一辆轿车，你不希望用户看到轿车的底盘，你可以通过设置相机的旋转范围属性来实现。

通过`.minPolarAngle`和`.maxPolarAngle`属性控制上下360度的旋转范围，通过`.minAzimuthAngle`和`.maxAzimuthAngle`属性控制左右360度的旋转范围，上下左右两个360度旋转也就是常说的720旋转展示。

```javascript
// 上下旋转范围
controls.minPolarAngle = 0;
controls.maxPolarAngle = Math.PI;
// 左右旋转范围
controls.minAzimuthAngle = -Math.PI * (100 / 180);
controls.maxAzimuthAngle = Math.PI * (100 / 180);
```

#### 方向键漫游

在一个大场景中，通过相机空间OrbitControls操作键盘方向键可以控制透视投影相机对象可以在场景中移动漫游。

如果你想实现一个场景漫游的效果，直接把透视投影相机对象`camera`作为`OrbitControls`构造函数的参数就可以。

```javascript
var controls = new THREE.OrbitControls(camera);
```

#### OrbitControls的变化事件`change`

对于一个静态的场景，可能不需要一直周期性调用渲染函数渲染场景，而是鼠标旋转缩放场景的时候才重新渲染，就可以通过相机空间OrbitControls的变化事件`change`监听触发函数调用渲染函数render。

```javascript
// 渲染函数
function render() {
  renderer.render(scene, camera);
}

var controls = new THREE.OrbitControls(camera);
//监听鼠标事件，触发渲染函数，更新canvas画布渲染效果
controls.addEventListener('change', render);
```

#### 相机空间作用的窗口范围

通过OrbitControls创建一个相机控件对象的时候，默认情况下，在浏览器的窗口整个内容区域body发生鼠标事件都会旋转、平移或缩放三维场景。在实际应用中如果你需要控制OrbitControls的作用范围，你需要通过`OrbitControls`构造函数的参数2设置。

Threejs渲染结果所在的Canvas画布，通过通过渲染器的`renderer.domElement`属性可以访问。

```html
<div id="webgl" style="position: absolute;left: 30px;top: 100px;">
<!-- 插入的canvas画布尺寸设置为800pxX600px -->
</div>
```

```javascript
// 在id为webgl的元素中插入canvas画布
document.getElementById('webgl').appendChild(renderer.domElement);
//设置canvas画布renderer.domElement渲染区域尺寸
renderer.setSize(800, 600);
...
var controls = new THREE.OrbitControls(camera, renderer.domElement);
```

#### 相机查看目标

执行`THREE.OrbitControls`构造函数时候，默认设置.target属性的值是`Vector3(0,0,0)`,如果在执行`new THREE.OrbitControls`之前设置了`camera.lookAt(特定位置);`，相当于再次设置`camera.lookAt(new THREE.Vector3(0,0,0));`。

