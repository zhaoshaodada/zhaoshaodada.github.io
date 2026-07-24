---
title: 小游戏
date: 2026-07-24 12:00:00
type: "games"
top_img: /img/tools.jpg
comments: true
description: "有趣的在线小游戏合集，包括迷宫逃脱等益智游戏，全部纯前端实现，无需下载即可游玩。"
keywords: "小游戏,迷宫游戏,益智游戏,在线游戏,迷宫逃脱"
---

## 🎮 小游戏合集

这里收集了一些有趣的在线小游戏，全部纯前端实现，打开即可游玩。

{% raw %}

<div class="game-grid">
  <a class="game-card" href="/games/maze.html">
    <div class="game-icon">🎮</div>
    <div class="game-title">迷宫逃脱</div>
    <div class="game-desc">使用方向键控制小人走出迷宫</div>
    <div class="game-tags">
      <span class="tag">益智</span>
      <span class="tag">迷宫</span>
    </div>
  </a>
</div>

<style>
.game-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 25px;
  margin-top: 30px;
}
.game-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40px 30px;
  background: var(--card-bg, #fff);
  border-radius: 16px;
  text-decoration: none;
  color: var(--font-color, #333);
  transition: all 0.3s ease;
  box-shadow: 0 4px 16px rgba(0,0,0,0.08);
  border: 1px solid var(--border-color, #eee);
}
.game-card:hover {
  transform: translateY(-6px);
  box-shadow: 0 12px 32px rgba(102, 126, 234, 0.15);
}
.game-icon {
  font-size: 64px;
  margin-bottom: 15px;
}
.game-title {
  font-size: 20px;
  font-weight: 600;
  margin-bottom: 10px;
}
.game-desc {
  font-size: 14px;
  color: var(--font-color-sub, #999);
  text-align: center;
  margin-bottom: 15px;
}
.game-tags {
  display: flex;
  gap: 8px;
}
.tag {
  padding: 4px 12px;
  background: #f0f0f0;
  border-radius: 20px;
  font-size: 12px;
  color: #666;
}
</style>

{% endraw %}