---
title: 提升Cesium渲染性能的方法
date: 2026-07-27 10:30:00
tags:
 - Cesium
 - GIS
 - 性能优化
 - 三维地图
categories:
 - 教程
keywords: "Cesium, 性能优化, FPS, maximumScreenSpaceError, terrainTileCacheSize, imageryTileCacheSize, 瓦片缓存, 三维地球"
description: "本文详细介绍如何通过调整Cesium的地形精度、瓦片缓存等参数来提升渲染性能和FPS，让三维地球场景运行更加流畅，适用于Web端GIS应用开发。"
---

## 提升Cesium渲染性能的方法

在使用Cesium开发Web三维GIS应用时，性能优化是一个绕不开的话题。当地球视角拉近、地形细节增多或者叠加多层影像数据时，FPS（每秒帧数）往往会明显下降，甚至会出现卡顿。本文就来分享几个通过调整核心参数来提升Cesium渲染性能的实用方法。

### 问题背景

Cesium默认的地形和影像加载配置，通常是为了保证最佳的视觉效果。但在实际项目中，尤其是面向普通PC或笔记本用户时，过高的精度设置会导致：

- GPU渲染压力过大，FPS下降
- 内存占用持续攀升，浏览器卡顿甚至崩溃
- 瓦片加载队列拥堵，画面迟迟无法完整显示

这时候就需要在**视觉效果**和**运行性能**之间做出权衡。

### 核心优化参数

#### 1. 限制地形加载精度 —— maximumScreenSpaceError

```javascript
// 初始化时配置
const viewer = new Cesium.Viewer('cesiumContainer', {
  terrainProvider: Cesium.createWorldTerrain({
    requestWaterMask: true,
    requestVertexNormals: true
  }),
  // 地形渲染精度
  globe: {
    maximumScreenSpaceError: 16
  }
})

// 或者初始化后动态修改
viewer.scene.globe.maximumScreenSpaceError = 16
```

**原理说明：**

`maximumScreenSpaceError` 决定了地形瓦片的细分程度。它的含义是：**屏幕空间误差阈值**，数值越小，地形越精细，加载的瓦片层级越高；数值越大，地形越粗糙，但性能越好。

- **默认值：2** —— 精度很高，适合高端设备
- **推荐值：8 ~ 16** —— 在视觉效果和性能之间取得平衡
- **低端设备：32** —— 优先保证流畅度

这个参数对FPS的影响非常显著。在视角拉近、地形起伏较大的区域，将它从默认值2调整到16，通常能让FPS提升30%~50%。

#### 2. 控制地形瓦片缓存 —— tileCacheSize

```javascript
// 初始化时
const viewer = new Cesium.Viewer('cesiumContainer', {
  globe: {
    tileCacheSize: 50
  }
})

// 动态修改
viewer.scene.globe.tileCacheSize = 50
```

**原理说明：**

`tileCacheSize` 控制Cesium在内存中保留的地形瓦片数量。Cesium为了提升浏览体验，会把已经加载过的瓦片缓存起来，避免用户转动视角时重复请求。但如果缓存过大，内存占用会急剧增加。

- **默认值：100** —— 缓存较多瓦片，内存占用大
- **推荐值：50** —— 适当减少缓存，释放内存
- **内存紧张：20** —— 最小化缓存，以网络请求换取内存

当用户设备内存有限（比如4GB或8GB内存的笔记本），或者同时运行多个Web应用时，适当降低这个值能有效减少浏览器的内存压力。

#### 3. 控制影像瓦片缓存 —— imageryCacheSize

```javascript
const viewer = new Cesium.Viewer('cesiumContainer', {
  imageryProvider: new Cesium.TileMapServiceImageryProvider({
    url: 'https://your-tile-server.com/tiles'
  }),
  // 影像图层缓存
  imageryCacheSize: 50
})
```

**原理说明：**

和地形瓦片类似，影像瓦片也会被缓存。当叠加了多层影像（如底图+标注+热力图+实景）时，影像瓦片的缓存占用会迅速累积。限制影像缓存大小可以避免内存溢出。

### 综合配置示例

将以上参数整合在一起，一个兼顾性能和效果的Cesium初始化配置如下：

```javascript
const viewer = new Cesium.Viewer('cesiumContainer', {
  // 地形数据源
  terrainProvider: Cesium.createWorldTerrain({
    requestWaterMask: false,        // 不需要水面效果时关闭
    requestVertexNormals: false     // 不需要地形光照时关闭
  }),

  // 场景优化
  scene3DOnly: true,               // 如果不需要2D/哥伦布视图，关闭以提升性能
  orderIndependentTranslucency: false,

  // 地球参数
  globe: {
    maximumScreenSpaceError: 16,   // 降低地形精度
    tileCacheSize: 50              // 限制地形缓存
  },

  // 影像缓存
  imageryCacheSize: 50,

  // 关闭不必要的UI组件
  baseLayerPicker: false,
  geocoder: false,
  homeButton: false,
  sceneModePicker: false,
  timeline: false,
  animation: false,
  fullscreenButton: false
})
```

### 更多性能优化建议

除了调整上述核心参数，还有以下几个常用的优化手段：

#### 关闭抗锯齿

```javascript
viewer.scene.postProcessStages.fxaa.enabled = false
```

抗锯齿（FXAA）会消耗一定的GPU性能，如果对画面边缘平滑度要求不高，可以关闭。

#### 限制请求并发数

```javascript
viewer.scene.globe.maximumSimultaneousTileLoads = 10
```

控制同时加载的瓦片数量，避免过多并发请求导致网络拥堵和渲染阻塞。

#### 根据视角距离动态调整精度

```javascript
viewer.scene.globe.maximumScreenSpaceError = 16

// 当相机高度较高时，进一步降低精度
viewer.camera.changed.addEventListener(() => {
  const height = viewer.camera.positionCartographic.height
  if (height > 100000) {
    viewer.scene.globe.maximumScreenSpaceError = 32
  } else {
    viewer.scene.globe.maximumScreenSpaceError = 16
  }
})
```

远处俯瞰时，用户对地形细节不敏感，可以大幅降低精度；拉近时再恢复。

#### 使用Primitive代替Entity

如果场景中需要大量动态对象（如标记点、模型），优先使用 `Primitive` 而不是 `Entity`。Primitive更底层、更轻量，渲染性能更好。

```javascript
// 性能更好的方式
const primitive = new Cesium.Primitive({
  geometryInstances: new Cesium.GeometryInstance({
    geometry: new Cesium.PointGeometry({
      positions: positions
    })
  }),
  appearance: new Cesium.PointAppearance()
})
viewer.scene.primitives.add(primitive)
```

### 总结

Cesium性能优化的核心思路是：**在视觉可接受的范围内，尽可能减少GPU和内存的压力**。

| 参数 | 默认值 | 推荐值 | 作用 |
|------|--------|--------|------|
| `maximumScreenSpaceError` | 2 | 16 | 降低地形精度，减少瓦片数量 |
| `tileCacheSize` | 100 | 50 | 限制地形瓦片缓存，释放内存 |
| `imageryCacheSize` | 100 | 50 | 限制影像瓦片缓存 |
| `maximumSimultaneousTileLoads` | 无限制 | 10 | 控制并发加载数 |

不同项目、不同硬件环境下，最佳的参数组合也不同。建议在实际项目中通过浏览器的Performance面板观察FPS和内存占用，逐步调整找到最适合自己场景的配置。
