---
title: Cesium Wall + Canvas 实现文字贴地贴墙动态标注
date: 2026-07-27 16:00:00
tags:
  - Cesium
  - Canvas
  - WebGL
  - 三维GIS
categories:
  - 技术分享
keywords: "Cesium, Canvas, Wall实体, 贴地标注, 贴墙标注, CallbackProperty, ImageMaterialProperty, turf.js, 三维地球, 动态材质"
description: "Cesium原生Label实体存在始终面向相机、像素级尺寸、样式能力有限等局限。本文介绍通过Wall实体+Canvas动态材质实现文字贴地/贴墙动态标注的完整技术方案，涵盖异步纹理生成、CallbackProperty逐帧刷新、相机自动定位等核心要点。"
---

![贴墙贴地动态标注效果演示](/img/cesium-wall-demo.gif)

## Cesium原生Label的局限

Cesium 原生提供了 `Label` 实体用于文字标注，调用简单几行代码就能把文字放到地球上，但它有三个明显的局限：

1. **始终面向相机**：Label 是 Billboard 式的，无论相机怎么转，它永远正对着你。文字无法以"墙面"的姿态立在地表上，做不到建筑铭牌、纪念碑碑文那种空间融入感。
2. **像素级尺寸**：Label 的大小单位是屏幕像素，不会随相机拉近拉远而自然缩放。远看糊成一团，近看又太大，和三维场景的尺度感格格不入。
3. **样式能力有限**：背景只是一块纯色矩形，没有渐变、边框装饰、多段文字混排、富文本等高阶排版能力。

这些局限的根源在于 Label 是为"信息标注"而非"空间内容"设计的。当需求变成"在地表立一块固定的广告牌"或"让文字像铭牌一样贴在地面上"时，需要换一个思路——**Wall 实体 + Canvas 动态材质**。

## 技术方案概述

Cesium 的 Wall 实体可以在两点之间建立一面竖直的"墙"，而 `ImageMaterialProperty` 可以接受 `HTMLCanvasElement` 作为纹理。两者结合，就能把任意 Canvas 绘制的内容（文字、图表、装饰边框、实时时间戳）作为墙面材质贴在三维地球上。

核心技术要点：
- **Canvas 异步绘制**：将图片加载封装为 Promise，确保绘制顺序正确
- **Wall 材质绑定**：Wall 实体 + Rectangle 实体共享同一份材质
- **CallbackProperty 动态刷新**：每帧重绘 Canvas，实现实时更新
- **相机自动定位**：借助 turf.js 计算中垂线目标点

## 环境搭建

项目采用纯前端方案，依赖通过 CDN 加载：

- Cesium 1.140：三维地球渲染引擎
- Vue 3.5：应用框架（仅用于组件化组织，核心逻辑与框架无关）
- turf.js 7.2：地理空间计算（方位角、中点、目标点）

```html
<script src="https://cdn.jsdelivr.net/npm/cesium@1.140.0/Build/Cesium/Cesium.js"></script>
<script src="https://cdn.jsdelivr.net/npm/vue@3.5.13/dist/vue.global.prod.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@turf/turf@7.2.0/turf.min.js"></script>
```

底图使用 ArcGIS 卫星影像服务，无需 API Key。

## 一、Canvas 文字材质的异步绘制

Cesium 的 `ImageMaterialProperty` 接受 `HTMLCanvasElement` 作为 `image` 参数，这使得在 Canvas 上动态绘制文字并用作三维材质成为可能。但直接使用回调式图片加载存在时序问题。

### 异步加载与 Promise 封装

常规的 `Image.onload` 回调模式会在图片尚未加载完成时提前返回 Canvas 对象，导致 Cesium 获取到空白材质。解决方案是将图片加载封装为 Promise，配合 `async/await` 保证绘制顺序：

```javascript
async function drawCanvasImage(text, options = {}) {
  const canvas = document.createElement("canvas");
  const ctx = canvas.getContext("2d");

  // 先等待背景图加载完成
  if (options.backgroundImage) {
    const img = await loadImage(options.backgroundImage);
    ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
  }

  // 再绘制文字层
  drawTextOnCanvas(ctx, text, options);
  return canvas;
}

function loadImage(src) {
  return new Promise((resolve, reject) => {
    const img = new Image();
    img.onload = () => resolve(img);
    img.onerror = reject;
    img.src = src;
  });
}
```

`loadImage` 将回调模式转为 Promise，`drawCanvasImage` 通过 `await` 确保背景图就位后再执行文字绘制，避免了竞态条件。

### 文字排版函数

文本采用 `;` 分隔的多段格式——首段为标题（右上角小号字体），其余为正文（左对齐加粗字体）：

