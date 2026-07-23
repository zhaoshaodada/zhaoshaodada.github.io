---
title: 使用Node Express创建后端返回假数据
date: 2026-07-23 19:10:00
tags:
  - Node.js
  - Express
  - Mock数据
  - 前后端联调
categories:
  - 教程
keywords: "Node.js, Express, Mock数据, 假数据, 前后端联调, 分页, JSONP"
description: "本文介绍如何使用Node.js与Express框架搭建本地后端服务，承接前端请求并分页返回假数据，支持JSONP跨域访问。该方案可用于编写无限下拉等依赖后端数据组件时的顺畅调试，也适合Mock数据模拟前后端联调，让真实数据接入时能快速对接上线。"
---

> 转载自：[简书-Jenny_L](https://www.jianshu.com/p/26262bea32e6)

## 前言

这篇文章创建了一个Node Express应用，承接前端请求，分页返回假数据。

这个方法可在编写无限下拉等依赖后端数据组件时，提供顺畅的假数据；也可Mock数据用于模拟前后端联调。这样在真正数据来的时候，能够顺畅对接，快速将代码推动上线。

创建一个Express假数据应用，需要以下步骤：

1. 安装Express及npm工具
2. 新建Express应用
3. 配置JSON文件
4. 配置Router承接请求并返回

## 一、安装Express及npm工具

1. 命令行安装 express
```bash
$ [sudo] npm install -g express
```

2. 命令行安装 express 应用生成器
```bash
$ [sudo] npm install -g express-generator
```

3. 效果Check：运行 `express -h` 命令，如果返回帮助信息，则安装成功

## 二、新建Express应用

1. 新建Express应用，会创建一个myapp文件夹
```bash
$ express myapp
```

2. 安装依赖（`&&` 在shell中的意思是连续执行多个命令）
```bash
$ cd myapp && npm install
```

3. 启动应用
```bash
# mac 启动应用
$ DEBUG=myapp npm start
# win 启动应用
$ set DEBUG=myapp & npm start
```

4. 效果Check：访问 `http://localhost:3000/` 来访问应用。如果看到欢迎页面，则说明建立成功。

## 三、配置JSON文件

在 `app.js` 文件中有这么一行代码，用于配置静态文件目录：

```js
app.use(express.static(path.join(__dirname, 'public')));
```

当在浏览器中访问静态文件，实际上是按照配置查找的 public 文件夹。比如：

- 实际访问：`http://0.0.0.0:3000/stylesheets/style.css`
- 文件存放：`myapp/public/stylesheets/style.css`

所以，可以把json文件也放到这个文件夹中，并在express中获取：

1. 在 `myapp/public` 目录下新建 json 文件夹。
2. 在 `myapp/public/json` 下新建 `infinite.json`，用于存储无限下拉的假数据。
3. `infinite.json` 内容填写：

```json
[{
  "title": "风信子",
  "img": "https://ss0.baidu.com/6ONWsjip0QIZ8tyhnq/it/u=3495450057,3472067227&fm=5"
}, {
  "title": "紫罗兰",
  "img": "https://ss2.baidu.com/6ONYsjip0QIZ8tyhnq/it/u=3903672296,3890938056&fm=5"
}]
```

4. 效果Check：浏览器访问 `http://0.0.0.0:3000/json/infinite.json`，如果看到正常返回数据，就说明文件创建成功。

## 四、配置Router承接请求并返回

Express Router（路由）的使用可以参考[中文文档](http://www.expressjs.com.cn/guide/routing.html)，这里只使用最简单的一种，GET。

### 1. 配置 get 路由

在 `app.js` 中，app声明后（估计在12行），增加以下代码。`res.send` 用于回应一个请求。

```js
app.get('/mip', function (req, res) {
  res.send('<h1>mip路径访问成功！</h1>');
});
```

需要注意的是，Express不会自动监听代码修改，所以修改完代码后需要重启应用。

效果Check：访问 `http://0.0.0.0:3000/mip` 能看到 `mip路径访问成功！`

### 2. 配置获取数据

- 在Node中，获取内容可以使用 `require`。只要找准路径，就可以获取到刚才创建的Json文件。
- `res.jsonp()` 用于跨域返回json数据。

```js
app.get('/mip', function (req, res) {
  var json = require('./public/json/infinite.json');
  res.jsonp(json);
});
```

效果Check：访问 `http://0.0.0.0:3000/mip` 能看到数据。

### 3. 按照无限下拉方式返回分页内容

如果需要模拟无限下拉，假设每页6条，分页逻辑应该是这样：

- 第一页：返回 0~5 条，如 `json[0]` ~ `json[5]`
- 第二页：返回 6~11 条
- 第 n 页：返回 `6*(n-1)` ~ `6*(n-1)+5` 条，需要保证下标小于 `json.length`

完整代码如下：

```js
app.get('/mip', function (req, res) {
  var currentPage = req.query.pn || 1; // 从请求中获取当前页数, 1为第一页
  var itemsPerPage = 6; // 每页条数
  // 获取JSON假数据
  var json = require('./public/json/infinite.json');
  // 创建返回数组, 如要获取第2页数据，就是下标第6-11条
  var itemArr = [];
  for (var i = 0; i < itemsPerPage; i++) {
    var currentIndex = (currentPage - 1) * 6 + i;
    // 保证获取数据的 index 不大于数据总条数
    if (currentIndex < json.length) {
      json[currentIndex]["index"] = currentIndex + 1;
      itemArr.push(json[currentIndex]);
    }
  }
  // 创建返回值
  res.jsonp({
    status: 0,
    data: {
      items: itemArr
    }
  });
});
```

效果Check：访问 `http://0.0.0.0:3000/mip`，看到第一页 index 1~6 六条数据。

- 由于不是异步请求，`req.query.pn` 为空，默认获取第一页内容，即前六条内容。
- 如果是异步请求，带有 `pn` 参数，就会返回对应的值。

## 总结

至此，一个专门用来返回假数据的Node服务就搭建完成。这个方案适用于：

- **无限下拉组件开发**：提供顺畅的分页假数据
- **前后端联调**：Mock数据模拟接口返回
- **快速原型开发**：不依赖真实后端即可推进前端开发

当真实数据来临时，只需替换接口地址即可无缝对接。
