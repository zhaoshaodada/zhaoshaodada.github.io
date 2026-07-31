---
title: Vibe Coding — 当 DeepSeek 遇见 Three.js 的治愈深海
date: 2026-07-27 22:00:00
tags:
  - Three.js
  - p5.js
  - WebGL
  - Web Audio API
  - 创意编程
categories:
  - 技术分享
keywords: "Three.js, p5.js, Web Audio API, 粒子系统, 窗帘代码雨, 虎鲸, DeepSeek, Vibe Coding, 创意编程, 纯前端"
description: "一个纯前端治愈系网页：深蓝深海背景，中文词语如窗帘般缓缓下滚，鼠标拨开时文字碎裂成粒子，背景里虎鲸在游动，点击发出海豚音。Three.js 画虎鲸，p5.js 画窗帘和粒子，Web Audio API 合成声音。"
---

## 一个治愈系网页

前两天看到一个网页，很有意思。作者还把用 DeepSeek Vibe Coding 的过程分享出来了！

> https://chat.deepseek.com/share/vvxp19xmnl9mndybpo

打开后，深蓝色的背景，像深海。前景是一排排竖着的中文词语，像窗帘一样缓缓往下滚动。鼠标滑过去，窗帘会被拨开，文字还会碎裂成粒子。背景里有一条虎鲸在游动，点击它会发出海豚音。

整个页面没有服务器，纯前端。Three.js 画虎鲸，p5.js 画窗帘和粒子。音频用 Web Audio API 生成。

## 实现思路

### 1. 虎鲸部分（Three.js）

用 `LatheGeometry` 旋转体轮廓生成身体，再加背鳍、胸鳍、尾鳍。全部转成粒子点云，看起来半透明、发光。鼠标控制位置，滚轮控制大小。背后还有一条更大的阴影虎鲸在缓慢旋转。

核心是定义虎鲸的身体轮廓曲线：

```javascript
const BODY_PROFILE = [
  [0.0, 2.35], [0.15, 2.32], [0.35, 2.25], [0.55, 2.12],
  [0.75, 1.95], [0.95, 1.75], [1.12, 1.45], [1.22, 1.15],
  [1.28, 0.85], [1.32, 0.55], [1.34, 0.25], [1.32, -0.05],
  [1.25, -0.35], [1.12, -0.65], [0.95, -0.95], [0.72, -1.25],
  [0.48, -1.55], [0.28, -1.85], [0.12, -2.1], [0.0, -2.3]
];
```

然后通过 `CatmullRomCurve3` 生成平滑曲线，再用 `LatheGeometry` 旋转成型。关键是将网格转为粒子点云：

```javascript
function meshToPoints(geometry, color, size = 0.04, opacity = 0.35) {
  const positions = geometry.attributes.position.array;
  const colors = new Float32Array(positions.length);
  const colorObj = new THREE.Color(color);
  for (let i = 0; i < positions.length; i += 3) {
    colors[i] = colorObj.r;
    colors[i + 1] = colorObj.g;
    colors[i + 2] = colorObj.b;
  }
  const pointGeo = new THREE.BufferGeometry();
  pointGeo.setAttribute('position', new THREE.BufferAttribute(positions.slice(), 3));
  pointGeo.setAttribute('color', new THREE.BufferAttribute(colors, 3));
  return new THREE.Points(pointGeo, new THREE.PointsMaterial({
    size, vertexColors: true,
    blending: THREE.AdditiveBlending,
    depthWrite: false, transparent: true, opacity
  }));
}
```

虎鲸的腹部白色和背部深蓝色通过顶点颜色混合实现：

```javascript
for (let i = 0; i < bPA.count; i++) {
  const y = bPA.getY(i);
  let c;
  if (y > 0.05) c = dB;           // 背部深蓝
  else if (y < -0.05) c = pW;     // 腹部白色
  else c = new THREE.Color().lerpColors(pW, dB, (y + 0.05) / 0.1); // 过渡带
  bCol[i * 3] = c.r;
  bCol[i * 3 + 1] = c.g;
  bCol[i * 3 + 2] = c.b;
}
```

动画部分通过 `lerp` 实现平滑跟随，胸鳍和尾鳍用 `sin` 波形摆动：

```javascript
orcaGroup.position.lerp(targetPos, 0.1);
lPG.rotation.x = -0.25 + Math.sin(t * 3.5) * 0.35;        // 左胸鳍
rPG.rotation.x = -0.25 + Math.sin(t * 3.5 + Math.PI) * 0.35; // 右胸鳍
const tf = Math.sin(t * 4.5) * 0.35;
tTG.rotation.x = -Math.PI / 2 + tf;  // 尾鳍
```

