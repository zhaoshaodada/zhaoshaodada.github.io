---
title: CSS实现漂亮的滚动条样式
date: 2026-07-23 15:00:00
tags:
  - CSS
  - 滚动条
  - 样式
categories:
  - 教程
keywords: "CSS滚动条, 自定义滚动条, 美化滚动条"
---

> 转载自：[简书-nomooo](https://www.jianshu.com/p/c2addb233acd)

## 实现代码

### 样式一：经典款

**HTML**

```html
<div class="test test-1">
  <div class="scrollbar"></div>
</div>
```

**CSS**

```css
.test {
  width: 50px;
  height: 200px;
  overflow: auto;
  float: left;
  margin: 5px;
  border: none;
}
.scrollbar {
  width: 30px;
  height: 300px;
  margin: 0 auto;
}
.test-1::-webkit-scrollbar {
  width: 10px;
  height: 1px;
}
.test-1::-webkit-scrollbar-thumb {
  border-radius: 10px;
  box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
  background: #535353;
}
.test-1::-webkit-scrollbar-track {
  box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
  border-radius: 10px;
  background: #ededed;
}
```

### 样式二：渐变款

**HTML**

```html
<div class="test test-5">
  <div class="scrollbar"></div>
</div>
```

**CSS**

```css
.test-5::-webkit-scrollbar {
  width: 10px;
  height: 1px;
}
.test-5::-webkit-scrollbar-thumb {
  border-radius: 10px;
  background-color: skyblue;
  background-image: -webkit-linear-gradient(
    45deg,
    rgba(255, 255, 255, 0.2) 25%,
    transparent 25%,
    transparent 50%,
    rgba(255, 255, 255, 0.2) 50%,
    rgba(255, 255, 255, 0.2) 75%,
    transparent 75%,
    transparent
  );
}
.test-5::-webkit-scrollbar-track {
  box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
  background: #ededed;
  border-radius: 10px;
}
```

## 总结

通过`::-webkit-scrollbar`相关伪元素，我们可以轻松自定义滚动条的外观：

- `::-webkit-scrollbar` — 滚动条整体样式
- `::-webkit-scrollbar-thumb` — 滚动条里面小方块
- `::-webkit-scrollbar-track` — 滚动条里面轨道

这些属性目前仅在WebKit内核的浏览器中支持，在Firefox和IE中需要使用不同的方法。
