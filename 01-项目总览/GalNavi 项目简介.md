---
tags: [GalNavi, 项目总览, 简介]
created: 2026-07-08
updated: 2026-07-08
aliases: [GalNavi简介, GALNAVI]
---

# GalNavi 项目简介

> [!quote] README 定义
> "GalNavi 是一个专注于 ACG 圈的开源导航站。不仅收录优质资源站点，更为每一个站点整理介绍、标签、论坛、GitHub、发布页、帮助文档等信息，让寻找资源不再依赖搜索引擎。"

## 一句话定位

GalNavi（GALNAVI）是一个**面向 ACG / Galgame 爱好者的纯净、无广告、开源资源导航站**，把分散在互联网各处的资源站点、模拟器、工具、会社、汉化组聚合到统一界面，并为每个站点附带结构化的介绍与官方矩阵链接。

## 关键事实

| 项 | 内容 |
|---|---|
| 项目名 | GalNavi / GALNAVI |
| 域名 | [galnavi.top](https://galnavi.top) |
| GitHub | [argb6/gal-navigation](https://github.com/argb6/gal-navigation) |
| 协议 | MIT License (Copyright © 2026 GALNAVI) |
| Discord | discord.gg/ShSxAuE2（入口页）/ discord.gg/4RpywtYc（主站）|
| 反馈邮箱 | galnavifeedback@protonmail.com |
| 部署平台 | Cloudflare（边缘网络）|
| 内容语言 | 中文 (zh-CN) |

## 它做什么

1. **收录** ACG 资源站点、模拟器、工具、会社、汉化组
2. **结构化**每个站点：标题、简介、标签、图标、官方链接
3. **聚合**站点的"官方矩阵"：帮助文档、GitHub、论坛、外部社群
4. **导航**：通过分类视图 + 标签 + 搜索快速定位
5. **安全跳转**：外部链接经倒计时确认页跳转，防止误入与 referer 泄露

## 它不做什么

- ❌ 不托管资源（仓库声明："本仓库仅为导航性质，收录的所有工具、网站链接均指向第三方资源"）
- ❌ 不放广告（"纯净无广"是核心卖点）
- ❌ GitHub 仓库不存放网站源代码（详见 [GitHub 仓库的角色](../04-数据与资源/GitHub 仓库的角色.md)）

## 收录规模（截至 2026-07-08）

- 共 **29** 条目（来自 [D1 数据](../04-数据与资源/data.json 数据结构.md)）
  - 模拟器 `simulators`：7 条（Joiplay、Kirikiroid2、Mine、Ryujinx、Tyranor、Winlator、盖世游戏）
  - 网站 `websites`：22 条（资源站、论坛、百科等）
- 分类体系规划 5 类：模拟器 / 站点 / 工具 / 会社 / 汉化组

## 视觉风格

- 暗色主题（`--bg: #0c0c1a`），蓝紫青色调
- 玻璃拟态（glassmorphism）面板 + 呼吸光特效
- 圆角 18px、深阴影
- 零外部 JS/CSS 依赖（全内联，详见 [内联 JS 总览与加载策略](../03-部署的JS/内联 JS 总览与加载策略.md)）

## 相关笔记

- 为什么做 → [为什么开发 GalNavi（目的与动机）](为什么开发 GalNavi（目的与动机）.md)
- 作用与价值 → [GalNavi 的作用与价值](GalNavi 的作用与价值.md)
- 技术实现 → [整体技术架构](../02-网站架构/整体技术架构.md)
- 上一级 → [00 知识库地图 (MOC)](../00 知识库地图 (MOC).md)
