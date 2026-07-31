---
title: Cesium 全球体积云移植：从 GIS 迈向图形的 90% 还原之路
date: 2026-07-31 20:00:00
tags:
  - Cesium
  - 体积云
  - 渲染管线
  - 大气散射
  - WebGL
  - 三维GIS
categories:
  - 技术分享
keywords: "Cesium, 体积云, 全球体积云, 大气散射, 渲染管线, MRT, GBuffer, Texture2DArray, 阴影资源管理, 三维地球, 图形学, WebGL"
description: "本文记录将全球体积云移植到 Cesium 的过程，目前还原度约 90%。涵盖渲染管线、坐标转换、MRT、GBuffer 以及基于 Texture2DArray 的阴影资源管理等核心要点，并对比 Three.js 与 Cesium 在渲染架构上的差异，分享 inscatter 贴地、tonemap 冲突、色彩空间偏白等典型问题与思考。"
---

![Cesium 全球体积云](/img/cesium-volumetric-clouds-cover.jpg)

## 从 GIS 脱坑，向图形更近一步

鸽了好久，终于脱坑 GIS 了，也算是离图形更近了一些，这下可以专心研究图形了。

断断续续移植的全球体积云，总算是有了个像样的版本，目前还原大概 90%。

中间实在是踩了太多坑，但到最后把经验落到 skill 的时候发现，还是对渲染管线、坐标转换不熟悉造成的。如果基础很扎实，那从一开始思路定好，还是能省很多时间的。

## 这个移植项目到底有没有使用价值

总有人问这个到底有没有使用价值，我只想说：这个过程中我学到的东西和对渲染管线的理解，比付出的时间大得多。Cesium 的 render 流程也很值得学习。

抛开国别不谈，这个移植项目的思路和渲染管线的使用真的很值得学习。因为之前做大气散射的时候，虽然也能实现效果，但很多东西都需要适配 Cesium 的实现。本项目实现了 `MRT`、`GBuffer` 等，以及基于 `Texture2DArray` 的阴影资源管理能力。

> 移植项目开源地址：https://takram-design-engineering.github.io/

## 效果演进

先放几张之前的效果——地面光照、inscatter、云层阴影、lightshaft 还没加上的效果。

![Cesium 体积云效果](/img/cesium-volumetric-clouds-effect.png)

### 目前效果——体积云

体积云的核心是在屏幕空间重建世界坐标，按视线方向对云层进行 ray march，再结合大气散射、光照、阴影来合成最终颜色。下面是 1.5 倍速的演示视频（集显录制，ghost 与低帧率被放大，见谅）。

<video src="/img/cesium-volumetric-clouds-demo.mp4" controls="true" preload="metadata" width="100%"></video>

### 大气散射

大气散射效果主要解决了模型光照和地形契合的问题，老客户会陆续更新，请耐心等待。

## 目前还存在的几个大问题

### 1. inscatter 贴不住地形

Cesium 的真实地形终点问题导致 inscatter 在贴近地形时无法正确贴合，这是目前最难啃的一块。

### 2. 3D Tiles / model 与 tonemap 的冲突

虽然实现了同步光照，但模型本身 Cesium 还是对其做了处理，导致 tonemap 环节存在冲突。

### 3. 色彩空间还有优化空间

目前画面偏白，色彩空间仍有进一步优化的空间。

## Three.js vs Cesium：两种不同的渲染思路

Three 的问题通常是 **"怎么设计一条管线"**；

Cesium 的问题通常是 **"怎么在已有地球引擎里重新划分 ownership"**，要不断和它原生地球渲染系统做边界协调。

总的来说，Cesium 不像 Three 那样可以完全接管整个管线。

Cesium 要先解决的是**全球尺度场景的稳定渲染**，而不是先给你一个完全裸露的 pass graph。这套设计牺牲了一部分自由度，但换来了地球引擎级别的工程能力。

## 关于演示视频

本来不想录视频了——集显效果实在是太差，ghost 和低帧率被无限放大。但集显才是大多数的硬件配置，你也没法要求甲方爸爸换一个显卡。

为了观看体验，我还是没忍住加了 1.5 倍速。

在这里我给我的 443 个（有一个因为我鸽太久取关了，我劝你赶紧回来）粉丝承诺：换了好显卡一定补录。

## 写在最后

有了对渲染管线的理解，后面再做别的效果想必会比这个轻松些，坚持持续尝试。
