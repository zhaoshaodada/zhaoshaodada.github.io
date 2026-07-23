---
title: 在线工具
date: 2026-07-23 19:30:00
type: "tools"
top_img: /img/tools.jpg
comments: true
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
