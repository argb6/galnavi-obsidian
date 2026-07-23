---
tags: [GalNavi, JS, 安全, XSS, escapeHtml]
created: 2026-07-08
updated: 2026-07-24
aliases: [escapeHtml, XSS防护, 输入转义, isSafeHttpUrl]
---

# XSS 防护与 escapeHtml

> [!info] 位置
>
> `escapeHtml` 与 `isSafeHttpUrl` 定义在主站**数据预加载脚本**最前，供主应用渲染与轮播 URL 校验使用。

## 为什么需要

卡片标题、简介、标签、图标 URL 均来自 D1。直接拼进 HTML 可能产生存储型 XSS，故渲染前统一转义。

## escapeHtml

```javascript
function escapeHtml(str) {
  if (!str) return '';
  return String(str)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#39;');
}
```

| 原字符 | 转义为 |
|---|---|
| `&` | `&amp;`（必须最先）|
| `<` / `>` | `&lt;` / `&gt;` |
| `"` / `'` | `&quot;` / `&#39;` |

配套：`escapeRegExp`（搜索高亮）。

## isSafeHttpUrl

```javascript
function isSafeHttpUrl(url) {
  if (!url || typeof url !== 'string') return false;
  try {
    var u = new URL(url, window.location.origin);
    return u.protocol === 'http:' || u.protocol === 'https:';
  } catch (e) {
    return false;
  }
}
```

轮播图等 URL 会过滤，只保留 `http:` / `https:`。

## buildCard 调用

动态值均经 `escapeHtml`：`cat`、`tags`、`icon`、`name`、`desc`、`url`。搜索高亮：**先转义再插入标记**，避免破坏转义结果。

## 相关笔记

- 上一级 → [[部署的JS]]
- 相关 → [[数据预加载脚本（D1载入）]]
