---
tags: [GalNavi, 数据, GitHub, 仓库]
created: 2026-07-08
updated: 2026-07-08
aliases: [仓库角色, GitHub仓库, argb6/gal-navigation]
---

# GitHub 仓库的角色

> [!important] 关键认知
> 仓库 `argb6/gal-navigation` **不存放网站源代码**，只存放**数据与素材资源**。这是一个容易被误解的点——它不是常规的"代码仓库"，而是 GalNavi 的"数据仓库 + 静态资源 CDN + 项目说明"。

## 仓库地址

[github.com/argb6/gal-navigation](https://github.com/argb6/gal-navigation)

## 仓库实际内容（git ls-files 核实）

### 非图片文件（仅 3 个）
| 文件 | 作用 |
|---|---|
| `README.md` | 项目说明（含截图、框架图、联系方式）|
| `LICENSE` | MIT 协议 |
| `docs/data.json` | 站点数据（D1 滞后镜像，仓库 26 条 vs D1 29 条）|

### 图片资源
- `assets/` — icon / logo / favicon / old
- `docs/hero/` — 3 张轮播图（被线上 KV 引用）
- `docs/mascot/` — 5 张吉祥物
- `docs/emoji/` — 约 40 个表情包
- `docs/*.png` — 入口/主站/详情截图（README 用）

### 没有的东西（重要）
- ❌ 没有 `package.json`、`wrangler.toml`
- ❌ 没有 `.js` / `.ts` / `.html` / `.css` 源文件
- ❌ 没有 Worker 代码、没有构建配置
- ❌ 没有测试文件

> 仓库无任何代码文件，**网站源代码不公开**，仅部署在 Cloudflare 上。

## 仓库的三大角色

### 角色 1：数据滞后镜像
- `docs/data.json` 是线上 D1（`/nav/api/nav`）的**滞后镜像**，字段结构一致但条目数可能不同步
- 实测：D1 有 29 条，仓库 data.json 仅 26 条（少 琉璃神社、绅士之庭、萌心次元 3 条网站）
- 推测工作流：站长改 D1 → 定期导出 data.json 提交仓库，存在滞后窗口
- 仓库让数据**可追溯、可 diff、可社区贡献**，但**非实时权威源**

### 角色 2：静态资源 CDN
- `docs/hero/hero*.png` 被线上 KV 引用（[轮播图脚本（Hero Carousel）](../03-部署的JS/轮播图脚本（Hero Carousel）.md)）
- 通过 `raw.githubusercontent.com` 访问，免费 CDN
- 详见 [图片素材资源](图片素材资源.md)

### 角色 3：项目门面与社区入口
- `README.md` 是项目的公开说明书
- Issues 是提交新站点/反馈的渠道（[开源与社区](../01-项目总览/开源与社区.md)）
- README 的 badges（license、discord、stars、issues）展示项目状态

## 仓库与线上的对应关系

```mermaid
graph LR
    subgraph GitHub 仓库
        R[README.md]
        L[LICENSE MIT]
        D[docs/data.json]
        H[docs/hero/*.png]
        M[docs/mascot/]
        E[docs/emoji/]
        A[assets/]
    end

    subgraph 线上 galnavi.top
        W[Worker SSR]
        D1[(D1 数据库)]
        KV[(KV)]
        P1[入口页]
        P2[主站]
        P3[帮助页]
    end

    D -.->|滞后镜像 26条 vs 29条| D1
    H -->|raw URL 引用| KV
    KV -->|hero 图片| P2
    D1 -->|/nav/api/nav| P2
    R -->|说明| 用户
    L -->|协议| 用户
```

## git 历史

- 截至 2026-07-08 共 **52 次提交**
- 近期提交主要是**清理 emoji 图片**（多次 Delete *.png/*.jpg）
- 说明项目在整理资源，可能精简表情包或迁移素材
- 提交信息较简单（"Add files via upload"、"Delete xxx.png"），无规范化 commit 规范

## 为何不公开源码？（推测）

1. **维护成本**：开源代码需处理 PR、Issue、文档，个人项目可能无力维护
2. **安全考虑**：Worker 代码含 D1/KV 绑定逻辑，公开可能暴露接口细节
3. **数据先行**：项目核心价值是数据（站点收录），代码是载体
4. **MIT 协议的适用**：MIT 协议主要约束仓库内容（数据/素材可自由使用），网站代码本身未开源不影响

## 对知识库的影响

正因为仓库无源码，本知识库对"部署的 JS"的分析全部来自**线上网站 galnavi.top 的实际渲染产物**（HTML 中内联的脚本），而非仓库源码。这也解释了为何 JS 是"内联部署"——便于在 Worker 中以字符串拼接方式生成，无需单独托管 JS 文件。

## 相关笔记

- 数据 → [data.json 数据结构](data.json 数据结构.md)
- 素材 → [图片素材资源](图片素材资源.md)
- 架构 → [整体技术架构](../02-网站架构/整体技术架构.md)
- 开源 → [开源与社区](../01-项目总览/开源与社区.md)
- 上一级 → [00 知识库地图 (MOC)](../00 知识库地图 (MOC).md)
