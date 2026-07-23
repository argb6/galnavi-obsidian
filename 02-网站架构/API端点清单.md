---
tags: [GalNavi, 架构, API, D1, KV]
created: 2026-07-08
updated: 2026-07-24
aliases: [API清单, 接口列表, 端点]
---

# API 端点清单

> [!info] 3 个真实 API
>
> 全部位于 `/nav/api/*`，返回 `application/json`，由主站客户端调用。

## 端点总览

| 端点 | 方法 | 存储 | 用途 |
|---|---|---|---|
| `/nav/api/nav` | GET | D1 | 站点列表 |
| `/nav/api/hero` | GET | KV | 轮播图 |
| `/nav/api/featured` | GET / POST / OPTIONS | KV | 站长推荐 |

> `/nav/group/`（「圣器殿堂」）不提供 JSON API：服务端直接查殿堂 D1 后 SSR。

## 1. `/nav/api/nav` —— 站点数据

### 请求
```
GET /nav/api/nav
```
客户端带 `cache: 'no-cache'` 与超时控制。

### 响应字段

| 字段 | 说明 |
|---|---|
| `item_key` | 唯一标识 |
| `title` | 显示名称 |
| `category` | `simulators` / `websites` / `tools` / `company` / `hanhua` |
| `tags` | 逗号分隔标签 |
| `short_desc` | 一句话简介 |
| `url` | 主站链接 |
| `icon_path` | 图标 URL |

### 客户端格式化

```javascript
const CAT_MAP = {
  simulators: 'simulator', websites: 'site',
  tools: 'tool', company: 'company', hanhua: 'hanhua'
};
// 转成 {id, cat, name, desc, tags[], url, icon}
```

相对路径图标会补全为 GitHub raw 基址。CORS 限定同源。

详见 「数据预加载脚本（D1载入）」、「data.json数据结构」。

## 2. `/nav/api/hero` —— 轮播图

返回 `{ images: string[], debug: string }`（客户端亦兼容纯数组）。Worker 对 KV 原始值做多重容错解析（JSON → 逗号分割 → 单元素）。图片多托管于 GitHub raw。客户端空数组兜底。

详见 「轮播图脚本（HeroCarousel）」。

## 3. `/nav/api/featured` —— 站长推荐

### GET 响应
```json
{ "item_keys": ["灵梦御所", "touchgal"] }
```

### 客户端
- KV 有数据：按 `item_keys` 顺序展示
- KV 为空：本地标签匹配 fallback 展示
- 写入接口仅供运维侧使用，浏览器端只读

详见 「主应用逻辑脚本（卡片与交互）」、[[存储层D1与KV]]。

## 圣器殿堂（非 API）

殿堂页服务端查询 `resources` 表并 SSR。失败时返回错误态。详见 「圣器殿堂」。

## 非真实端点（fallback）

以下路径可能返回入口页 HTML，**不是** JSON API：
- `/api/items`、`/api/data`、`/data.json` 等

探测时看 `Content-Type`：真实 API 为 `application/json`。

## 相关笔记

- 上一级 → [[网站架构]]
- [[存储层D1与KV]]
- [[数据预加载脚本（D1载入）]]
