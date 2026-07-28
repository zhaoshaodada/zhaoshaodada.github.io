---
title: Cesium 近地天空盒：把天空拉回地面
date: 2026-07-27 18:00:00
tags:
  - Cesium
  - 天空盒
  - WebGL
  - 三维GIS
categories:
  - 技术分享
keywords: "Cesium, 天空盒, SkyBoxOnGround, DrawCommand, Cubemap, 近地天空盒, 全景图, 三维地球, 自定义渲染"
description: "Cesium默认天空盒适合太空视角，相机贴近地面时天空显得不贴合。本文介绍通过自定义DrawCommand做近地天空盒，并按相机高度自动切回默认天空的完整技术方案，涵盖SkyBoxOnGround实现、Cubemap方向映射、天空盒制作三步走等核心要点。"
---

![Cesium 近地天空盒](/img/cesium-skybox-cover.jpg)

## 为什么要单独做近地天空盒

Cesium 默认天空在视觉上有一个挺别扭的问题：相机落到几千米甚至更低时，Cesium 默认天空还是那个"站在太空外看地球"的味道，虽然 Cesium 做了大气和天空的优化，但近景看天空还是过于单一。

尤其是做街景感、城市漫游、低空飞行、海边全景这类场景时，天空不应该只是一个遥远的宇宙背景。它更像一个包住相机的环境球。这就是近地天空盒要解决的事儿：相机低的时候用一套贴近地面的 Cubemap，高的时候恢复 Cesium 原生天空和大气。

## 核心思路：不是换背景图，而是换 Scene 的 skyBox

Cesium 的天空来自 `viewer.scene.skyBox`。默认情况下，它配合 `skyAtmosphere` 给出地球外空间的大气效果。

近地天空盒要做两件事：

1. 自己构造一个能被 Cesium 渲染管线接受的对象
2. 在低空时替换 `scene.skyBox`，离开低空后再换回默认对象

Demo 里拆成两个类：

- `SkyBoxOnGround`：真正负责创建近地天空盒的渲染命令
- `SkyBox`：一个薄封装，负责把高度判断和切换逻辑挂到 Viewer 上

关键点：`SkyBoxOnGround` 不是普通 Entity，也不是 ImageryLayer。它实现了一个 `update(frameState, useHdr)` 方法，Cesium 每帧渲染时会调用它，并期待它返回一个 `DrawCommand`。

**说白了：我们把自己塞进了 Cesium 的渲染队列里。**

## SkyBoxOnGround：复制 Cesium SkyBox，再改一点点

代码开头把 Cesium 内部对象都取出来：`BoxGeometry`、`CubeMap`、`DrawCommand`、`ShaderProgram`、`VertexArray`、`RenderState` 等。看起来有点硬，但这类近地天空盒本来就不是高层 API 能覆盖的功能。

`SkyBoxOnGround` 的构造函数主要做三件事：

- 保存六面贴图 `sources`
- 创建一个 `DrawCommand`
- 缓存 `_cubeMap`、`_attributeLocations`、`_useHdr`

还有一段兼容逻辑：

```javascript
if (!Cesium.defined(Cesium.Matrix4.getRotation)) {
  Cesium.Matrix4.getRotation = Cesium.Matrix4.getMatrix3;
}
```

这是为了兼容较新版本的 Cesium。以前能直接用的 `Matrix4.getRotation` 在新版本里可能不存在，所以这里补到 `getMatrix3`。这事儿很小，但不补就会在运行时炸。

## 渲染命令：一个盒子、一个 Cubemap、两段 Shader

近地天空盒本质上是一个始终围绕相机的盒子。代码里用 `BoxGeometry.fromDimensions` 创建了一个 2×2×2 的立方体：

```javascript
const geometry = BoxGeometry.createGeometry(
  BoxGeometry.fromDimensions({
    dimensions: new Cartesian3(2.0, 2.0, 2.0),
    vertexFormat: VertexFormat.POSITION_ONLY,
  })
);
```

然后用 `VertexArray.fromGeometry` 交给 GPU。片元着色器负责从 `samplerCube` 里采样：

```glsl
vec4 color = texture(u_cubeMap, normalize(v_texCoord));
fragColor = vec4(czm_gammaCorrect(color).rgb, czm_morphTime);
```

顶点着色器才是近地天空盒的重点：

```glsl
vec3 p = czm_viewRotation * u_rotateMatrix *
  (czm_temeToPseudoFixed * (czm_entireFrustum.y * position));
```