### 2. 窗帘部分（p5.js）

每列是一个 `CurtainColumn`，里面是随机从词库抽的中文短句，竖直滚动。鼠标移动时给附近的列施加力，像拨窗帘。速度够快时，碰到的字符会碎裂成粒子，并播放一小段音效。

词库是一组治愈系中文短句：

```javascript
const wordPool = [
  '早安', '午安', '晚安', '好梦', '休息', '放松',
  '阳光', '微风', '细雨', '星空', '月光', '云朵',
  '海浪', '森林', '山谷', '溪流', '花瓣', '落叶',
  '猫咪', '小狗', '兔子', '小鸟', '金鱼', '蝴蝶',
  '书页', '咖啡', '热茶', '毛毯', '沙发', '台灯',
  '代码', '画笔', '音符', '诗歌', '故事', '日记',
  '晴天', '彩虹', '晚霞', '晨露', '雪花', '春风',
  '港湾', '小憩', '发呆', '散步', '深呼吸', '慢慢来',
  '今天很好', '明天也是', '没事的', '我在呢',
  '放轻松', '别着急', '你很好', '会好的',
  '吃了吗', '睡了吗', '想你了', '等你呢',
  '光', '暖', '静', '安', '闲', '梦',
  '虎鲸', 'Deepseek', '稳稳的接住', '你说得对', '看见', '靠近',
];
```

窗帘的物理模拟用弹簧+阻尼模型：

```javascript
update() {
  this.prevOffsetX = this.offsetX;
  const stiffness = 0.08, damping = 0.78;
  this.velocityX = (this.velocityX - this.offsetX * stiffness) * damping;
  this.offsetX += this.velocityX;
  this.offsetX = constrain(this.offsetX, -this.maxOffset, this.maxOffset);
  this.glowAlpha *= 0.9;
  this.scrollOffset += this.scrollSpeed;
}
```

鼠标快速划过时，文字碎裂成粒子：

```javascript
checkShatter(mx, my, ms) {
  if (ms < 5) return;
  const x = this.baseX + this.offsetX;
  const chars = this.phrase.split('');
  const startY = -this.scrollOffset;
  for (let i = 0; i < chars.length; i++) {
    if (this.shatteredIndices.has(i)) continue;
    const y = startY + i * this.charSpacing;
    const d = dist(mx, my, x, y);
    const threshold = this.fontSize * 0.8;
    if (d < threshold) {
      this.shatteredIndices.add(i);
      const count = floor(random(8, 16));
      for (let j = 0; j < count; j++) {
        shatterParticles.push(new ShatterParticle(x, y, this.r, this.g, this.b));
      }
      return true;
    }
  }
  return false;
}
```

碎裂粒子有重力、衰减、发光效果：

```javascript
class ShatterParticle {
  constructor(x, y, r, g, b) {
    this.x = x; this.y = y;
    this.vx = (random() - 0.5) * 4.5;
    this.vy = (random() - 0.5) * 4.5 - random() * 2;
    this.life = 1.0;
    this.decay = 0.025 + random() * 0.04;
    this.size = random(2, 6);
    this.r = r + random() * 40;
    this.g = g + random() * 25;
    this.b = b + random() * 20;
    this.glow = random() > 0.5;
  }
  update() {
    this.x += this.vx;
    this.y += this.vy;
    this.vy += 0.04;  // 重力
    this.life -= this.decay;
    this.size *= 0.985;
  }
}
```

### 3. 声音（Web Audio API）

全部用 Web Audio API 合成，不依赖任何音频文件：

- **拨动窗帘**：五声音阶的轻响（triangle 波 + sine 泛音）
- **点击虎鲸**：上升的正弦波，模拟海豚音
- **文字碎裂**：短促的带通滤波声

```javascript
function playDolphinSound() {
  const now = audioCtx.currentTime;
  const duration = 0.25;
  const baseFreq = 750 + Math.random() * 450;
  const endFreq = baseFreq * 1.6;
  const osc = audioCtx.createOscillator();
  osc.type = 'sine';
  osc.frequency.setValueAtTime(baseFreq, now);
  osc.frequency.linearRampToValueAtTime(endFreq, now + duration);
  const gain = audioCtx.createGain();
  gain.gain.setValueAtTime(0, now);
  gain.gain.linearRampToValueAtTime(0.18, now + 0.02);
  gain.gain.exponentialRampToValueAtTime(0.001, now + duration);
  osc.connect(gain).connect(masterGain);
  osc.start(now);
  osc.stop(now + duration);
}
```

