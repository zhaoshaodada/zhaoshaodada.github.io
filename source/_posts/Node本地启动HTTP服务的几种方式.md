---
title: Node本地启动HTTP服务的几种方式
date: 2026-07-23 12:00:00
tags:
  - Node.js
  - HTTP
  - 服务器
categories:
  - 教程
keywords: "Node.js, HTTP服务器, 本地服务, express"
description: "本文总结使用Node.js本地启动HTTP服务的多种方式，涵盖Node内置http模块的使用、express框架的快速搭建以及其他常见方案。文章以代码示例对比各方式的优劣与适用场景，帮助开发者在学习与调试时快速选择合适的本地服务搭建方法。"
---

> 转载自：[CSDN-weixin_34336292](https://blog.csdn.net/weixin_34336292/article/details/91375864)

## 前言

最近学习 Nodejs，总结出本地启动 node 服务的几种方式，供大家参考。

## 方法一：Node内置模块http的使用

```javascript
var http = require('http')
http.createServer(function(req, res){
  res.writeHead(200, {'Content-Type': 'text-plain'});
  res.end('Hello World');
}).listen(8083);
```

## 方法二：使用express

```javascript
const express = require('express')
const app = express()

app.use(express.static('public'))

app.listen(8083, () => {
  console.log('server start!')
})
```

最后访问`localhost:8083`即可访问。

## 方法三：使用http-server（最方便）

```bash
# Step 1：全局安装 http-server
npm install http-server -g

# Step 2：进入目标文件夹启动 http-server
cd dist
http-server

# 或者指定端口号
http-server -p 3000

# Step 3：访问
# localhost:8080 或者 localhost:3000
```

## 总结

| 方法 | 优点 | 缺点 |
|---|---|---|
| http模块 | 原生，无需依赖 | 功能简陋 |
| express | 功能丰富，生态完善 | 需要安装依赖 |
| http-server | 零配置，启动最快 | 功能有限 |
