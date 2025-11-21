---
title: mockjs导致cesium地图无法加载
date: 2025-11-21 10:49:28
cover:
tags:
 - Cesium
 - mockjs
categories:
 - 教程 
keywords: "axios,教程"
---






# mockjs 导致cesium地图无法加载

[大盅码](/user/1869602729233272/posts)

2022-09-15 2,526 阅读1分钟

关注
1.  问题描述

    昨天在vue3+vite项目中加了cesium 用来展示3d地图，出现如下错误


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/175d56f04c184df2a84f722c6589d962~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

```markdown
js 体验AI代码助手 代码解读复制代码Uncaught (in promise) TypeError: Failed to execute 'createImageBitmap' on 'Window': The provided value is not of type '(Blob or HTMLCanvasElement or HTMLImageElement or HTMLVideoElement or ImageBitmap or ImageData or OffscreenCanvas or SVGImageElement or VideoFrame)'.
```

2.  问题分析

    面向搜索引擎一番，没有发现类似问题 开始以为是cesium的key有问题，更换key，更换成高德的图层都无效 于是仔细研究报错信息，发现它说的是提供的数据格式不对，开F12查看请求数据


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2551a06206ee41d2b0797df78ed54c87~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?) 发现请求数据http状态是304，而且是由mock.js发起的请求

3.问题解决

查了一下mock.js的工作原理，它直接替换了xhr请求，改成了自己的实现，如果路由没有匹配到，就进行跳转。这种实现方式对二进制流有较大的影响。 解决方案有两个，一个是去掉mock.js, 目前我们的项目阶段不允许。二是修改mock.js的源码。 代码路径：项目//node\_modules/mockjs/dist/mock.js 大约在8360行：

```markdown
js 体验AI代码助手 代码解读复制代码// 原生 XHR
                if (!this.match) {
                    this.custom.xhr.send(data)
                    return
                }
```

修改为

```markdown
js 体验AI代码助手 代码解读复制代码// 原生 XHR

                if (!this.match) {
                    this.custom.xhr.responseType = this.responseType
                    this.custom.xhr.send(data)
                    return
                }
```

当然这是一种比较偷懒的方式。修改完了以后要刷新。

4.  总结

    这个问题耽误了一些时间，开始是靠猜测调代码，这是主要浪费时间的地方。要善用开发者工具啊，能够省很多事，而且要仔细看调试信息