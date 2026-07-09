---
tags: [GalNavi, 架构, D1, KV, Cloudflare, 存储]
created: 2026-07-08
updated: 2026-07-09
aliases: [存储层, D1数据库, KV存储, 数据库设计]
---

# 存储层 D1 与 KV

> [!summary] 分工原则
> GalNavi 用 **D1** 存"结构化、需查询的核心业务数据"（站点列表），用 **KV** 存"低频变更、整存整取的配置型数据"（轮播图、推荐）。这是 Cloudflare 生态里典型的"关系数据 vs 配置数据"分工。

## D1（Cloudflare SQLite 数据库）

### 定位
存放站点核心数据，是卡片渲染的数据源。

### 对应端点
[`/nav/api/nav`](API 端点清单.md)

### 表结构
D1 中有一张业务表，存放站点数据。字段如下：

| 列 | 类型 | 说明 | 查询处 |
|---|---|---|---|
| `item_key` | TEXT PRIMARY KEY | 唯一键，如 `joiplay`、`灵梦御所` | 主站 + 详情 |
| `title` | TEXT | 标题 | 主站 + 详情 |
| `category` | TEXT | `simulators`/`websites`/`tools`/`company`/`hanhua` | 主站 |
| `tags` | TEXT | 逗号分隔标签 | 主站 + 详情 |
| `short_desc` | TEXT | 简介 | 主站 + 详情 |
| `url` | TEXT | 主站链接（可空）| 主站 |
| `icon_path` | TEXT | 图标 URL | 主站 |
| `md_content` | TEXT | **详情页 Markdown 内容**（分支/网盘/密码等）| 详情页 |

> 注：主站 API 只 SELECT 前 7 个字段，`md_content` 仅详情页查询。这就是为何 [详情页](../05-页面详解/详情与外链跳转.md) 能展示网盘链接/密码等 data.json 中没有的扩展信息。

### 数据规模
- 当前 29 条（7 模拟器 + 22 网站）
- 与仓库 `docs/data.json` 字段结构一致但条目滞后（D1 29 条 vs 仓库 26 条）→ data.json 是 D1 的滞后导出/镜像

### 为什么用 D1 而非 KV
- 需要**按 category 过滤**、未来可能按标签查询 → 关系型查询
- 数据有结构（固定字段）→ 表结构合适
- 可用 SQL 维护（增删改查）

## KV（Cloudflare Workers KV）

### 定位
存放配置型、低频变更、整存整取的数据。

### 两个独立 KV 绑定

GalNavi 使用**两个独立的 KV namespace 绑定**，而非单一 KV：

#### KV 绑定 1（轮播图）
- **对应端点**：`/nav/api/hero`
- **存储内容**：图片 URL（JSON 数组字符串或逗号分隔字符串，Worker 兼容解析）
- **值示例**：`["https://raw.githubusercontent.com/.../hero1.png",...]` 或逗号分隔串
- **特点**：3 张图，变更频率极低（换通知 banner 时才改）
- **容错**：源码对 KV 值做多重解析——先 `JSON.parse`，失败则按逗号分割，再失败当作单元素数组

#### KV 绑定 2（站长推荐）
- **对应端点**：`/nav/api/featured`
- **存储内容**：推荐站点的 item_key 数组（JSON 字符串）
- **值示例**：`["灵梦御所","touchgal"]`
- **特点**：站长手动调整推荐时才变；支持 GET 读取 + POST 写入

### 为什么用 KV 而非 D1
- 整存整取，**无需查询/过滤**
- 读取频繁但变更极少 → KV 全球边缘缓存收益大
- 结构简单（一个 JSON 字符串即可）
- 拆成两个独立 namespace：隔离不同配置的读写权限与缓存策略

### KV 的自愈机制
featured 有有趣的"客户端回写"逻辑：
1. 客户端先 GET `/nav/api/featured`
2. 若 KV 为空 → 用标签匹配算法本地计算推荐
3. 计算后 **POST 回 `/nav/api/featured`**（Worker 写入对应 KV 绑定）写入 KV
4. 后续请求即可直接命中 KV

> 这是一种"惰性初始化 + 边缘缓存预热"模式，避免站长手动配置也能生效，配好后又被 KV 接管。

## D1 vs KV 对比

| 维度 | D1 | KV |
|---|---|---|
| 模型 | 关系型（SQLite）| 键值对 |
| 查询 | SQL，支持过滤/排序 | 按 key 整取 |
| 一致性 | 强一致 | 最终一致（边缘缓存）|
| 适合 | 结构化业务数据 | 配置/缓存 |
| GalNavi 用途 | 站点列表（业务表）| 轮播图、推荐（两个 KV namespace）|
| 读取延迟 | 略高（需查询）| 极低（边缘缓存）|

## 绑定方式

Cloudflare Workers 中通过 `wrangler.toml` 绑定 D1 与两个 KV namespace，示例如下：
```toml
[​[d1_databases]]
binding = "<D1绑定名>"
database_name = "<数据库名>"

[​[kv_namespaces]]
binding = "<轮播图KV绑定名>"
id = "<namespace ID>"

[​[kv_namespaces]]
binding = "<推荐KV绑定名>"
id = "<namespace ID>"
```

Worker 代码中的调用示意：
```javascript
// D1 查询（主站）：列出所有站点
const { results } = await env.<D1绑定>.prepare(
  "SELECT item_key, title, category, tags, short_desc, url, icon_path FROM <表名> ORDER BY category ASC"
).all();

// D1 查询（详情页，参数化绑定防注入）
const stmt = env.<D1绑定>.prepare(
  "SELECT title, short_desc, tags, md_content FROM <表名> WHERE item_key = ?"
);
const row = await stmt.bind(itemKey).first();

// KV 读取（轮播图）
const heroJson = await env.<轮播图KV绑定>.get('<轮播图键名>');

// KV 读写（推荐）
const raw = await env.<推荐KV绑定>.get('<推荐键名>');
await env.<推荐KV绑定>.put('<推荐键名>', JSON.stringify(keys));
```

> 注：安全要点：所有 SQL 均用**参数化绑定**（`.bind(itemKey)`）防注入，KV 仅存配置型数据。

## 相关笔记

- API 端点 → [API 端点清单](API 端点清单.md)
- 数据结构 → [data.json 数据结构](../04-数据与资源/data.json 数据结构.md)
- 整体架构 → [整体技术架构](整体技术架构.md)
- 上一级 → [00 知识库地图 (MOC)](../00 知识库地图 (MOC).md)
