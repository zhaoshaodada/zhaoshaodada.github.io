---
title: Cesium基础绘制（点线面）
date: 2026-07-23 10:00:00
tags:
  - Cesium
  - 点线面
  - GIS
categories:
  - 教程
keywords: "Cesium, 点线面, 基础绘制, Entity, 地图开发"
description: "本文讲解在Cesium三维地图场景中如何进行点、线、面的基础绘制实现，属于GIS地图开发的核心功能。文章基于Entity API介绍简单绘制的实现思路，对比二维与三维场景的差异，帮助开发者快速上手Cesium的图形绘制与地图可视化开发。"
---

> 转载自：[CreateFree的博客](https://www.cnblogs.com/CreateFree/p/11488492.html)

## 前言

对于一个地图GIS场景，绘制点、线、面属于是基础功能，无论是二维地图还是三维地图场景均是如此，尤其对于三维场景来说比二维应该是更加困难了些。

但是基础的简单绘制不用考虑太多，下面我们开始学习在Cesium的三维场景中如何进行基础绘制的实现。

## 使用原始Cesium的Entity方法绘制

Cesium中封装了几何对象的接口，也就是点、线、面、圆柱体、长方体、圆锥体等等，还有特殊的几何对象：corridor、ellipse、ellipsoid；以及billboard和model。但这次主要是使用点、线、面这三个几何对象，其他的几何对象都是类似的，使用方法大同小异，主要是看每个几何对象自身内部所需要的参数有哪些罢了。

### 绘制点Entity

首先看看[PointGraphics](https://cesiumjs.org/Cesium/Build/Documentation/PointGraphics.html)点几何对象，它需要的参数是点的位置（坐标、最主要的），样式（颜色、轮廓的宽度、颜色等）。那么我们传给它这些属性即可实现绘制点的功能。

核心代码如下：

1. 使用handler构建鼠标事件：

```javascript
document.getElementById("drawpoint").addEventListener("click", function () {
  var handler = new Cesium.ScreenSpaceEventHandler(viewer.scene.canvas);
  handler.setInputAction(function (movement) {
    tooltip.style.left = movement.endPosition.x + 10 + "px";
    tooltip.style.top = movement.endPosition.y + 20 + "px";
  }, Cesium.ScreenSpaceEventType.MOUSE_MOVE);
  handler.setInputAction(function (movement) {
    position = viewer.camera.pickEllipsoid(movement.position, viewer.scene.globe.ellipsoid);
    let point = drawPoint(position);
    tempEntities.push(point);
  }, Cesium.ScreenSpaceEventType.LEFT_CLICK);
  handler.setInputAction(function () {
    handler.destroy();
    handler = null;
  }, Cesium.ScreenSpaceEventType.LEFT_DOUBLE_CLICK);
  handler.setInputAction(function () {
    handler.destroy();
    handler = null;
  }, Cesium.ScreenSpaceEventType.RIGHT_CLICK);
});
```

2. 使用entity添加点：

```javascript
function drawPoint(position, config) {
  var config = config ? config : {};
  var pointGeometry = viewer.entities.add({
    name: "点几何对象",
    position: position,
    point: {
      color: Cesium.Color.SKYBLUE,
      pixelSize: 10,
      outlineColor: Cesium.Color.YELLOW,
      outlineWidth: 3,
      disableDepthTestDistance: Number.POSITIVE_INFINITY
    }
  });
  return pointGeometry;
};
```

### 绘制线Entity

可以看看[PolylineGraphics](https://cesiumjs.org/Cesium/Build/Documentation/PolylineGraphics.html)对象中的属性，两点连成线，positions属性就是用来存放各个拐点的；还有线的宽度width，样式等和点类似。

1. 使用handler构建鼠标事件：

```javascript
document.getElementById("drawline").addEventListener("click", function () {
  handler = new Cesium.ScreenSpaceEventHandler(viewer.scene.canvas);
  var tempPoints = [];
  handler.setInputAction(function (movement) {
    tooltip.style.left = movement.endPosition.x + 10 + "px";
    tooltip.style.top = movement.endPosition.y + 20 + "px";
  }, Cesium.ScreenSpaceEventType.MOUSE_MOVE);
  handler.setInputAction(function (click) {
    position = viewer.camera.pickEllipsoid(click.position, viewer.scene.globe.ellipsoid);
    tempPoints.push(position);
    let tempLength = tempPoints.length;
    let point = drawPoint(tempPoints[tempPoints.length - 1]);
    tempEntities.push(point);
    if (tempLength > 1) {
      let pointline = drawPolyline([tempPoints[tempPoints.length - 2], tempPoints[tempPoints.length - 1]]);
      tempEntities.push(pointline);
    } else {
      tooltip.innerHTML = "请绘制下一个点，右键结束";
    }
    return;
  }, Cesium.ScreenSpaceEventType.LEFT_CLICK);
  handler.setInputAction(function (click) {
    tooltip.style.display = "none";
    tooltip.innerHTML = "左键单击绘制,右键结束绘制";
    tempPoints = [];
    handler.destroy();
    handler = null;
  }, Cesium.ScreenSpaceEventType.RIGHT_CLICK);
});
```

2. 使用entity添加线：

```javascript
function drawPolyline(positions, config) {
  if (positions.length < 1) return;
  var config = config ? config : {};
  var polylineGeometry = viewer.entities.add({
    name: "线几何对象",
    polyline: {
      positions: positions,
      width: config.width ? config.width : 5.0,
      material: new Cesium.PolylineGlowMaterialProperty({
        color: config.color ? new Cesium.Color.fromCssColorString(config.color) : Cesium.Color.GOLD,
      }),
      depthFailMaterial: new Cesium.PolylineGlowMaterialProperty({
        color: config.color ? new Cesium.Color.fromCssColorString(config.color) : Cesium.Color.GOLD,
      }),
    }
  });
  return polylineGeometry;
};
```

### 绘制面Entity

面几何对象，我们使用[PolygonGraphics](https://cesiumjs.org/Cesium/Build/Documentation/PolygonGraphics.html)来绘制。绘制一个面，我们会绘制多个点、多条线组合成。它的属性也是类似的，其中positions属性同线对象是一样的。

1. 使用handler构建鼠标事件：

```javascript
document.getElementById("drawploy").addEventListener("click", function () {
  var tempPoints = [];
  handler = new Cesium.ScreenSpaceEventHandler(viewer.scene.canvas);
  tooltip.style.display = "block";
  handler.setInputAction(function (movement) {
    tooltip.style.left = movement.endPosition.x + 10 + "px";
    tooltip.style.top = movement.endPosition.y + 20 + "px";
  }, Cesium.ScreenSpaceEventType.MOUSE_MOVE);
  handler.setInputAction(function (click) {
    position = viewer.camera.pickEllipsoid(click.position, viewer.scene.globe.ellipsoid);
    tempPoints.push(position);
    let tempLength = tempPoints.length;
    let point = drawPoint(position);
    tempEntities.push(point);
    if (tempLength > 1) {
      let pointline = drawPolyline([tempPoints[tempPoints.length - 2], tempPoints[tempPoints.length - 1]]);
      tempEntities.push(pointline);
    } else {
      tooltip.innerHTML = "请绘制下一个点，右键结束";
    }
    return;
  }, Cesium.ScreenSpaceEventType.LEFT_CLICK);
  handler.setInputAction(function (click) {
    let cartesian = viewer.camera.pickEllipsoid(click.position, viewer.scene.globe.ellipsoid);
    if (cartesian) {
      let tempLength = tempPoints.length;
      if (tempLength < 3) {
        alert('请选择3个以上的点再执行闭合操作命令');
      } else {
        let pointline = drawPolyline([tempPoints[tempPoints.length - 1], tempPoints[0]]);
        tempEntities.push(pointline);
        drawPolygon(tempPoints);
        tempEntities.push(tempPoints);
        handler.destroy();
        handler = null;
      }
    }
  }, Cesium.ScreenSpaceEventType.RIGHT_CLICK);
});
```

2. 使用entity添加面：

```javascript
function drawPolygon(positions, config) {
  if (positions.length < 2) return;
  var config = config ? config : {};
  var polygonGeometry = viewer.entities.add({
    name: "线几何对象",
    polygon: {
      height: 0.1,
      hierarchy: new Cesium.PolygonHierarchy(positions),
      material: config.color ? new Cesium.Color.fromCssColorString(config.color).withAlpha(.2) : new Cesium.Color.fromCssColorString("#FFD700").withAlpha(.2),
      perPositionHeight: true,
    }
  });
  return polygonGeometry;
};
```

## 总结

使用自己制作的基础绘制工具，其优点是方便、快捷，缺点是简陋，但好方法、好技术都是慢慢积累出来的，这里我只是使用了Cesium的Entity中的点、线、面绘制，Entity中还有很多的几何对象绘制，可以自行查看API学习，如果之后有时间会考虑再补充。

插件是好东西，对于我们使用来说极其方便，用起来也是好用，但是对于我们理解Cesium来说就少了些内容，如果有时间应当自己去模仿一下插件的实现，或者阅读其源码，看看它是如何实现的。
