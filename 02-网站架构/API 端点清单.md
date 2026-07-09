---
tags: [GalNavi, 架构, API, D1, KV]
created: 2026-07-08
updated: 2026-07-08
aliases: [API清单, 接口列表, 端点]
---

# API 端点清单

> [!summary] 实测确认的 3 个真实 API
> 全部位于 `/nav/api/*` 下，返回 `application/json`，由主站客户端 JS 调用。

## 端点总览

| 端点 | 方法 | 存储 | 用途 | 返回大小 |
|---|---|---|---|---|
| `/nav/api/nav` | GET | **D1** | 站点核心数据 | ~9.2 KB |
| `/nav/api/hero` | GET | **KV** | 轮播图图片列表 | ~463 B |
| `/nav/api/featured` | GET | **KV** | 站长推荐 item_keys | ~41 B |

## 1. `/nav/api/nav` —— D1 站点数据

**存储**：Cloudflare D1（SQLite）
**用途**：返回所有站点/模拟器条目，供卡片渲染

### 请求
```
GET /nav/api/nav
```
客户端调用时带 `cache: 'no-cache'` 与 10 秒 `AbortController` 超时。

### 响应结构
```json
[
  {
    "item_key": "joiplay",
    "title": "Joiplay",
    "category": "simulators",
    "tags": "安卓,rpg,多功能,插件化",
    "short_desc": "独特的插件生态",
    "url": "",
    "icon_path": "https://joiplay.net/assets/images/icon.png"
  },
  ...
]
```

### 字段说明
| 字段 | 类型 | 说明 |
|---|---|---|
| `item_key` | string | 唯一标识（也作 D1 主键，featured 引用它）|
| `title` | string | 显示名称 |
| `category` | string | `simulators` / `websites` / `tools` / `company` / `hanhua` |
| `tags` | string | 逗号分隔的标签串（前端 split 成数组）|
| `short_desc` | string | 一句话简介 |
| `url` | string | 主站链接（模拟器类为空）|
| `icon_path` | string | 图标 URL |

### 客户端格式化
前端拿到后做映射（见 [数据预加载脚本（D1 载入）](../03-部署的JS/数据预加载脚本（D1 载入）.md)）：
```javascript
const CAT_MAP = {
  simulators: 'simulator', websites: 'site',
  tools: 'tool', company: 'company', hanhua: 'hanhua'
};
// 转成 {id, cat, name, desc, tags[], url, icon}
```

> 该端点返回线上 D1 数据（29 条），与 GitHub 仓库 `docs/data.json`（26 条）**字段结构一致但条目滞后**——D1 比仓库多 3 条网站。说明 D1 是权威源，仓库 data.json 是滞后镜像。详见 [data.json 数据结构](../04-数据与资源/data.json 数据结构.md)。

### 源码实现细节（websearch.js）
- **SQL**：`SELECT item_key, title, category, tags, short_desc, url, icon_path FROM <表名> ORDER BY category ASC`（参数化，无注入风险）
- **图标路径补全**：`icon_path` 若以 `./` 开头，会拼接 GitHub raw 基址 `https://raw.githubusercontent.com/argb6/gal-navigation/main`
- **CORS**：`Access-Control-Allow-Origin: https://galnavi.top`（仅允许同源）
- **D1 绑定**：通过 `env.<D1绑定名>` 访问
- **错误处理**：D1 未配置返回 500，异常返回 500

## 2. `/nav/api/hero` —— KV 轮播图

**存储**：Cloudflare KV（轮播图专用 namespace）
**用途**：返回主站顶部轮播图图片 URL 列表

### 响应结构
```json
{
  "images": [
    "https://raw.githubusercontent.com/argb6/gal-navigation/main/docs/hero/hero1.png",
    "https://raw.githubusercontent.com/argb6/gal-navigation/main/docs/hero/hero2.png",
    "https://raw.githubusercontent.com/argb6/gal-navigation/main/docs/hero/hero3.png"
  ],
  "debug": "heroJson length=241, first 80 chars=..., | JSON parse error: ..."
}
```

### 注意点
- 当前响应含 `debug` 字段，说明 KV 中存的原始值是**纯字符串**（非 JSON 数组），Worker 尝试 `JSON.parse` 失败后回退处理。正式模式应返回纯数组。
- **Worker 容错解析**（源码确认）：先 `JSON.parse` → 失败则按逗号分割 → 再失败当作单元素数组，保证总返回 images 数组
- 图片指向 **GitHub raw**（`raw.githubusercontent.com/argb6/gal-navigation/main/docs/hero/`），验证了仓库作为静态资源 CDN 的角色。
- 客户端有 `HERO_FALLBACK = []` 兜底（当前为空数组，KV 失败时轮播为空）。

详见 [轮播图脚本（Hero Carousel）](../03-部署的JS/轮播图脚本（Hero Carousel）.md)。

## 3. `/nav/api/featured` —— KV 站长推荐

**存储**：Cloudflare KV（推荐专用 namespace）
**用途**：返回被推荐的站点 item_key 列表；支持 GET 读取 + POST 写入

### 响应结构
```json
{
  "item_keys": ["灵梦御所", "touchgal"]
}
```

### 客户端逻辑
- KV 优先：若 featured 有数据，按 item_keys 顺序展示
- Fallback：若 KV 为空，用标签匹配算法挑选推荐
- **同步回写 KV**：fallback 计算出推荐后，会 POST `/nav/api/featured`（Worker 写入推荐 KV 绑定）写入 KV，让后续请求直接命中（自愈机制）
- **CORS**：支持 OPTIONS 预检（`GET, POST, OPTIONS`），`Access-Control-Allow-Origin: https://galnavi.top`

详见 [主应用逻辑脚本（卡片与交互）](../03-部署的JS/主应用逻辑脚本（卡片与交互）.md) 的 featured 部分。

## 非真实端点（fallback）

以下路径虽返回 HTTP 200，但返回的是**入口页 HTML**（SPA fallback），**不是 JSON API**：
- `/api/items`、`/api/data`、`/api/sites`
- `/data.json`、`/go/`、`/entry`、`/main`、`/help`、`/about`

> 探测时务必看 `Content-Type`：真实 API 是 `application/json`，fallback 是 `text/html`。

## 调用关系图

```mermaid
sequenceDiagram
    participant B as 浏览器
    participant W as Cloudflare Worker
    participant D1 as D1 数据库
    participant KV as KV 存储

    B->>W: GET /nav/ (SSR HTML)
    W-->>B: HTML + 内联JS
    B->>W: GET /nav/api/nav
    W->>D1: SELECT * FROM items
    D1-->>W: 29 rows
    W-->>B: JSON 数组
    B->>W: GET /nav/api/hero
    W->>KV: get("hero")
    KV-->>W: 图片URL串
    W-->>B: {images, debug}
    B->>W: GET /nav/api/featured
    W->>KV: get("featured")
    KV-->>W: item_keys
    W-->>B: {item_keys}
```

## 相关笔记

- 存储分工 → [存储层 D1 与 KV](存储层 D1 与 KV.md)
- 调用方 JS → [数据预加载脚本（D1 载入）](../03-部署的JS/数据预加载脚本（D1 载入）.md)
- 数据结构 → [data.json 数据结构](../04-数据与资源/data.json 数据结构.md)
- 上一级 → [00 知识库地图 (MOC)](../00 知识库地图 (MOC).md)
