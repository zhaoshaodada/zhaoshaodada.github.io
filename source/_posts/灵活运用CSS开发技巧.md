---
title: 灵活运用CSS开发技巧
date: 2026-07-23 13:00:00
tags:
  - CSS
  - 技巧
  - 前端
categories:
  - 教程
keywords: "CSS技巧, 布局技巧, 前端开发, 样式优化"
---

> 转载自：[掘金-JowayYoung](https://juejin.im/post/5d4d0ec651882549594e7293)

## 前言

何为技巧，意指表现在文学、工艺、体育等方面的巧妙技能。代码作为一门现代高级工艺，推动着人类科学技术的发展，同时犹如文字一样承托着人类文化的进步。

作为程序猿的我们，写代码同样也需要大量的写作技巧。一份良好的代码能让人耳目一新，让人容易理解，让人舒服自然，同时也让自己成就感满满。

## Layout Skill：布局技巧

### 使用vw定制rem自适应布局

```css
html {
  font-size: calc(100vw / 7.5);
}
```

### 使用:nth-child()选择指定元素

通过`:nth-child()`筛选指定的元素设置样式，适用于表格着色、边界元素排版。

### 使用writing-mode排版竖文

通过`writing-mode`调整文本排版方向，适用于竖行文字、文言文、诗词。

### 使用text-align-last对齐两端文本

```css
.text {
  text-align-last: justify;
}
```

### 使用:not()去除无用属性

```css
li:not(:last-child) {
  border-right: 1px solid #ccc;
}
```

### 使用object-fit规定图像尺寸

```css
img {
  object-fit: cover;
}
```

### 使用overflow-x排版横向列表

```css
.container {
  overflow-x: auto;
}
```

### 使用text-overflow控制文本溢出

```css
.single-ellipsis {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.multiline-ellipsis {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 4;
  overflow: hidden;
}
```

### 使用transform描绘1px边框

```css
.elem::before {
  content: '';
  position: absolute;
  width: 200%;
  height: 200%;
  border: 1px solid #ccc;
  transform: scale(0.5);
  transform-origin: top left;
}
```

### 使用transform翻转内容

```css
.flip-horizontal {
  transform: scaleX(-1);
}
```

### 使用letter-spacing排版倒序文本

```css
.text {
  letter-spacing: -0.5em;
}
```

### 使用margin-left排版左重右轻列表

```css
.flex-container {
  display: flex;
}

.last-item {
  margin-left: auto;
}
```

## Behavior Skill：行为技巧

### 使用overflow-scrolling支持弹性滚动

```css
body {
  -webkit-overflow-scrolling: touch;
}
```

### 使用transform启动GPU硬件加速

```css
.elem {
  transform: translate3d(0, 0, 0);
}
```

### 使用attr()抓取data-\*

```css
[data-tip]::after {
  content: attr(data-tip);
}
```

### 使用:valid和:invalid校验表单

```css
input:valid {
  border-color: green;
}
input:invalid {
  border-color: red;
}
```

### 使用pointer-events禁用事件触发

```css
.disabled {
  pointer-events: none;
}
```

### 使用:focus-within分发冒泡响应

```css
.form-group:focus-within {
  border-color: blue;
}
```

### 使用max-height切换自动高度

```css
.collapse {
  max-height: 0;
  overflow: hidden;
  transition: max-height 0.3s;
}
.collapse.open {
  max-height: 1000px;
}
```

### 使用transform模拟视差滚动

```css
.parallax {
  transform: translateZ(-1px) scale(2);
}
```

### 使用animation-delay保留动画起始帧

```css
.animation {
  animation-delay: -1s;
}
```

### 使用resize拉伸分栏

```css
.resizeable {
  resize: horizontal;
  overflow: auto;
}
```

## Color Skill：色彩技巧

### 使用color改变边框颜色

```css
.elem {
  border: 1px solid;
  color: #f66;
}
```

### 使用filter开启悼念模式

```css
body {
  filter: grayscale(100%);
}
```

### 使用::selection改变文本选择颜色

```css
::selection {
  background: #f66;
  color: #fff;
}
```

### 使用linear-gradient控制背景渐变

```css
.gradient {
  background: linear-gradient(to right, #f66, #f90);
}
```

### 使用caret-color改变光标颜色

```css
input {
  caret-color: #f66;
}
```

### 使用::scrollbar改变滚动条样式

```css
::-webkit-scrollbar {
  width: 8px;
}
::-webkit-scrollbar-thumb {
  background: #ccc;
}
```

## 总结

以上是一些CSS开发中常用的技巧，希望能帮助你写出更好的代码。
