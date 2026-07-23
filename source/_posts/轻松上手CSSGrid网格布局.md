---
title: 轻松上手CSS Grid网格布局
date: 2026-07-23 16:00:00
tags:
  - CSS
  - Grid
  - 布局
categories:
  - 教程
keywords: "CSS Grid, 网格布局, Grid布局, 布局技巧"
description: "本文以实际错落网格布局需求为切入点，系统讲解CSS Grid网格布局的基本概念与使用技巧。文章从格子的组成入手，结合实际案例演示Grid布局的实现方式，帮助前端开发者轻松上手并灵活运用CSS Grid完成复杂的二维布局任务。"
---

> 转载自：[CSDN-original_heart](https://blog.csdn.net/original_heart/article/details/94761295)

## 前言

今天刚好要做一个好多div格子错落组成的布局，不是田字格，不是九宫格，12个格子这样子，看起来有点复杂。关键的是笔者有点懒，要写那么多div和css真是不想下手啊。多看了两眼，这布局不跟网格挺像吗？css grid好像就是长这样子的？会不会很简单呢？反正也不熟，实在不行就当学习了。说干就干，说不定能偷点懒呢哈哈～

## 基本概念

Grid布局与Flex布局有一定的相似性，都可以指定容器内部多个项目的位置。但是，它们也存在重大区别。

- **Flex布局**：轴线布局，只能指定"项目"针对轴线的位置，可以看作是**一维布局**
- **Grid布局**：将容器划分成"行"和"列"，产生单元格，然后指定"项目所在"的单元格，可以看作是**二维布局**

Grid布局远比Flex布局强大。

## 示例：实现12格布局

### HTML结构

```html
<div class="main">
  <div class="grid-container">
    <div class="grid-item one">1</div>
    <div class="grid-item two">2</div>
    <div class="grid-item three">3</div>
    <div class="grid-item four">4</div>
    <div class="grid-item five">5</div>
    <div class="grid-item six">6</div>
    <div class="grid-item seven">7</div>
    <div class="grid-item eight">8</div>
    <div class="grid-item nine">9</div>
    <div class="grid-item ten">10</div>
    <div class="grid-item eleven">11</div>
    <div class="grid-item twelve">12</div>
  </div>
</div>
```

### 第一步：设置网格属性

```css
.main {
  width: 1200px;
  margin: 30px auto 0;
}
.grid-container {
  display: grid;
  grid-template-columns: 385px 180px 180px 180px 180px;
  grid-template-rows: 180px 180px 180px 180px;
}
.grid-item {
  border: 2px solid rgb(233, 171, 88);
  border-radius: 5px;
  background-color: rgba(233, 171, 88, .5);
}
```

### 第二步：添加间距

```css
.grid-container {
  grid-column-gap: 24px;
  grid-row-gap: 24px;
}
```

### 第三步：跨行跨列

```css
.grid-item.one {
  grid-row: 1 / 3;
}
.grid-item.two {
  grid-column: 2 / 4;
}
.grid-item.three {
  grid-column: 4 / 6;
}
.grid-item.six {
  grid-column: 4 / 6;
}
.grid-item.eight {
  grid-column: 2 / 4;
  grid-row: 3 / 5;
}
.grid-item.ten {
  grid-column: 5 / 6;
  grid-row: 3 / 5;
}
```

### 使用span关键字

上面的代码还可以使用`span`关键字简化：

```css
.grid-item.one {
  grid-row: 1 / span 2;
}
.grid-item.two {
  grid-column: 2 / span 2;
}
.grid-item.three {
  grid-column: 4 / span 2;
}
```

## 关键属性

### 容器属性

- `display: grid` — 设置容器为网格布局
- `grid-template-columns` — 定义每一列的列宽
- `grid-template-rows` — 定义每一行的行高
- `grid-column-gap` — 设置列与列的间隔
- `grid-row-gap` — 设置行与行的间隔

### 项目属性

- `grid-column-start` / `grid-column-end` — 左右边框所在的垂直网格线
- `grid-row-start` / `grid-row-end` — 上下边框所在的水平网格线
- `grid-column` — `grid-column-start`和`grid-column-end`的简写
- `grid-row` — `grid-row-start`和`grid-row-end`的简写

## 总结

Grid网格布局很强大，采用网格布局的区域，称为"容器"（container）。容器内部子元素，称为"项目"（item）。利用Grid布局可以很轻松地实现很多的网页布局。

**注意**：微信浏览器下不兼容Grid布局，可选用flex或者float等布局实现。