五声音阶的随机轻响营造治愈氛围：

```javascript
let pentatonicScale = [523.25, 587.33, 659.25, 783.99, 880.0, 1046.5, 1174.66];
```

### 4. 氛围层

背景有缓慢下沉的星点、水波纹、小鱼群。底部和顶部还有淡淡的渐变光。

- **星点**：100 个随机粒子，缓慢上浮+闪烁
- **水波纹**：多层正弦波叠加，模拟水面光影
- **小鱼群**：32 条 p5.js 绘制的卡通小鱼，随机游动
- **Three.js 远景鱼群**：200 个点粒子在三维空间中游动
- **星海**：900 个球面分布的远景点

水波纹用多层正弦波叠加：

```javascript
for (let y = 0; y < height; y += 4) {
  let wv = sin(y * 0.018 + frameCount * 0.022) * 0.45
         + sin(y * 0.027 - frameCount * 0.016 + 1.7) * 0.35;
  wv += cos(y * 0.009 + frameCount * 0.011 + 3.1) * 0.28
      + sin(y * 0.035 + frameCount * 0.028 + 5.4) * 0.2;
  const a = constrain((wv + 1.28) * 4.5, 0, 13);
  if (a > 0.4) {
    ctxWater.fillStyle = `rgba(175,210,240,${a / 255})`;
    ctxWater.fillRect(0, y, width, 3.5);
  }
}
```

## 技术架构总结

| 层 | 技术 | 职责 |
|----|------|------|
| 背景层（z-index: 1） | Three.js | 虎鲸、阴影虎鲸、远景鱼群、星海 |
| 前景层（z-index: 2） | p5.js | 窗帘文字、碎裂粒子、水波纹、小鱼、涟漪 |
| 音频层 | Web Audio API | 五声音阶轻响、海豚音、碎裂声 |
| UI 层（z-index: 10） | HTML/CSS | 提示文字 |

## 完整源码

> 保存成 `.html` 文件，用浏览器打开就能跑（需要联网加载 Three.js 和 p5.js）。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>窗帘代码雨 · 虎鲸深海</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            background: #0a0e1a;
            overflow: hidden;
            font-family: 'Courier New', 'PingFang SC', 'Microsoft YaHei', monospace;
            cursor: default;
        }
        #orca-canvas {
            position: fixed; top: 0; left: 0;
            z-index: 1; pointer-events: none;
        }
        .p5canvas {
            position: fixed; top: 0; left: 0;
            z-index: 2; pointer-events: auto;
        }
        .hint {
            position: fixed; bottom: 26px; left: 50%;
            transform: translateX(-50%); z-index: 10;
            color: rgba(170, 210, 240, 0.45);
            font-size: 12px; letter-spacing: 0.05em;
            pointer-events: none;
            text-shadow: 0 0 14px rgba(150, 200, 235, 0.35);
            font-family: 'PingFang SC', 'Microsoft YaHei', sans-serif;
            transition: opacity 1.8s;
        }
    </style>
</head>
<body>
    <div class="hint" id="hint">✨ 拨动窗帘 · 文字碎裂 · 点击虎鲸听海豚音</div>

    <script type="importmap">
        { "imports": { "three": "https://unpkg.com/three@0.128.0/build/three.module.js" } }
    </script>
    <script type="module">
        import * as THREE from 'three';
        // ... 完整代码见原文 ...
    </script>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.0/p5.min.js"></script>
    <script>
        // ... p5.js 窗帘和粒子代码 ...
    </script>
    <div id="p5-container"></div>
</body>
</html>
```

> 完整源码较长（约 1000 行），已在原文中贴出。核心逻辑已在上文中拆解说明。

## 写在最后

这个页面没有复杂的业务逻辑，就是视觉和交互。做完之后自己玩了好几分钟，挺治愈的。

如果你也喜欢这种"打开浏览器就能看到一点小惊喜"的东西，可以把源码存下来，改改词库，换成自己喜欢的句子。

**Vibe Coding 的意义就在这里**——不是为了解决什么技术难题，而是用代码创造一点点美。DeepSeek 帮你把想法变成现实，你只需要告诉它你想要什么感觉。
