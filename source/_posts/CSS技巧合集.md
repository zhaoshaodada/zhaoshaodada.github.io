---
title: CSS技巧合集
date: 2026-07-23 14:00:00
tags:
  - CSS
  - 技巧
categories:
  - 教程
keywords: "CSS技巧, 三角形, 滚动条, 时间轴, 卡券"
description: "本文总结了一批常用又有趣的CSS技巧，涵盖三角形绘制、滚动条样式、时间轴布局、卡券效果等多种实用场景。文章以减少查资料时间为目的，提供可直接复用的代码片段与实现思路，帮助前端开发者提升样式编写效率与表现力。"
---

> 转载自：[掘金-前端论道](https://juejin.im/post/5ed3c27ee51d455f9a6368c9)

下面总结了一些常用又有趣的css技巧，希望大家收藏，以减少查资料的时间。

## 三角形

最常见的一种形状了。切图，不存在的。

```css
.triangle {
  width: 0;
  height: 0;
  border-style: solid;
  border-width: 0 25px 40px 25px;
  border-color: transparent transparent rgb(245, 129, 127) transparent;
}
```

倒三角：

```css
.triangle {
  width: 0;
  height: 0;
  border-style: solid;
  border-width: 40px 25px 0 25px;
  border-color: rgb(245, 129, 127) transparent transparent transparent;
}
```

## 虚线效果

```css
.dotted-line {
  border: 1px dashed transparent;
  background: linear-gradient(white, white) padding-box,
    repeating-linear-gradient(-45deg, #ccc 0, #ccc .25em, white 0, white .75em);
}
```

css自带的border-style属性dotted/dashed效果展示出来太密了，并不符合UI设计。具体的虚线的颜色和间距都可以通过repeating-linear-gradient生成的条纹背景去调整。

## 文本超出省略号

单行文本：

```css
.single-ellipsis {
  width: 500px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
```

多行文本：

```css
.multiline-ellipsis {
  display: -webkit-box;
  word-break: break-all;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 4;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

## 时间轴

HTML结构：

```html
<div class="timeline-content">
  <div v-for='(item, index) in timeLine' :key='index' class="time-line">
    <div :class="`state-${item.state} state-icon`"></div>
    <div class="timeline-title">{{item.title}}</div>
  </div>
</div>
```

CSS样式：

```css
.timeline-content {
  display: flex;
}
.time-line {
  padding: 10px 10px 10px 20px;
  position: relative;
  &::before {
    content: '';
    height: 1px;
    width: calc(100% - 34px);
    background: #EBEBEB;
    position: absolute;
    left: 24px;
    top: 0;
  }
}
.state-icon {
  width: 20px;
  height: 20px;
  position: absolute;
  top: -12px;
  left: 0;
}
```

## 滚动条

```css
.scroll-container {
  height: 250px;
  border: 1px solid #ddd;
  padding: 15px;
  overflow: auto;
}
.scroll-container::-webkit-scrollbar {
  width: 8px;
  background: white;
}
.scroll-container::-webkit-scrollbar-track {
  background-color: rgba(180, 160, 120, 0.1);
  box-shadow: inset 0 0 1px rgba(180, 160, 120, 0.5);
}
.scroll-container::-webkit-scrollbar-thumb {
  background-color: #00adb5;
}
```

## 卡券效果

```css
.coupon {
  width: 300px;
  height: 100px;
  position: relative;
  background: 
    radial-gradient(circle at right bottom, transparent 10px, #ffffff 0) top right /50% 51px no-repeat,
    radial-gradient(circle at left bottom, transparent 10px, #ffffff 0) top left / 50% 51px no-repeat,
    radial-gradient(circle at right top, transparent 10px, #ffffff 0) bottom right / 50% 51px no-repeat,
    radial-gradient(circle at left top, transparent 10px, #ffffff 0) bottom left / 50% 51px no-repeat;
  filter: drop-shadow(2px 2px 2px rgba(0,0,0,.2));
}
```

## 阴影效果

```css
.shadow-triangle {
  width: 0;
  height: 0;
  border-style: solid;
  border-width: 0 50px 50px 50px;
  border-color: transparent transparent rgb(245, 129, 127) transparent;
  filter: drop-shadow(10px 0px 10px rgba(238, 125, 55, 0.5));
}
```

## 等高布局

```css
.parent {
  display: flex;
}
```
