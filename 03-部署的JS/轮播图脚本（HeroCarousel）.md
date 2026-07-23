---
tags: [GalNavi, JS, 轮播图, Carousel, KV]
created: 2026-07-08
updated: 2026-07-08
aliases: [Hero轮播, 轮播图, initHeroCarousel]
---

# 轮播图脚本（Hero Carousel）

> ！位置
> 
> 主站 `/nav/` 主应用中的 `initHeroCarousel()`：轮播图加载、slide 生成、自动播放与手动控制。

## 职责

1. 从 KV（`/nav/api/hero`）获取轮播图片列表
2. KV 失败时用硬编码 `HERO_FALLBACK` 兜底
3. 动态生成 slide 元素与指示点（dots）
4. 绑定前进/后退/点击点切换
5. 4 秒自动播放，手动操作后重置计时

## 完整源码

```javascript
async function initHeroCarousel() {
    const carousel = document.getElementById('heroCarousel');
    const gradient = carousel.querySelector('.hero-gradient');
    const dotsContainer = document.getElementById('heroDots');

    // 1. 尝试从 KV 获取图片列表
    let images = [];
    try {
        const ctrl2 = new AbortController();
        const t2 = setTimeout(() => ctrl2.abort(), 10000);
        const resp = await fetch('/nav/api/hero', { cache: 'no-cache', signal: ctrl2.signal });
        clearTimeout(t2);
        if (resp.ok) {
            const data = await resp.json();
            // debug 模式返回 {images, debug}，正式模式返回纯数组
            const arr = Array.isArray(data) ? data : (data.images || []);
            if (Array.isArray(arr) && arr.length > 0) {
                images = arr;
            }
        }
    } catch (e) {}

    // 2. KV 为空或失败 → 使用硬编码 fallback
    if (images.length === 0) {
        images = HERO_FALLBACK;
    }

    // 3. 动态生成 slide 和 dot 元素
    images.forEach((url, i) => {
        const slide = document.createElement('div');
        slide.className = 'hero-slide' + (i === 0 ? ' active' : '');
        slide.style.backgroundImage = "url('" + url + "')";
        carousel.insertBefore(slide, gradient);

        const dot = document.createElement('span');
        dot.className = 'hero-dot' + (i === 0 ? ' active' : '');
        dotsContainer.appendChild(dot);
    });

    // 4. 绑定轮播事件
    const slides = document.querySelectorAll('.hero-slide');
    const dots = document.querySelectorAll('.hero-dot');
    const total = slides.length;

    function goTo(index) {
        heroIndex = (index + total) % total;
        slides.forEach((s, i) => s.classList.toggle('active', i === heroIndex));
        dots.forEach((d, i) => d.classList.toggle('active', i === heroIndex));
    }
    function next() { goTo(heroIndex + 1); }
    function prev() { goTo(heroIndex - 1); }

    document.getElementById('heroPrev').addEventListener('click', () => { prev(); resetAuto(); });
    document.getElementById('heroNext').addEventListener('click', () => { next(); resetAuto(); });
    dots.forEach((dot, i) => {
        dot.addEventListener('click', () => { goTo(i); resetAuto(); });
    });

    function startAuto() { heroInterval = setInterval(next, 4000); }
    function resetAuto() { clearInterval(heroInterval); startAuto(); }

    startAuto();
}
```

## 逐段解析

### 1. KV 优先获取
```javascript
const resp = await fetch('/nav/api/hero', { cache: 'no-cache', signal: ctrl2.signal });
```
- 与数据预加载同样的 10 秒超时保护
- `cache: 'no-cache'` 保证新鲜
- 响应兼容两种格式：
  - 纯数组 `[url1, url2, ...]`（正式模式）
  - 对象 `{images: [...], debug: "..."}`（debug 模式）
- `Array.isArray(data) ? data : (data.images || [])` 兼容处理

> Worker 现版固定返回对象 `{ images, debug }`（非纯数组）；客户端用上面的兼容写法取值。`debug` 为服务端解析诊断字段，业务展示只依赖 `images`。详见 「API端点清单」。

### 2. Fallback 兜底
```javascript
if (images.length === 0) { images = HERO_FALLBACK; }
```
- `HERO_FALLBACK` 是主应用脚本里定义的硬编码数组
- **当前 `HERO_FALLBACK = []`**（空数组），所以 KV 失败时轮播为空
- 这是个潜在改进点：fallback 应预置几张图保证可用性

### 3. 动态生成 DOM
- 每个 image 生成一个 `.hero-slide` div，`backgroundImage` 设为该 URL
- 第一个加 `active` 类（默认显示）
- 在 `.hero-gradient`（渐变遮罩）之前插入 slide
- 同时生成 `.hero-dot` 指示点

### 4. 切换逻辑 goTo
```javascript
function goTo(index) {
    heroIndex = (index + total) % total;  // 循环取模
    slides.forEach((s, i) => s.classList.toggle('active', i === heroIndex));
    dots.forEach((d, i) => d.classList.toggle('active', i === heroIndex));
}
```
- `(index + total) % total` 实现首尾循环（到末尾 next 回到 0）
- 用 `toggle('active', bool)` 切换激活态

### 5. 自动播放
```javascript
function startAuto() { heroInterval = setInterval(next, 4000); }
function resetAuto() { clearInterval(heroInterval); startAuto(); }
```
- 每 4 秒自动 `next()`
- 手动操作（点前后/点 dot）后 `resetAuto()` 清掉旧计时器重启，避免手动切完立刻又自动切

## 图片来源

当前 KV 中的图片指向 GitHub raw：
```
https://raw.githubusercontent.com/argb6/gal-navigation/main/docs/hero/hero1.png
https://raw.githubusercontent.com/argb6/gal-navigation/main/docs/hero/hero2.png
https://raw.githubusercontent.com/argb6/gal-navigation/main/docs/hero/hero3.png
```
这些图片存放在仓库 `docs/hero/` 目录（详见 「图片素材资源」），验证了 GitHub 仓库作为静态资源 CDN 的角色。

## 设计特点

| 特点 | 评价 |
|---|---|
| KV 优先 + fallback | 可用性好，但当前 fallback 空数组是隐患 |
| 10 秒超时 | 防止 KV 慢拖累轮播 |
| 兼容两种响应格式 | 健壮，兼容 debug/正式模式 |
| 循环取模 | 首尾无缝循环 |
| 手动操作重置计时 | 体验好，避免冲突 |
| background-image 而非 img | 便于用 CSS 控制铺满/渐变遮罩 |

## 视觉效果（基于 CSS 与截图）

- 全宽 Hero 区，图片作背景
- `.hero-gradient` 渐变遮罩压在图片上，底部渐深，便于叠文字
- slide 切换通过 `active` class 的 opacity/transform 过渡（CSS 动画）
- 底部 dots 指示当前位置，可点击跳转
- 左右有 prev/next 按钮

## 相关笔记

- 上一级 → [[部署的JS]]
- [[API端点清单]]
- [[主应用逻辑脚本（卡片与交互）]]
