---
tags: [GalNavi, 安全, CSP, 内容安全策略]
created: 2026-07-08
updated: 2026-07-09
aliases: [CSP, Content-Security-Policy, 内容安全策略]
---

# 内容安全策略 CSP

> ！位置
> 
> **仅主站** `/nav/`的 `<meta http-equiv="Content-Security-Policy">`。其他 worker **未设置 CSP**。

## 完整 CSP 策略（主站）

```http
Content-Security-Policy:
  default-src 'self';
  script-src 'self' 'unsafe-inline'
             https://lf26-cdn-tos.bytecdntp.com
             https://static.cloudflareinsights.com;
  style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
  img-src 'self' data: https:;
  connect-src 'self' https://galnavi.top;
  font-src 'self' data: https://fonts.gstatic.com;
```

## 逐指令解析

### `default-src 'self'`
- 默认所有资源只允许同源
- 是其他指令未覆盖时的兜底

### `script-src 'self' 'unsafe-inline' https://lf26-cdn-tos.bytecdntp.com https://static.cloudflareinsights.com`
| 源 | 用途 | 实际使用 |
|---|---|---|
| `'self'` | 同源脚本 | 当前未用外部脚本文件 |
| `'unsafe-inline'` | **内联脚本** | ✅ 必需，所有 JS 内联 |
| `https://lf26-cdn-tos.bytecdntp.com` | 字节跳动 CDN | ⚠️ 预留/历史，当前未加载 |
| `https://static.cloudflareinsights.com` | Cloudflare Web Analytics | ✅ 分析用 |

> `'unsafe-inline'` 是因全内联策略必须开的口子。GalNavi 用 [escapeHtml](../03-部署的JS/XSS 防护与 escapeHtml.md) 在渲染层弥补。

### `style-src 'self' 'unsafe-inline' https://fonts.googleapis.com`
- 允许内联样式（CSS 全内联，必需）
- 允许 Google Fonts 样式表：主站 CSP **放行**该域，但主站 HTML **并不** `<link>` 字体；实际加载 Noto Sans SC / Outfit 的是**圣器殿堂、详情页**（二者无此 CSP meta）。入口 / 关于 / 帮助以系统中文字体为主

### `font-src 'self' data: https://fonts.gstatic.com`
- 主站 CSP 预留的 Google Fonts 字体文件域；当前主站未引用，主要对应殿堂 / 详情页的字体实践

### `img-src 'self' data: https:`
- 同源图片、`data:` 内联 base64（favicon）、**任何 https 图片源**
- `https:` 较宽松，因站点图标来自各站（bing、各站自身、第三方图标库），无法预知所有域

### `connect-src 'self' https://galnavi.top` —— 最关键
- **fetch/XHR/WebSocket 只允许同源**
- 客户端 JS 只能 fetch `galnavi.top` 自身的 API（`/nav/api/*`）
- **不能** fetch 第三方域（防止数据外泄到外部服务器）
- 这也解释了为何所有 API 都在 `/nav/api/` 下

## CSP 仅作用于主站

| 页面 | CSP | 说明 |
|---|---|---|
| `/nav/`（主站）| ✅ 有 | 完整 CSP meta |
| `/nav/detail/`（详情页）| ❌ 无 | 未设 CSP；会拉 Google Fonts |
| `/nav/group/`（圣器殿堂）| ❌ 无 | 未设 CSP；会拉 Google Fonts |
| `/nav/about/`（关于页）| ❌ 无 | 未设 CSP meta |
| `/nav/help/`（帮助页）| ❌ 无 | 未设 CSP meta |
| `galnavi.top`（入口页）| ❌ 无 | 未设 CSP meta |

> 注：多数子页为 SSR 直出，外链已用 `rel="noopener noreferrer"`；理想情况下各页都应设 CSP。

## CSP 防护效果

| 攻击 | CSP 防护 |
|---|---|
| 外部脚本注入 | ✅ script-src 白名单阻止 |
| 内联脚本 XSS | ⚠️ unsafe-inline 允许，靠 escapeHtml 弥补 |
| 数据外泄 | ✅ connect-src 限制只回同源 |
| 图片滥用 | ⚠️ img-src https: 较宽 |
| 样式注入 | ⚠️ unsafe-inline，但影响小 |

## 与 API CORS 的配合

API 端点（`/nav/api/*`）在响应头中设置 CORS（源码确认）：
```http
Access-Control-Allow-Origin: https://galnavi.top
```
- CSP 的 `connect-src` 限制**客户端能发请求到哪里**
- CORS 的 `Access-Control-Allow-Origin` 限制**哪些源能读取响应**
- 两者配合：只有 galnavi.top 自身的页面能调这些 API，形成双重约束
- featured 端点额外支持 OPTIONS 预检（`GET, POST, OPTIONS`）

## 相关笔记

- XSS 防护 → [XSS 防护与 escapeHtml.md](XSS防护与escapeHtml.md)
- 安全响应头（含 CORS/Cache-Control）→ [安全响应头](安全响应头.md)
- 外链安全 → [外链安全设计](外链安全设计.md)
- 内联策略 → [内联 JS 总览与加载策略.md](内联JS总览与加载策略.md)
- 上一级 → [00 知识库地图 (MOC).md](00知识库地图(MOC).md)
