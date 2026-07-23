---
tags: [GalNavi, SEO, 可发现性, sitemap, OG]
created: 2026-07-08
updated: 2026-07-09
aliases: [SEO, 搜索引擎优化, sitemap, robots, OG标签]
---

# SEO 与可发现性

> ！概述
> 
> GalNavi 在 SEO 上做了系统化配置：sitemap、robots、meta 标签、Open Graph、canonical。让搜索引擎与社交平台能正确理解与展示站点。**入口页 SEO 配置最完整**，主站次之。

## SEO 资产清单

| 资产 | 入口页 | 主站 | 其他页 |
|---|---|---|---|
| robots meta | ✅ | ✅ `index,follow` | — |
| canonical | ✅ | ✅ `/nav/` | — |
| description/keywords | ✅ | — | — |
| Open Graph | ✅ | — | — |
| Twitter Card | ✅ | — | — |
| sitemap 收录 | ✅ priority 1.0 | ✅ priority 1.0 | help priority 0.8 |

## robots.txt

```
User-agent: *
Allow: /
Sitemap: https://galnavi.top/sitemap.xml
```
- **完全开放**：允许所有爬虫抓取所有路径
- 声明 sitemap 位置
- 无 Disallow，说明无隐藏内容

## sitemap.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
<url>
<loc>https://galnavi.top/</loc>
<priority>1.0</priority>
</url>
<url>
<loc>https://galnavi.top/nav/</loc>
<priority>1.0</priority>
</url>
<url>
<loc>https://galnavi.top/nav/detail/</loc>
<priority>0.8</priority>
</url>
<url>
<loc>https://galnavi.top/nav/about/</loc>
<priority>0.8</priority>
</url>
<url>
<loc>https://galnavi.top/nav/group/</loc>
<priority>0.8</priority>
</url>
<url>
<loc>https://galnavi.top/nav/help/</loc>
<priority>0.8</priority>
</url>
</urlset>
```

收录策略：
- `/` 与 `/nav/`：priority 1.0
- `/nav/detail/`、`/nav/about/`、`/nav/group/`、`/nav/help/`：priority 0.8
- **不逐条收录**：`/nav/detail/?item_key=xxx`、API 端点

> 关于页 `/nav/about/` 已正式上线并写入 sitemap（不再是维护占位页）。

## 各页 meta 标签

### 入口页 `/`（SEO 最完整）
```html
<title>GALNAVI 永久发布页 - 最新版本与更新日志</title>
<meta name="description" content="GALNAVI 官方永久发布页，提供最新版本下载、更新日志与项目动态。纯净无广，一站直达。">
<meta name="keywords" content="GALNAVI, 发布页, 下载, 更新日志, 二次元导航, ACG, 资源聚合">
<link rel="canonical" href="https://galnavi.top/">
<link rel="sitemap" type="application/xml" title="Sitemap" href="https://galnavi.top/sitemap.xml">
```
> 入口页标题已从早期的"ACG 二次元资源聚合导航"改为"永久发布页 - 最新版本与更新日志"，聚焦发布页定位。

### 主站 `/nav/`
```html
<title>GALNAVI</title>
<meta name="color-scheme" content="dark">
<meta name="robots" content="index, follow">
<link rel="canonical" href="https://galnavi.top/nav/">
```
> 主站 title 简短（应用页），但明确设了 `robots: index,follow` 和 canonical `/nav/`，确保被正确收录并集中权重。

## Open Graph（社交分享）

```html
<meta property="og:type" content="website">
<meta property="og:title" content="GALNAVI - ACG 二次元资源聚合导航">
<meta property="og:description" content="一个专注于 ACG 二次元资源网站聚合与收录的纯净导航站点。纯净无广，秒速响应，一站直达。">
<meta property="og:url" content="https://galnavi.top/">
<meta property="og:site_name" content="GALNAVI">
<meta property="og:image" content="https://galnavi.top/favicon.ico">
```
- 分享到社交平台（微信/Facebook/Twitter 等）时展示的卡片信息
- og:image 用 favicon（较小，可改进为用仓库 icon.png）

## Twitter Card

```html
<meta name="twitter:card" content="summary">
<meta name="twitter:title" content="GALNAVI - ACG 二次元资源聚合导航">
<meta name="twitter:description" content="...">
<meta name="twitter:image" content="https://raw.githubusercontent.com/argb6/gal-navigation/main/assets/icon.png">
```
- `summary` 类型（小图卡片）
- twitter:image 指向**仓库的 icon.png**（GitHub raw），与 og:image 不同

## canonical

- 入口页：`<link rel="canonical" href="https://galnavi.top/">`
- 防止重复 URL（如带参数、http/https）被当作不同页面，集中权重

## 关键词策略

入口页 keywords：`GALNAVI, ACG, 二次元导航, 动漫资源, 漫画网站, 游戏社区, GalGame, 视觉小说, 资源导航`
- 覆盖品牌词（GALNAVI）+ 行业词（ACG/二次元导航）+ 内容词（Galgame/视觉小说）
- 中文为主，符合目标用户搜索习惯

## 可发现性设计

1. **SSR 直出**：HTML 含完整内容，搜索引擎可直接抓取（非纯 CSR）
2. **sitemap + robots**：标准引导
3. **语义化**：虽然内联 CSS/JS，但 HTML 结构语义清晰
4. **深链接**：`/nav/detail/?item_key=` 可被分享与索引（虽不在 sitemap）
5. **canonical**：防重复

## 可改进点

| 点 | 建议 |
|---|---|
| og:image | 改用更高清的仓库 icon.png 而非 favicon |
| 结构化数据 | 可加 JSON-LD（如 `WebSite` + `ItemList`），当前未发现 |
| 详情页 | 可考虑加 `<meta name="robots" content="noindex">` 防止收录变动频繁的网盘页 |
| HSTS | 见 [安全响应头](安全响应头.md)，max-age=0 待修正 |

## 相关笔记

- 上一级 → [[安全与SEO]]