它把盒子放到当前相机视锥远端，并乘上 `u_rotateMatrix`。这个矩阵来自：

```javascript
Transforms.eastNorthUpToFixedFrame(frameState.camera._positionWC)
```

也就是说，天空盒不是固定在世界坐标里，而是按相机所在位置的东北天坐标系重新对齐。相机走到哪，天空盒就跟到哪。这才有"贴地环境"的感觉。

![近地天空盒效果](/img/cesium-skybox-result.jpg)

## 六面贴图：最容易错的是方向映射

页面里的贴图来源写在 `createSkyBoxSources()`：

```javascript
function createSkyBoxSources() {
  return {
    positiveX: "./skyboxImage/px.png",
    negativeX: "./skyboxImage/nx.png",
    positiveY: "./skyboxImage/pz.png",
    negativeY: "./skyboxImage/nz.png",
    positiveZ: "./skyboxImage/py.png",
    negativeZ: "./skyboxImage/ny.png",
  };
}
```

注意，这里不是机械地把 `px` 放到 `positiveX` 就完事。全景图拆成 Cubemap 后，不同工具导出的坐标约定可能不一样。

| Cesium 字段 | 使用图片 | 说明 |
|-------------|----------|------|
| `positiveX` | `px.png` | +X 面 |
| `negativeX` | `nx.png` | -X 面 |
| `positiveY` | `pz.png` | 用 pz 图接到 +Y 面 |
| `negativeY` | `nz.png` | 用 nz 图接到 -Y 面 |
| `positiveZ` | `py.png` | 用 py 图接到 +Z 面 |
| `negativeZ` | `ny.png` | 用 ny 图接到 -Z 面 |

为什么会这样？因为全景拆图工具常见的"上、下、前、后、左、右"，和 Cesium shader 里这套 Cubemap 采样方向不一定同轴。实际做的时候，建议直接在图上标 1、2、3、4、5、6，再一面一面试。

> 天空盒方向错了不一定立刻看出来，只有地平线、建筑、海面这种连续元素接不上时，才会露馅。

## 高度切换：低空用自定义，高空还给 Cesium

近地天空盒不能一直开。高空看地球时，它会把 Cesium 默认大气和宇宙背景都盖掉，效果反而怪。

```javascript
_this.viewer.scene.postRender.addEventListener(() => {
  const e = _this.viewer.camera.position;
  if (Cesium.Cartographic.fromCartesian(e).height < height) {
    _this.viewer.scene.skyBox = currentSkyBox;
    _this.viewer.scene.skyAtmosphere.show = false;
  } else {
    _this.viewer.scene.skyBox = defaultSkyBox;
    _this.viewer.scene.skyAtmosphere.show = true;
  }
});
```

Demo 里阈值是 `120000` 米。低于这个高度，使用 `SkyBoxOnGround`；高于这个高度，恢复默认 `skyBox` 和 `skyAtmosphere`。

为什么挂在 `postRender`？因为相机高度变化本来就是渲染过程里的状态，每帧检查一次最直接。

## 天空盒制作方法：三步走完

### 第一步：全景图下载

