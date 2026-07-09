# 📚 GalNavi 知识库

> 围绕 [GalNavi](https://galnavi.top) 构建的多维度 Obsidian 知识库，从项目总览、网站架构、部署脚本、数据资源、页面详解、安全与 SEO 六大维度进行系统性整理。

**线上站点**：[galnavi.top](https://galnavi.top) ｜ **源仓库**：[argb6/gal-navigation](https://github.com/argb6/gal-navigation)

---

## ✨ 特性

- 🗂️ **6 大维度、33 篇笔记**，覆盖项目全貌
- 🔗 笔记间双向链接互相关联，可自由跳转
- 📊 全部基于线上实测（HTML / JS / API 实际抓取分析），非臆测
- 🏷️ 每篇笔记含 YAML front matter（tags、created、aliases）

## 📂 目录结构

```
galnavigation/
├── 00 知识库地图 (MOC).md          ← 总入口
├── 01-项目总览/                      ← 用途、目的、价值、开源社区
│   ├── GalNavi 项目简介
│   ├── 为什么开发 GalNavi（目的与动机）
│   ├── GalNavi 的作用与价值
│   └── 开源与社区
├── 02-网站架构/                      ← 架构、路由、API、存储
│   ├── 整体技术架构
│   ├── 路由与页面体系
│   ├── API 端点清单
│   ├── 存储层 D1 与 KV
│   └── 请求与渲染流程
├── 03-部署的JS/                      ← 内联脚本逐个分析
│   ├── 内联 JS 总览与加载策略
│   ├── 数据预加载脚本（D1 载入）
│   ├── 主应用逻辑脚本（卡片与交互）
│   ├── 轮播图脚本（Hero Carousel）
│   ├── 外链跳转脚本（Redirect 倒计时）
│   ├── 入口页发布页弹窗脚本
│   ├── 帮助页侧栏与锚点脚本
│   └── XSS 防护与 escapeHtml
├── 04-数据与资源/                    ← data.json、分类、标签、素材
│   ├── data.json 数据结构
│   ├── 标签体系
│   ├── 图片素材资源
│   └── GitHub 仓库的角色
├── 05-页面详解/                      ← 各页面详细分析
│   ├── 入口页（永久发布页）
│   ├── 主站导航页
│   ├── 帮助页
│   ├── 关于页
│   └── 详情与外链跳转
└── 06-安全与SEO/                     ← CSP、安全头、SEO、外链安全
    ├── 内容安全策略 CSP
    ├── 安全响应头
    ├── SEO 与可发现性
    └── 外链安全设计
```

## 🚀 使用方式

### 本地浏览（推荐）

1. 克隆仓库

```bash
git clone https://github.com/argb6/galnavi-obsidian.git
```

2. 用 [Obsidian](https://obsidian.md/) 打开 `galnavigation/` 作为 Vault
3. 从 [00 知识库地图 (MOC)](00知识库地图(MOC).md) 开始浏览

### 在线浏览

直接在 GitHub 上点击各文件夹和 `.md` 文件即可阅读，双向链接会在 GitHub 中显示为普通超链接。

## 📋 知识库维度概览

| 维度 | 目录 | 核心内容 |
|---|---|---|
| 项目总览 | `01-项目总览/` | 定位、动机、价值、开源社区 |
| 网站架构 | `02-网站架构/` | Cloudflare Workers + D1 + KV 技术栈 |
| 部署脚本 | `03-部署的JS/` | 8 个内联脚本的逐个分析 |
| 数据资源 | `04-数据与资源/` | 站点分类、标签、素材 |
| 页面详解 | `05-页面详解/` | 6 个页面的 SSR / 交互细节 |
| 安全 SEO | `06-安全与SEO/` | CSP、响应头、外链保护、SEO |

## 📝 信息来源

| 来源                                                              | 用途                       |
| --------------------------------------------------------------- | ------------------------ |
| [galnavi.top](https://galnavi.top) 实际抓取                         | HTML 结构、内联 JS、API 端点、响应头 |
| [argb6/gal-navigation](https://github.com/argb6/gal-navigation) | README、data.json、图片素材    |

## ⚖️ 许可

[MIT License](https://github.com/argb6/galnavi-obsidian?tab=MIT-1-ov-file) © 2026
