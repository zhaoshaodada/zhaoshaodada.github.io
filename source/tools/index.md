---
title: 在线工具
date: 2026-07-23 19:30:00
type: "tools"
top_img: /img/tools.jpg
comments: true
description: "前端开发者在线工具箱，提供Base64编解码、JSON格式化、时间戳转换、二维码生成、正则表达式测试、图片压缩、图片格式转换、图片尺寸修改、Markdown编辑器、颜色选择器、UUID生成、密码生成、字符统计、单位换算、HTTP状态码查询、CSV与JSON互转、CRON表达式生成等19款纯前端工具，数据不上传，安全高效。"
keywords: "在线工具, 前端工具, Base64, JSON格式化, 时间戳转换, 二维码, 正则表达式, 图片压缩, 图片格式转换, PNG转JPEG, 图片尺寸修改, Markdown, 颜色选择器, UUID, 密码生成, 字符统计, 单位换算, HTTP状态码, CSV, JSON, CRON"
---

## 🛠️ 在线工具箱

收集了一些前端开发常用的小工具，纯前端实现，数据不会上传服务器，安全放心。

{% raw %}

<div class="tool-grid">
  <a class="tool-card" href="/tools/base64.html">
    <div class="tool-icon">🔐</div>
    <div class="tool-title">Base64 编解码</div>
    <div class="tool-desc">文本与 Base64 互相转换</div>
  </a>
  <a class="tool-card" href="/tools/json.html">
    <div class="tool-icon">📋</div>
    <div class="tool-title">JSON 格式化</div>
    <div class="tool-desc">JSON 格式化、压缩、校验</div>
  </a>
  <a class="tool-card" href="/tools/timestamp.html">
    <div class="tool-icon">⏰</div>
    <div class="tool-title">时间戳转换</div>
    <div class="tool-desc">Unix 时间戳与日期互转</div>
  </a>
  <a class="tool-card" href="/tools/qrcode.html">
    <div class="tool-icon">📱</div>
    <div class="tool-title">二维码生成</div>
    <div class="tool-desc">输入文本生成二维码图片</div>
  </a>
  <a class="tool-card" href="/tools/regex.html">
    <div class="tool-icon">🔎</div>
    <div class="tool-title">正则表达式测试</div>
    <div class="tool-desc">实时匹配高亮，支持捕获组</div>
  </a>
  <a class="tool-card" href="/tools/image-compress.html">
    <div class="tool-icon">🖼️</div>
    <div class="tool-title">图片压缩</div>
    <div class="tool-desc">Canvas压缩，支持调质量</div>
  </a>
  <a class="tool-card" href="/tools/markdown.html">
    <div class="tool-icon">📝</div>
    <div class="tool-title">Markdown 编辑器</div>
    <div class="tool-desc">实时预览，支持导出HTML</div>
  </a>
  <a class="tool-card" href="/tools/color-picker.html">
    <div class="tool-icon">🎨</div>
    <div class="tool-title">颜色选择器</div>
    <div class="tool-desc">HEX/RGB/HSL/HSV互转</div>
  </a>
  <a class="tool-card" href="/tools/image-base64.html">
    <div class="tool-icon">🔗</div>
    <div class="tool-title">图片转 Base64</div>
    <div class="tool-desc">图片转DataURL，支持拖拽</div>
  </a>
  <a class="tool-card" href="/tools/base64-image.html">
    <div class="tool-icon">🖼️</div>
    <div class="tool-title">Base64 转图片</div>
    <div class="tool-desc">Base64还原为图片并下载</div>
  </a>
  <a class="tool-card" href="/tools/image-resize.html">
    <div class="tool-icon">📐</div>
    <div class="tool-title">图片尺寸修改</div>
    <div class="tool-desc">按百分比或自定义宽高缩放</div>
  </a>
  <a class="tool-card" href="/tools/image-convert.html">
    <div class="tool-icon">🔄</div>
    <div class="tool-title">图片格式转换</div>
    <div class="tool-desc">PNG/JPEG/WebP 互转</div>
  </a>
  <a class="tool-card" href="/tools/uuid.html">
    <div class="tool-icon">🆔</div>
    <div class="tool-title">UUID 生成器</div>
    <div class="tool-desc">批量生成UUID v4</div>
  </a>
  <a class="tool-card" href="/tools/password.html">
    <div class="tool-icon">🔑</div>
    <div class="tool-title">密码生成器</div>
    <div class="tool-desc">自定义长度和字符集</div>
  </a>
  <a class="tool-card" href="/tools/char-count.html">
    <div class="tool-icon">🔢</div>
    <div class="tool-title">字符统计</div>
    <div class="tool-desc">字数/行数/字节数统计</div>
  </a>
  <a class="tool-card" href="/tools/unit-convert.html">
    <div class="tool-icon">📏</div>
    <div class="tool-title">单位换算</div>
    <div class="tool-desc">长度/重量/温度/数据存储</div>
  </a>
  <a class="tool-card" href="/tools/http-status.html">
    <div class="tool-icon">🌐</div>
    <div class="tool-title">HTTP 状态码</div>
    <div class="tool-desc">查询状态码含义和原因</div>
  </a>
  <a class="tool-card" href="/tools/csv-json.html">
    <div class="tool-icon">💱</div>
    <div class="tool-title">CSV/JSON 互转</div>
    <div class="tool-desc">CSV与JSON双向转换</div>
  </a>
  <a class="tool-card" href="/tools/cron.html">
    <div class="tool-icon">⏲️</div>
    <div class="tool-title">CRON 表达式</div>
    <div class="tool-desc">可视化生成定时任务表达式</div>
  </a>
</div>

<style>
.tool-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 20px;
  margin-top: 30px;
}
.tool-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 30px 20px;
  background: var(--card-bg, #fff);
  border-radius: 12px;
  text-decoration: none;
  color: var(--font-color, #333);
  transition: all 0.3s ease;
  box-shadow: 0 2px 12px rgba(0,0,0,0.08);
  border: 1px solid var(--border-color, #eee);
}
.tool-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 24px rgba(0,0,0,0.12);
}
.tool-icon {
  font-size: 48px;
  margin-bottom: 12px;
}
.tool-title {
  font-size: 18px;
  font-weight: 600;
  margin-bottom: 8px;
}
.tool-desc {
  font-size: 14px;
  color: var(--font-color-sub, #999);
  text-align: center;
}
</style>

{% endraw %}