先准备一张完整的 360 全景图，可以从 [polyhaven.com](https://polyhaven.com/zh) 下载。它通常是横向长图，宽高比例大多接近 `2:1`。

![全景图示例](/img/cesium-skybox-panorama.jpg)

注意两点：
- 画面最好有连续地平线（海岸、道路、建筑边缘），匹配六面图时接缝错了能一眼看出来
- 别选透视太乱、主体太近的图，近地天空盒只是背景环境

### 第二步：全景图拆分

全景图不能直接塞给 Cesium 的 `SkyBox`，需要拆成六张 Cubemap 图片，可以在 [HDRI-to-CubeMap](https://matheowis.github.io/HDRI-to-CubeMap/) 上拆分。

![全景图拆分示意](/img/cesium-skybox-split.jpg)

拆出来一般会得到这六类文件：

- `px.png`：positive x
- `nx.png`：negative x
- `py.png`：positive y
- `ny.png`：negative y
- `pz.png`：positive z
- `nz.png`：negative z

问题来了：拆图工具的 px / py / pz，不一定等于 Cesium 最终需要的 positiveX / positiveY / positiveZ。不同工具对"前、后、左、右、上、下"的坐标定义可能不同。

### 第三步：天空盒匹配

最后一步就是把六张图映射到 Cesium 的 `sources`。这份 Demo 里，`positiveY` 接的是 `pz.png`，`positiveZ` 接的是 `py.png`。这不是写错，而是为了让画面在 Cesium 近地视角下连续。

其次，HDRI-to-CubeMap 生成的单个图片方向都是正向的，但 Cesium 中使用是需要旋转的：

- **nx.png 顺时针旋转 90°**
- **px.png 逆时针旋转 90°**
- **nz.png 顺时针旋转 180°**

天空盒匹配过程：

1. 先按拆图工具原始名字接一遍
2. 低空进入场景，看地平线和建筑边缘是否连续
3. 如果上下颠倒，优先交换 `positiveY / negativeY` 或 `positiveZ / negativeZ`
4. 如果左右错位，优先交换 `positiveX / negativeX`
5. 每次只换一组，不要一次改三四个字段

> 真正要记住的不是某个固定表格，而是"按接缝校正映射"这个方法。

## 完整代码

```html
<!doctype html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Cesium 近地天空盒</title>
    <script src="https://cdn.jsdelivr.net/npm/cesium@1.133.0/Build/Cesium/Cesium.js"></script>
    <link href="https://cdn.jsdelivr.net/npm/cesium@1.133.0/Build/Cesium/Widgets/widgets.css" rel="stylesheet" />
    <style>
      html, body, #cesiumContainer {
        width: 100%; height: 100%; margin: 0; padding: 0;
        overflow: hidden; background: #000;
        font-family: "Microsoft YaHei", "PingFang SC", sans-serif;
      }
      .control-panel {
        position: absolute; top: 12px; left: 12px; z-index: 10;
        width: 280px; padding: 14px 16px; color: #e8eef8;
        background: rgba(13, 19, 33, 0.82);
        border: 1px solid rgba(255, 255, 255, 0.12);
        border-radius: 8px; backdrop-filter: blur(10px);
      }
      button {
        height: 32px; color: #f8fbff;
        background: rgba(45, 99, 201, 0.86);
        border: 1px solid rgba(255, 255, 255, 0.16);
        border-radius: 6px; cursor: pointer; font-size: 12px;
      }
      button:hover { background: rgba(58, 119, 232, 0.95); }
    </style>
  </head>
  <body>
    <div id="cesiumContainer"></div>
    <div class="control-panel">
      <div class="control-title">近地天空盒 SkyBoxOnGround</div>
      <p class="control-desc">相机高度低于阈值时切换为自定义近地天空盒，高于阈值时恢复 Cesium 默认天空和大气。</p>
      <div class="status-row">
        <span class="status-label">当前相机高度</span>
        <span id="cameraHeight" class="status-value">-- m</span>
      </div>
      <div class="status-row">
        <span class="status-label">天空盒状态</span>
        <span id="skyboxStatus" class="status-value">--</span>
      </div>
      <div class="status-row">
        <span class="status-label">切换阈值</span>
        <span id="heightLimit" class="status-value">120000 m</span>
      </div>
      <div class="button-row">
        <button id="nearBtn" type="button">低空视角</button>
        <button id="highBtn" type="button">高空视角</button>
      </div>
    </div>

    <script>
      window.CESIUM_BASE_URL = "https://cdn.jsdelivr.net/npm/cesium@1.133.0/Build/Cesium/";

      const BoxGeometry = Cesium.BoxGeometry;
      const Cartesian3 = Cesium.Cartesian3;
      const defaultValue = Cesium.defaultValue;
      const defined = Cesium.defined;
      const destroyObject = Cesium.destroyObject;
      const DeveloperError = Cesium.DeveloperError;
      const GeometryPipeline = Cesium.GeometryPipeline;
      const Matrix3 = Cesium.Matrix3;
      const Matrix4 = Cesium.Matrix4;
      const Transforms = Cesium.Transforms;
      const VertexFormat = Cesium.VertexFormat;
      const BufferUsage = Cesium.BufferUsage;
      const CubeMap = Cesium.CubeMap;
      const DrawCommand = Cesium.DrawCommand;
      const loadCubeMap = Cesium.loadCubeMap;
      const RenderState = Cesium.RenderState;
      const VertexArray = Cesium.VertexArray;
      const BlendingState = Cesium.BlendingState;
      const SceneMode = Cesium.SceneMode;
      const ShaderProgram = Cesium.ShaderProgram;
      const ShaderSource = Cesium.ShaderSource;
      const skyboxMatrix3 = new Matrix3();

      class SkyBoxOnGround {
        constructor(options) {
          if (!Cesium.defined(Cesium.Matrix4.getRotation)) {
            Cesium.Matrix4.getRotation = Cesium.Matrix4.getMatrix3;
          }
          this.sources = options.sources;
          this._sources = undefined;
          this.show = defaultValue(options.show, true);
          this._command = new DrawCommand({
            modelMatrix: Matrix4.clone(Matrix4.IDENTITY),
            owner: this,
          });
          this._cubeMap = undefined;
          this._attributeLocations = undefined;
          this._useHdr = undefined;
        }

        update(frameState, useHdr) {
          const that = this;
          const SkyBoxFS = `#version 300 es
            precision highp float;
            uniform samplerCube u_cubeMap;
            in vec3 v_texCoord;
            out vec4 fragColor;
            void main() {
                vec4 color = texture(u_cubeMap, normalize(v_texCoord));
                fragColor = vec4(czm_gammaCorrect(color).rgb, czm_morphTime);
            }`;

          const SkyBoxVS = `#version 300 es
            precision highp float;
            in vec3 position;
            out vec3 v_texCoord;
            uniform mat3 u_rotateMatrix;
            void main() {
                vec3 p = czm_viewRotation * u_rotateMatrix * (czm_temeToPseudoFixed * (czm_entireFrustum.y * position));
                gl_Position = czm_projection * vec4(p, 1.0);
                v_texCoord = position;
            }`;

          if (!this.show) return undefined;
          if (frameState.mode !== SceneMode.SCENE3D && frameState.mode !== SceneMode.MORPHING) return undefined;
          if (!frameState.passes.render) return undefined;

          const context = frameState.context;

          if (this._sources !== this.sources) {
            this._sources = this.sources;
            const sources = this.sources;
            if (typeof sources.positiveX === "string") {
              loadCubeMap(context, this._sources).then(function (cubeMap) {
                that._cubeMap = that._cubeMap && that._cubeMap.destroy();
                that._cubeMap = cubeMap;
              });
            }
          }

          const command = this._command;
          command.modelMatrix = Transforms.eastNorthUpToFixedFrame(frameState.camera._positionWC);

          if (!defined(command.vertexArray)) {
            command.uniformMap = {
              u_cubeMap: function () { return that._cubeMap; },
              u_rotateMatrix: function () {
                return Matrix4.getRotation(command.modelMatrix, skyboxMatrix3);
              },
            };
            const geometry = BoxGeometry.createGeometry(
              BoxGeometry.fromDimensions({
                dimensions: new Cartesian3(2.0, 2.0, 2.0),
                vertexFormat: VertexFormat.POSITION_ONLY,
              })
            );
            const attributeLocations = (this._attributeLocations =
              GeometryPipeline.createAttributeLocations(geometry));
            command.vertexArray = VertexArray.fromGeometry({
              context: context, geometry: geometry,
              attributeLocations: attributeLocations,
              bufferUsage: BufferUsage.STATIC_DRAW || BufferUsage._DRAW,
            });
            command.renderState = RenderState.fromCache({
              blending: BlendingState.ALPHA_BLEND,
            });
          }

          if (!defined(command.shaderProgram) || this._useHdr !== useHdr) {
            const fs = new ShaderSource({
              defines: [useHdr ? "HDR" : ""],
              sources: [SkyBoxFS],
            });
            command.shaderProgram = ShaderProgram.fromCache({
              context: context,
              vertexShaderSource: SkyBoxVS,
              fragmentShaderSource: fs,
              attributeLocations: this._attributeLocations,
            });
            this._useHdr = useHdr;
          }

          if (!defined(this._cubeMap)) return undefined;
          return command;
        }

        isDestroyed() { return false; }
        destroy() {
          const command = this._command;
          command.vertexArray = command.vertexArray && command.vertexArray.destroy();
          command.shaderProgram = command.shaderProgram && command.shaderProgram.destroy();
          this._cubeMap = this._cubeMap && this._cubeMap.destroy();
          return destroyObject(this);
        }
      }

      class SkyBox {
        constructor(viewer) {
          this.viewer = viewer;
          this.sources = {};
        }

        SkyBoxOnGround(options) {
          const _this = this;
          const height = !options.height ? "5705" : options.height;
          const defaultSkyBox = _this.viewer.scene.skyBox;
          const currentSkyBox = new SkyBoxOnGround({ sources: options.sources });

          _this.viewer.scene.postRender.addEventListener(() => {
            const e = _this.viewer.camera.position;
            if (Cesium.Cartographic.fromCartesian(e).height < height) {
              _this.viewer.scene.skyBox = currentSkyBox;
              _this.viewer.scene.skyAtmosphere.show = false;
            } else {
              _this.viewer.scene.skyBox = defaultSkyBox;
              _this.viewer.scene.skyAtmosphere.show = true;
            }
          });
        }
      }

      function createSkyBoxSources() {
        return {
          positiveX: "./skyboxImage/px.png",
          negativeX: "./skyboxImage/nx.png",
          positiveY: "./skyboxImage/pz.png",
          negativeY: "./skyboxImage/nz.png",
          positiveZ: "./skyboxImage/py.png",
          negativeZ: "./skyboxImage/ny.png",
        };
      }

      const heightLimit = 120000;
      document.getElementById("heightLimit").textContent = `${heightLimit} m`;

      const viewer = new Cesium.Viewer("cesiumContainer", {
        baseLayer: false, animation: false, timeline: false,
        fullscreenButton: false, infoBox: false, geocoder: false,
        homeButton: false, sceneModePicker: false, baseLayerPicker: false,
        navigationHelpButton: false, selectionIndicator: false,
        shouldAnimate: true,
      });

      viewer.imageryLayers.addImageryProvider(
        new Cesium.UrlTemplateImageryProvider({
          url: "https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}",
          maximumLevel: 18,
        })
      );

      viewer.scene.globe.enableLighting = false;
      viewer.scene.fog.enabled = true;
      viewer.scene.fog.density = 0.00018;
      viewer.scene.skyAtmosphere.show = true;
      viewer.scene.debugShowFramesPerSecond = true;
      viewer._cesiumWidget._creditContainer.style.display = "none";

      const skyBox = new SkyBox(viewer);
      skyBox.SkyBoxOnGround({
        height: heightLimit,
        sources: createSkyBoxSources(),
      });

      const cameraHeightEl = document.getElementById("cameraHeight");
      const skyboxStatusEl = document.getElementById("skyboxStatus");

      viewer.scene.postRender.addEventListener(() => {
        const height = Cesium.Cartographic.fromCartesian(viewer.camera.position).height;
        cameraHeightEl.textContent = `${height.toFixed(0)} m`;
        skyboxStatusEl.textContent = height < heightLimit ? "近地天空盒" : "默认天空盒";
      });

      function flyToNear() {
        viewer.camera.flyTo({
          destination: Cesium.Cartesian3.fromDegrees(116.3913, 39.9075, 5200),
          orientation: {
            heading: Cesium.Math.toRadians(18),
            pitch: Cesium.Math.toRadians(-12),
            roll: 0,
          },
          duration: 1.4,
        });
      }

      function flyToHigh() {
        viewer.camera.flyTo({
          destination: Cesium.Cartesian3.fromDegrees(116.3913, 39.9075, 280000),
          orientation: {
            heading: Cesium.Math.toRadians(0),
            pitch: Cesium.Math.toRadians(-80),
            roll: 0,
          },
          duration: 1.4,
        });
      }

      document.getElementById("nearBtn").addEventListener("click", flyToNear);
      document.getElementById("highBtn").addEventListener("click", flyToHigh);

      flyToNear();
      window.viewer = viewer;
    </script>
  </body>
</html>
```

## 总结

这个 Demo 看起来只是"换一组天空图"，但真正有价值的点在于：它展示了 Cesium 高层 API 之外的一条路——通过自定义 `DrawCommand` 直接插入渲染管线。

核心技术要点：

1. **自定义 DrawCommand**：实现 `update(frameState, useHdr)` 方法，返回 `DrawCommand` 加入渲染队列
2. **ENU 坐标对齐**：通过 `Transforms.eastNorthUpToFixedFrame` 让天空盒跟随相机位置
3. **高度切换**：`postRender` 回调中判断相机高度，低空用自定义天空盒，高空恢复默认
4. **Cubemap 方向映射**：拆图工具的坐标约定与 Cesium 不一致，需要手动调试匹配

使用建议：如果只是普通三维地球展示，用默认天空就好；如果你要做低空漫游、近地全景、沉浸式场景，那 `SkyBoxOnGround` 这套思路值得收起来。
