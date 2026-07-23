---
title: FormData对象上传二进制文件
date: 2026-07-23 19:00:00
tags:
  - Ajax
  - jQuery
  - FormData
  - 文件上传
categories:
  - 教程
keywords: "FormData, 文件上传, AJAX, jQuery, 二进制文件, XMLHttpRequest"
description: "本文详细讲解如何使用FormData对象组装键值对，配合XMLHttpRequest实现二进制文件的异步上传。涵盖文件对象结构、jQuery与原生Ajax两种实现方式，以及multipart/form-data编码格式下表单数据传输的原理，帮助开发者灵活方便地完成文件上传功能。"
---

> 转载自：[博客园-礼拜16](https://www.cnblogs.com/baiyygynui/p/8463771.html)

## 前言

通过FormData对象可以组装一组用 [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) 发送请求的键/值对。它可以更灵活方便的发送表单数据，因为可以独立于表单使用。如果你把表单的编码类型设置为 `multipart/form-data`，则通过FormData传输的数据格式和表单通过 [submit()](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLFormElement/submit) 方法传输的数据格式相同，也就是二进制文件。

## 文件对象的结构

不用 `<form>` 表单构造FormData对象，`var file = fileInput.files[0];` 它的 file 值为以下的图片的对象：

```js
{
  lastModified: 1247549551674,
  lastModifiedDate: "Tue Jul 14 2009 13:32:31 GMT+0800 (中国标准时间)",
  name: "ju.jpg",
  size: 879394,
  type: "image/jpeg",
  webkitRelativePath: ""
}
```

## 创建 FormData 对象

可以自己创建一个FormData对象，然后通过调用它的 [append()](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/append) 方法添加字段：

```js
var formData = new FormData();

formData.append("username", "Groucho");
formData.append("accountnum", 123456); // 数字 123456 会被立即转换成字符串 "123456"

// HTML 文件类型input，由用户选择
formData.append("userfile", fileInputElement.files[0]);

// JavaScript file-like 对象
var content = '<a id="a"><b id="b">hey!</b></a>';
var blob = new Blob([content], { type: "text/xml" });
formData.append("webmasterfile", blob);

var request = new XMLHttpRequest();
request.open("POST", "http://foo.com/submitform.php");
request.send(formData);
```

## 通过表单创建 FormData 对象

```html
<form id="uploadForm" enctype="multipart/form-data">
  <input id="file" type="file" name="file"/>
  <button id="upload" type="button">upload</button>
</form>
```

> `enctype="multipart/form-data"` 文件的二进制属性

## 使用 jQuery 的 Ajax 上传文件

上传文件部分只有底层的XMLHttpRequest对象发送上传请求，那么怎么通过jQuery的Ajax上传呢？

### 方式一：使用 `<form>` 表单初始化 FormData 对象

```js
$.ajax({
  url: '/upload',
  type: 'POST',
  cache: false,
  data: new FormData($('#uploadForm')[0]),
  processData: false,
  contentType: false
}).done(function(res) {
  // 上传成功
}).fail(function(res) {
  // 上传失败
});
```

**注意事项：**

- `processData` 设置为 `false`。因为 data 值是FormData对象，不需要对数据做处理。
- `<form>` 标签添加 `enctype="multipart/form-data"` 属性。
- `cache` 设置为 `false`，上传文件不需要缓存。
- `contentType` 设置为 `false`。因为是由 `<form>` 表单构造的FormData对象，且已经声明了属性 `enctype="multipart/form-data"`，所以这里设置为 false。

上传后，服务器端代码需要使用从查询参数名为 `file` 获取文件输入流对象，因为 `<input>` 中声明的是 `name="file"`。

### 方式二：使用 FormData 对象添加字段（常用）

这种方式不用 `<form>` 表单构造 FormData 对象，更加灵活：

```html
<div id="uploadForm">
  <input id="file" type="file" multiple/>
  <button id="upload" type="button">upload</button>
</div>
```

```js
var formData = new FormData();
formData.append('file', $('#file')[0].files[0]);

$.ajax({
  url: '/upload',
  type: 'POST',
  cache: false,
  data: formData,
  processData: false,
  contentType: false
}).done(function(res) {
  // 上传成功
}).fail(function(res) {
  // 上传失败
});
```

**与方式一的不同之处：**

- `append()` 的第二个参数应是文件对象，即 `$('#file')[0].files[0]`。
- `contentType` 也要设置为 `false`。

从代码 `$('#file')[0].files[0]` 中可以看到一个 `<input type="file">` 标签能够上传多个文件，只需要在 `<input type="file" multiple>` 里添加 `multiple` 或 `multiple="multiple"` 属性。

## 总结

通过 FormData 对象上传文件有两种主要方式：

1. **表单方式**：用 `<form>` 表单初始化 FormData 对象，适合表单内包含多个字段的场景
2. **手动 append 方式**：手动创建 FormData 对象并 append 字段，更加灵活，适合动态添加文件的场景

两种方式都需要在 jQuery 的 ajax 中设置 `processData: false` 和 `contentType: false`，以确保文件以二进制流的形式正确发送。
