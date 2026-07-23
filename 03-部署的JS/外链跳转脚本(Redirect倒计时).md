---
tags: [GalNavi, JS, 安全, 外链跳转, 倒计时]
created: 2026-07-08
updated: 2026-07-24
aliases: [startRedirect, 外链跳转, 倒计时跳转, redirectOverlay]
---

# 外链跳转脚本（Redirect 倒计时）

> [!info] 位置
>
> 主站 `/nav/` 主应用脚本中的 `startRedirect(targetUrl)`。详情页有**同模式**的倒计时层（独立实现）。

## 设计目的

拦截外部链接 → 3 秒倒计时确认 → 可取消 → 到时用 `noopener,noreferrer` 打开。

## 触发机制（主站：全局事件委托）

```javascript
document.addEventListener('click', function(e) {
    var anchor = e.target.closest('a');
    if (!anchor) return;
    var href = anchor.getAttribute('href');
    if (!href) return;
    if (href.startsWith('https://galnavi.top/nav/') || href.startsWith('/nav/')) return;
    if (href.startsWith('#') || href.startsWith('javascript:')) return;
    if (!href.startsWith('http')) return;
    try { if (new URL(href).hostname === window.location.hostname) return; } catch (_) { return; }
    e.preventDefault();
    e.stopPropagation();
    startRedirect(href);
});
```

只拦截非同源 `http(s)` 外链；内部路由 / 锚点 / 同源不拦截。

## 主站实现（现源码）

```javascript
function startRedirect(targetUrl) {
    var overlay = document.getElementById('redirectOverlay');
    var countdownEl = document.getElementById('redirectCountdown');
    var cancelBtn = document.getElementById('redirectCancel');
    if (!overlay || !countdownEl) {
        window.open(targetUrl, '_blank', 'noopener,noreferrer');
        return;
    }
    // 若已有进行中的倒计时，先清理
    if (redirectTimerId) {
        clearInterval(redirectTimerId);
        redirectTimerId = null;
    }
    if (redirectCancelHandler && cancelBtn) {
        cancelBtn.removeEventListener('click', redirectCancelHandler);
        redirectCancelHandler = null;
    }

    var seconds = 3;
    overlay.classList.add('active');
    countdownEl.textContent = seconds;

    function tick() {
        seconds--;
        if (seconds <= 0) {
            clearInterval(redirectTimerId);
            redirectTimerId = null;
            window.open(targetUrl, '_blank', 'noopener,noreferrer');
            overlay.classList.remove('active');
            if (redirectCancelHandler && cancelBtn) {
                cancelBtn.removeEventListener('click', redirectCancelHandler);
                redirectCancelHandler = null;
            }
            return;
        }
        countdownEl.textContent = seconds;
    }

    redirectTimerId = setInterval(tick, 1000);

    redirectCancelHandler = function cancel() {
        clearInterval(redirectTimerId);
        redirectTimerId = null;
        overlay.classList.remove('active');
        if (cancelBtn) cancelBtn.removeEventListener('click', redirectCancelHandler);
        redirectCancelHandler = null;
    };

    if (cancelBtn) cancelBtn.addEventListener('click', redirectCancelHandler);
}
```

相对旧版的变化：用模块级 `redirectTimerId` / `redirectCancelHandler` 管理状态，重复点击会先清理上一次倒计时，避免多重定时器。

## 详情页

详情页对 `.link-entry` 绑定点击 → 同样 3 秒 overlay → `window.open(..., 'noopener,noreferrer')`。**不是**直接裸跳。

## 安全要点

| 手段 | 作用 |
|---|---|
| 3 秒确认 + 取消 | 防误点 |
| `noopener` | 防反向 Tab 劫持 |
| `noreferrer` | 不送 Referer |

## 相关笔记

- 上一级 → [[部署的JS]]
- 详情页 → [[详情与外链跳转]]
