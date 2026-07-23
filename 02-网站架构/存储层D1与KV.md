---
tags: [GalNavi, 架构, D1, KV, Cloudflare, 存储]
created: 2026-07-08
updated: 2026-07-24
aliases: [存储层, D1数据库, KV存储, 数据库设计]
---

# 存储层 D1 与 KV

> [!info] 分工原则
>
> **D1** 存结构化、需查询的核心业务数据；**KV** 存低频变更、整存整取的配置数据。

## D1

### 主站库（站点导航）

用于 `/nav/`、`/nav/detail/`、`/nav/api/nav`。

| 列 | 说明 | 查询处 |
|---|---|---|
| `item_key` | 唯一键 | 主站 + 详情 |
| `title` | 标题 | 主站 + 详情 |
| `category` | 分类 | 主站 |
| `tags` | 标签 | 主站 + 详情 |
| `short_desc` | 简介 | 主站 + 详情 |
| `url` | 主站链接 | 主站 |
| `icon_path` | 图标 URL | 主站 |
| `md_content` | 详情 Markdown | 仅详情页 |

主站 API 只取前 7 列；`md_content` 仅详情页查询。

### 殿堂库（圣器殿堂）

用于 `/nav/group/`，表 `resources`，与主站库分离：

| 列 | 说明 |
|---|---|
| `id` | 主键 |
| `category` | 神器 / 魔器 / 仙器 |
| `name` | 名称 |
| `official_url` / `details_url` | 官网 / 详情 |
| `link1` ~ `link3` | 额外链接 |

```sql
SELECT * FROM resources ORDER BY category, id
```

详见 [[圣器殿堂]]。

### 为何用 D1
- 需按分类过滤，结构固定
- 可用 SQL 维护

## KV

### 用途

| 用途 | 对应端点 | 说明 |
|---|---|---|
| 轮播图 | `/nav/api/hero` | 图片 URL 列表；解析容错（JSON / 逗号分割）|
| 站长推荐 | `/nav/api/featured` | 推荐站点的 item_key 列表；GET 公开读 |

### 为何用 KV
- 整存整取，无需复杂查询
- 读多写少，适合边缘缓存
- 轮播与推荐分属不同配置空间，互不影响

### 推荐展示策略
1. 客户端 GET 推荐列表
2. 有数据则按序展示
3. 为空则本地标签匹配 fallback（仅前端展示，不写回）

## D1 vs KV

| 维度 | D1 | KV |
|---|---|---|
| 模型 | 关系型（SQLite）| 键值对 |
| 查询 | SQL 过滤/排序 | 按 key 整取 |
| 一致性 | 强一致 | 最终一致 |
| GalNavi | 站点列表、殿堂器物 | 轮播、推荐 |

SQL 使用参数化查询，避免注入。

## 相关笔记

- [[API端点清单]]
- [[data.json数据结构]]
- [[圣器殿堂]]
- [[整体技术架构]]
- [[00知识库地图(MOC)]]