```javascript
function drawTextOnCanvas(ctx, text, { width, height, fontSize, fontColor, align }) {
  const lines = text.split(";");
  const marginTop = height / 4;
  const marginLeft = width / 10;

  // 标题段 — 右上角对齐
  const title = lines.shift();
  ctx.font = `${fontSize / 1.5}px Arial`;
  const titleX = width - ctx.measureText(title).width - marginLeft;
  ctx.fillText(title, titleX, marginTop - fontSize * 0.565);

  // 正文 — 按对齐方式逐行绘制
  ctx.font = `600 ${fontSize}px PingFang SC-Regular, Microsoft YaHei, sans-serif`;
  ctx.fillStyle = fontColor;
  lines.forEach((line, idx) => {
    const x = calcTextX(ctx, line, width, marginLeft, align);
    ctx.fillText(line, x, marginTop + fontSize * 1.5 * (idx + 0.5));
  });
}

function calcTextX(ctx, text, width, marginLeft, align) {
  if (align === "center") return (width - ctx.measureText(text).width) / 2;
  if (align === "right") return width - ctx.measureText(text).width - marginLeft;
  return marginLeft;
}
```

> Canvas 的 `measureText` 在中文字体下的精度有限，但 Wall 材质通常处于远距离观察状态，误差在视觉上可忽略。

## 二、Wall 实体与 Canvas 材质的绑定

Cesium 的 Wall 实体通过四个参数定义：两端点的经纬度（`positions`）、墙顶高度（`maximumHeights`）、墙底高度（`minimumHeights`）、墙面材质（`material`）。

```javascript
class CreateLabelInPolygon {
  constructor(viewer, params = {}) {
    this.viewer = viewer;
    this.p = params;
    this.entity = null;
    this._groundEntity = null;  // 贴地矩形
    this._baseCanvas = null;    // 缓存底图，避免每帧重绘
  }

  async add() {
    // 一次性生成底图 Canvas（包含背景图 + 文字排版）
    this._baseCanvas = await drawCanvasImage(this.p.text, {
      fontSize: this.p.fontSize || 100,
      fontColor: this.p.fontColor,
      align: this.p.align,
      backgroundImage: this.p.backgroundImage,
    });

    const w = this._baseCanvas.width;
    const h = this._baseCanvas.height;

    // 共享材质：贴墙（Wall）和贴地（Rectangle）使用同一个 CallbackProperty
    const sharedMaterial = new Cesium.ImageMaterialProperty({
      image: new Cesium.CallbackProperty(() => {
        const canvas = document.createElement("canvas");
        canvas.width = w;
        canvas.height = h;
        const ctx = canvas.getContext("2d");
        ctx.drawImage(this._baseCanvas, 0, 0);   // 复制底图
        drawTimestamp(ctx, w, h);                  // 叠加实时时间
        return canvas;
      }, true),   // isConstant=false → 每帧重新求值
      color: Cesium.Color.fromCssColorString("rgba(255,255,255,0.9999)"),
    });

    // ① 贴墙：Wall 实体
    this.entity = this.viewer.entities.add({
      wall: {
        positions: Cesium.Cartesian3.fromDegreesArray(this.p.coordinates),
        maximumHeights: [this.p.maxHeight || 50, this.p.maxHeight || 50],
        minimumHeights: [this.p.minHeight || 25, this.p.minHeight || 25],
        material: sharedMaterial,
      },
    });

    // ② 文字贴地：在地表铺一个与墙面等比例的矩形
    const [lon1, lat1, lon2, lat2] = this.p.coordinates;
    const lonSpanM =
      Math.abs(lon2 - lon1) *
      111320 *
      Math.cos(Cesium.Math.toRadians(Math.min(lat1, lat2)));
    const latSpanM = lonSpanM * (h / w);
    const latSpanDeg = latSpanM / 111320;
    const north = Math.max(lat1, lat2);
    const south = north - latSpanDeg * 2;

    this._groundEntity = this.viewer.entities.add({
      rectangle: {
        coordinates: Cesium.Rectangle.fromDegrees(
          Math.min(lon1, lon2), south,
          Math.max(lon1, lon2), north
        ),
        material: sharedMaterial,
      },
    });

    return this;
  }
}
```

### 设计要点

- `add()` 声明为 `async`，确保 Canvas 绘制完成后再创建实体
- `sharedMaterial` 被 Wall 和 Rectangle 两个实体共用，同一个 `CallbackProperty` 实例分别为每个实体独立求值
- `ImageMaterialProperty` 的 `image` 参数直接接收 `HTMLCanvasElement` 对象，而非 URL 字符串
- `color` 参数设为 `rgba(255,255,255,0.9999)`（白色、透明度 0.9999），让 Canvas 自身的颜色和文字内容透出
- `_baseCanvas` 缓存一次性绘制的高成本内容，每帧只需 `drawImage` 复制底图 + `fillText` 一行时间字符串

### 贴地 Rectangle 坐标计算

- 经度跨度（米）= 坐标差 × `111320 × cos(lat)`
- 按 Canvas 宽高比反算纬度跨度，保证纹理不变形
- `north` 取 `coordinates` 中最大纬度，矩形从该线向南延伸，紧贴在 Wall 脚下

## 三、CallbackProperty 动态刷新与实时时间戳

在墙面右下角叠加一个**每秒刷新的实时时间戳**，依赖 Cesium 的 `CallbackProperty` 机制：

