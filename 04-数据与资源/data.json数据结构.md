---
tags:
  - GalNavi
  - 数据
  - D1
created: 2026-07-08
updated: 2026-07-08
aliases:
  - data.json
  - 数据结构
  - 站点数据
---

# data.json 数据结构

> ！概述
> 
> `docs/data.json` 是 GitHub 仓库中的数据文件，是线上 D1 数据库（[/nav/api/nav](../02-网站架构/API端点清单.md)）的**滞后镜像**。
> - 线上 D1（权威）：截至 2026-07-08 共 **29 条**（simulators 7 + websites 22）
> - 仓库 data.json（镜像）
## 文件位置

- 仓库：[docs/data.json](https://github.com/argb6/gal-navigation/blob/main/docs/data.json)（8147 字节）
- 线上：`GET https://galnavi.top/nav/api/nav`（9231 字节）

> ⚠️ 两者**不完全同步**：D1 是权威源，仓库 data.json 是定期导出的滞后副本。详见 [GitHub 仓库的角色](GitHub仓库的角色.md) 的 角色 1：数据滞后镜像。

## 数据格式

顶层是一个 **JSON 数组**，每个元素是一条站点/模拟器记录：

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

## 字段定义

| 字段 | 类型 | 必填 | 说明 | 示例 |
|---|---|---|---|---|
| `item_key` | string | ✅ | 唯一标识，作 D1 主键，方便引用它 | `"joiplay"` / `"灵梦御所"` |
| `title` | string | ✅ | 名称 | `"Joiplay"` |
| `category` | string | ✅ | 分类 | `simulators` / `websites` |
| `tags` | string | ✅ | 逗号分隔的标签串 | `"安卓,rpg,多功能,插件化"` |
| `short_desc` | string | ✅ | 一句话简介 | `"独特的插件生态"` |
| `url` | string | ✅ | 主站链接 | `""` 或 `"https://..."` |
| `icon_path` | string | ✅ | 图标 URL | `"https://joiplay.net/.../icon.png"` |

## 字段细节

### item_key
- 唯一主键
- 可用英文（`joiplay`、`kirikiroid2`）或中文（`盖世游戏`、`灵梦御所`）
- 被 [/nav/api/featured](../02-网站架构/API端点清单.md) 的 `item_keys` 引用
- 用于详情页 URL：`/nav/detail/?item_key=xxx`

### category（5 类，复数）
| category | 含义 | 前端映射（单数）|
|---|---|---|---|
| `simulators` | 模拟器 | `simulator` |
| `websites` | 站点 | `site` |
| `tools` | 工具 | `tool` |
| `company` | 会社 | `company` |
| `hanhua` | 汉化组 | `hanhua` |

> 当前只有 `simulators` 和 `websites` 有数据，`tools`/`company`/`hanhua` 是规划中的分类（导航栏已预留视图，见 [路由与页面体系](../02-网站架构/路由与页面体系.md)）。

映射关系见 [数据预加载脚本（D1 载入）](../03-部署的JS/数据预加载脚本（D1载入）.md) 的 `CAT_MAP`。

### tags（逗号分隔字符串）
- 存储为字符串：`"安卓,rpg,多功能,插件化"`
- 前端 `.split(',')` 转数组
- 一个条目平均 4–10 个标签
- 详见 [标签体系](标签体系.md)

### url
- 模拟器类（7 条）**url 为空**（`""`），因为模拟器无单一官网或需多链接
- 网站类（22 条）有 url，指向各站主域
- 详情页 `/nav/detail/` 会展示更多链接（官网、网盘等）

### icon_path
- 多为各站 favicon 或图标 URL
- 部分用 Bing 的 ODF 缩略图服务（`bing.com/th/id/ODF...`）
- 部分用站点自身图标
- 图标加载失败时显示默认 🔗（见 [主应用逻辑脚本（卡片与交互）](../03-部署的JS/主应用逻辑脚本（卡片与交互）.md) 的 buildCard）

## 数据来源与维护

- 仓库 `docs/data.json` 是**静态副本**（可被 GitHub raw 访问，也被 hero 图片引用同一仓库）
- 线上 D1 是**权威数据源**，Worker 从 D1 查询返回
- 二者**不完全同步**：D1（29 条）比仓库 data.json（26 条）多 3 条网站（琉璃神社、绅士之庭、萌心次元）
- 推测工作流：站长更新 D1 后定期导出到仓库 data.json，存在滞后窗口
- 详见 [GitHub 仓库的角色](GitHub仓库的角色.md)

## 相关笔记

- 标签语义 → [标签体系](标签体系.md)
- 数据加载 JS → [数据预加载脚本（D1 载入）](../03-部署的JS/数据预加载脚本（D1载入）.md)
- 存储 → [存储层 D1 与 KV](../02-网站架构/存储层D1与KV.md)
- 上一级 → [00 知识库地图 (MOC)](../00知识库地图(MOC).md)
