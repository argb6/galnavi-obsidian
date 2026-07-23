# 📚 GalNavi 知识库

> 围绕 [GalNavi](https://galnavi.top) 构建的多维度 Obsidian 知识库，从项目总览、网站架构、部署脚本、数据资源、页面详解、安全与 SEO 六大维度进行系统性整理。

**线上站点**：[galnavi.top](https://galnavi.top) ｜ **源仓库**：[argb6/gal-navigation](https://github.com/argb6/gal-navigation)

---

## ✨ 特性

- 🗂️ **分层索引**：主地图 → 分节索引 → 叶子笔记，图谱按文件夹成簇
- 📊 基于线上实测（HTML / API / 响应头）
- 🏷️ 每篇笔记含 YAML front matter（tags、created、aliases）

## 📂 目录结构

```
galnavi-obsidian/
├── 00知识库地图(MOC).md              ← 总入口（只链分节索引）
├── 网站框架总览.md                    ← Mermaid 全站总图
├── 01-项目总览/
│   ├── 项目总览.md                    ← 本节索引
│   ├── GalNavi项目简介
│   ├── 为什么开发GalNavi（目的与动机）
│   ├── GalNavi的作用与价值
│   ├── 开源与社区
│   └── 纳普吉祥物
├── 02-网站架构/
│   ├── 网站架构.md                    ← 本节索引
│   ├── 整体技术架构
│   ├── 路由与页面体系
│   ├── API端点清单
│   ├── 存储层D1与KV
│   └── 请求与渲染流程
├── 03-部署的JS/
│   ├── 部署的JS.md                    ← 本节索引
│   ├── 内联JS总览与加载策略
│   ├── 数据预加载脚本（D1载入）
│   ├── 主应用逻辑脚本（卡片与交互）
│   ├── 轮播图脚本（HeroCarousel）
│   ├── 外链跳转脚本(Redirect倒计时)
│   ├── 入口页发布页弹窗脚本
│   ├── 帮助页侧栏与锚点脚本
│   └── XSS防护与escapeHtml
├── 04-数据与资源/
│   ├── 数据与资源.md                  ← 本节索引
│   ├── data.json数据结构
│   ├── 标签体系
│   ├── 图片素材资源
│   └── GitHub仓库的角色
├── 05-页面详解/
│   ├── 页面详解.md                    ← 本节索引
│   ├── 入口页（永久发布页）
│   ├── 主站导航页
│   ├── 圣器殿堂
│   ├── 帮助页
│   ├── 关于页
│   └── 详情与外链跳转
└── 06-安全与SEO/
    ├── 安全与SEO.md                   ← 本节索引
    ├── 内容安全策略CSP
    ├── 安全响应头
    ├── SEO与可发现性
    └── 外链安全设计
```

## 🚀 使用方式

### 本地浏览（推荐）

1. 克隆仓库

```bash
git clone https://github.com/argb6/galnavi-obsidian.git
```

2. 用 [Obsidian](https://obsidian.md/) 打开本仓库根目录作为 Vault
3. 从 [00知识库地图(MOC)](00知识库地图(MOC).md) 进入分节索引再下钻

### 在线浏览

直接在 GitHub 上点击各文件夹和 `.md` 文件即可阅读。

## 📋 知识库维度概览

| 维度 | 目录索引 | 核心内容 |
|---|---|---|
| 项目总览 | `项目总览.md` | 定位、动机、价值、开源、纳普 |
| 网站架构 | `网站架构.md` | Workers + D1 + KV |
| 部署脚本 | `部署的JS.md` | 内联脚本分析 |
| 数据资源 | `数据与资源.md` | data.json、标签、素材 |
| 页面详解 | `页面详解.md` | 各路由页面 |
| 安全 SEO | `安全与SEO.md` | CSP、响应头、外链、SEO |

## 📝 信息来源

| 来源 | 用途 |
|---|---|
| [galnavi.top](https://galnavi.top) 实际抓取 | HTML / 内联脚本 / API / 响应头 |
| [argb6/gal-navigation](https://github.com/argb6/gal-navigation) | README、data.json、图片素材 |

## ⚖️ 许可

[MIT License](https://github.com/argb6/galnavi-obsidian?tab=MIT-1-ov-file) © 2026
