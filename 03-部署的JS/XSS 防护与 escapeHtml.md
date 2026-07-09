---
tags: [GalNavi, JS, 安全, XSS, escapeHtml]
created: 2026-07-08
updated: 2026-07-08
aliases: [escapeHtml, XSS防护, 输入转义]
---

# XSS 防护与 escapeHtml

> [!info] 位置
> `escapeHtml` 定义在主站 `/nav/` 行1 的数据预加载脚本中（最早可用），供主应用脚本（行948）的 `buildCard` 等渲染函数调用。

## 为什么需要

GalNavi 的卡片内容（标题、简介、标签、图标 URL）**全部来自 D1 数据库**，而这些数据：
- 部分由站长手动维护
- 未来可能开放社区提交（Issue/PR）
- title 含中文、特殊字符

若直接拼接到 HTML，恶意或意外的 `<script>` 等内容会被执行（存储型 XSS）。GalNavi 用 `escapeHtml` 在**渲染前**对所有动态文本转义。

## 完整源码

```javascript
// ==================== HTML 转义函数（防 XSS） ====================
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

配套的还有正则转义函数：
```javascript
function escapeRegExp(str) {
    return str.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
}
```

## 转义规则

| 原字符 | 转义为 | 原因 |
|---|---|---|
| `&` | `&amp;` | 必须最先转义，否则会破坏后续实体 |
| `<` | `&lt;` | 防止注入标签 |
| `>` | `&gt;` | 防止闭合标签 |
| `"` | `&quot;` | 防止逃逸属性值（双引号）|
| `'` | `&#39;` | 防止逃逸属性值（单引号）|

**要点**：`&` 必须第一个转义，否则后面转义产生的 `&amp;` 中的 `&` 会被再次转义成 `&amp;amp;`。

## 调用点（buildCard 中）

```javascript
function buildCard(item, keyword) {
    let cat = escapeHtml(item.cat || '');
    // 标签
    .map(t => `<span class="tag-badge" data-tag="${escapeHtml(t)}">${escapeHtml(t)}</span>`)
    // 图标 URL
    ico = `<img src="${escapeHtml(iconUrl)}" ...>`;
    // 名称
    const escapedName = escapeHtml(item.name || '');
    // 简介
    '<div class="card-subtitle">' + escapeHtml(item.desc || '') + '</div>'
    // 外链 URL
    '<a href="' + escapeHtml(item.url) + '" ...>'
}
```

**每个动态值都被转义**：
- `item.cat` → data-cat 属性
- `item.tags` → data-tag 属性 + 文本内容
- `item.icon` → img src 属性
- `item.name` → 文本内容（搜索高亮前先转义）
- `item.desc` → 文本内容
- `item.url` → a href 属性

## 搜索高亮的安全处理

```javascript
const escapedName = escapeHtml(item.name || '');        // 1. 先转义
const escapedKeyword = keyword ? escapeRegExp(keyword) : '';
const nameDisplay = keyword
    ? escapedName.replace(new RegExp(`(${escapedKeyword})`, 'gi'),
        '<span class="search-highlight">$1</span>')     // 2. 再包裹高亮 span
    : escapedName;
```

**顺序很重要**：
1. **先** `escapeHtml` 转义名称（此时是纯文本，无 HTML）
2. **再** 用正则把关键词替换为 `<span class="search-highlight">$1</span>`

> 注意：用户搜索的关键词本身不会被转义后注入，而是作为正则匹配 `escapedName`（已转义）中的对应文本。关键词中的特殊字符用 `escapeRegExp` 处理避免正则注入。这样高亮 span 是代码可控的，用户输入不会产生意外标签。

## escapeRegExp 的作用

```javascript
function escapeRegExp(str) {
    return str.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
}
```
转义正则元字符，防止用户搜索 `.*` 等时被当作正则语法。例如搜 `a.b` 不会匹配 `aXb`，只匹配字面 `a.b`。

## 防护层次总结

GalNavi 的 XSS 防护是多层的：

| 层 | 措施 | 笔记 |
|---|---|---|
| 输入转义 | `escapeHtml` 所有动态值 | 本笔记 |
| 正则安全 | `escapeRegExp` 用户搜索词 | 本笔记 |
| CSP | 严格的 script-src 白名单 | [内容安全策略 CSP](内容安全策略CSP.md) |
| 外链安全 | noopener noreferrer | [外链跳转脚本（Redirect 倒计时）](外链跳转脚本（Redirect倒计时）.md) |
| 内容类型 | nosniff 头 | [安全响应头](安全响应头.md) |

## 局限性

- `escapeHtml` 是手写实现，未用成熟库（如 DOMPurify）。对于纯文本场景足够，但若未来引入富文本（如站点介绍含格式）需改用 sanitize 方案。
- 当前 `item.url` 只转义 HTML 实体，未校验协议（如 `javascript:` 协议）。不过 buildCard 里 `hasUrl` 判断与全局拦截器排除了非 http，且详情页外链由服务端控制，风险较低。

## 相关笔记

- 调用方 → [主应用逻辑脚本（卡片与交互）](主应用逻辑脚本（卡片与交互）.md)
- 定义位置 → [数据预加载脚本（D1 载入）](数据预加载脚本（D1载入）.md)
- CSP → [内容安全策略 CSP](内容安全策略CSP.md)
- 总览 → [内联 JS 总览与加载策略](内联JS总览与加载策略.md)
- 上一级 → [00 知识库地图 (MOC)](00知识库地图(MOC).md)