```javascript
function drawTimestamp(ctx, width, height) {
  const now = new Date();
  const timeStr = now.toLocaleString("zh-CN", {
    year: "numeric", month: "2-digit", day: "2-digit",
    hour: "2-digit", minute: "2-digit", second: "2-digit",
    hour12: false,
  });
  ctx.save();
  ctx.font = "32px PingFang SC-Regular, Microsoft YaHei, monospace";
  ctx.fillStyle = "rgba(102, 217, 255, 0.85)";
  ctx.textAlign = "right";
  ctx.textBaseline = "bottom";
  ctx.fillText(timeStr, width - 60, height - 40);
  ctx.restore();
}
```

`CallbackProperty` 的第二个参数 `true` 表示 `isConstant=false`，Cesium 会在**每一帧渲染前**重新调用回调函数获取最新的 Canvas 对象。回调内部的工作流程：

1. 创建一个新的临时 Canvas（与底图同尺寸）
2. 用 `drawImage` 将缓存好的 `_baseCanvas` 完整复制到新 Canvas
3. 调用 `drawTimestamp` 叠加格式化后的时间字符串
4. 返回新 Canvas，Cesium 将其作为 Wall 的当前帧纹理

> 这种"缓存底图 + 每帧叠加动态层"的模式，在保证动态效果的同时将每帧的绘制成本控制在极低水平。同样适用于滚动词幕、实时数据面板等场景。

## 四、相机自动定位到最佳观察角度

Wall 创建后，相机可能处于地球任意位置。使用 turf.js 计算中垂线目标点，让相机移动到墙面的中垂线上：

```javascript
function getDestination(point1, point2, distance, bearing, options = {}) {
  const bearing0 = turf.bearing(point1, point2);        // ① 墙的方向角
  const center = turf.midpoint(point1, point2);          // ② 墙的中点
  const dest = turf.destination(
    center, distance, bearing0 + (bearing || 90)          // ③ 偏移 90° → 中垂线
  );
  return dest.geometry.coordinates;
}
```

通过 `camera.flyTo` 实现平滑过渡：

```javascript
locate(cameraInfo) {
  if (cameraInfo?.position) {
    this.viewer.camera.flyTo({
      destination: cameraInfo.position,
      orientation: {
        heading: Cesium.Math.toRadians(cameraInfo.heading ?? 0),
        pitch: Cesium.Math.toRadians(cameraInfo.pitch ?? -10),
        roll: cameraInfo.roll ?? 0,
      },
      duration: 1,
    });
  }
}
```

调用示例：

```javascript
const dest = getDestination(p1, p2, 40000, 100);
const bearing = turf.bearing(dest, [(p1[0] + p2[0]) / 2, (p1[1] + p2[1]) / 2]);
const cameraInfo = {
  position: Cesium.Cartesian3.fromDegrees(dest[0], dest[1], 10000),
  heading: bearing,
};
labelPolygon.locate(cameraInfo);
```

![贴墙+贴地L形展示效果](/img/cesium-wall-result.png)

## 完整使用示例

```javascript
const labelPolygon = await addLabelInPolygon(viewer, {
  coordinates: [120, 33, 120.2, 33],
  backgroundImage: plaqueBg,
  fontSize: 50,
  text: "出师表;先帝创业未半而中道崩殂。;今天下三分益州疲弊,此诚危急存亡之秋也。;然侍卫之臣不懈于内,忠志之士忘身于外者,盖追先帝之殊遇，;欲报之于陛下也。诚宜开张圣听，以光先帝遗德，;恢弘志士之气，不宜妄自菲薄，引喻失义，以塞忠谏之路也。",
  fontColor: "rgba(102, 217, 255, 1)",
  minHeight: 0.1,
  maxHeight: 0.1 + (20000 * 1080) / 1920,
  align: "left",
});
```

## 总结

本文分析了基于 Cesium Wall + Canvas 动态材质实现文字贴墙/贴地动态标注的完整链路，核心技术要点：

1. **异步纹理生成**：将图片加载转为 Promise，通过 `async/await` 确保 Canvas 绘制与 Cesium 材质绑定的时序正确性
2. **动态材质接口**：利用 `ImageMaterialProperty` 对 `HTMLCanvasElement` 的原生支持，实现程序化纹理生成
3. **CallbackProperty 逐帧刷新**：通过 `CallbackProperty(isConstant=false)` 实现每帧重绘 Canvas——缓存高成本的底图，仅叠加轻量的动态层
4. **贴墙 + 贴地双实体**：Wall 负责贴墙、Rectangle 负责贴地，共享同一份 `CallbackProperty` 材质，形成 L 形展示效果
5. **地理空间定位**：借助 turf.js 的中垂线计算能力，实现相机自动飞到最佳观察角度

### 扩展方向

- **多点阵列**：沿地理路径排列多个 Wall，构建连续的"信息廊道"效果
- **地形适配**：结合 Cesium 地形数据，动态调整 `minimumHeight` 以贴合地表起伏
- **数据驱动**：将 `CallbackProperty` 的回调接入外部数据源（WebSocket / 轮询），在墙面上实时渲染传感器读数、股票行情等动态数据面板
