# ABYSSAL PIXEL PROBE · 深渊像素探勘器

一台被遗弃在深海科考站的 1980 年代终端。低分辨率像素画面、严格 12 色板、Bayer 有序抖动、缓慢上升的气泡、偶发掠过的巨大剪影。移动鼠标操控探照灯，空格发射声呐，滚轮下潜。

**单文件、零依赖、零外部资源**——一个 HTML 打开即用，没有构建步骤，没有网络请求。

![status](https://img.shields.io/badge/build-none-brightgreen) ![type](https://img.shields.io/badge/single--file-HTML-blue) ![deps](https://img.shields.io/badge/dependencies-0-brightgreen)

## 打开

双击 `abyssal-pixel-probe.html`，或拖进任意现代浏览器。

想在线分享时用 GitHub Pages（Settings → Pages → Branch: `main` / root），页面是纯前端，不需要任何配置。

## 操作

| 输入 | 作用 |
|---|---|
| 鼠标移动 / 触摸拖拽 | 控制探照灯 |
| `SPACE` / 点击 PING | 发射声呐脉冲 |
| `S` / QUIET | 静默模式（停止扫描，换一种听法） |
| `A` / 右下 AUDIO | 环境音频（**默认关闭**，首次需手动开启） |
| `WHEEL` / `↑` `↓` | 改变深度 |
| `R` | 复位 |
| `ESC` | 取消选择 |

深度分为 PHOTIC / TWILIGHT / ABYSSAL / HADAL 四层，越深越暗、观察越慢、声呐冷却越长。靠近信号点会显形，点击可归档；集齐六个触发一次全归档事件。

## 技术约束

- 主画面 384×216 逻辑分辨率 canvas，`image-rendering: pixelated` 放大，canvas 上不使用 alpha。
- 12 色深海色板，色板外颜色一律不用（透明度除外）；渐变一律用 Bayer 4×4/8×8 有序抖动。
- HUD 为 DOM + CSS，不用外部字体。
- Web Audio 懒加载，默认关闭，首次开启必须由用户手势触发。
- 无障碍：`sr-only`、`aria-live`、焦点可见、`aria-pressed`、`prefers-reduced-motion` 分支。
- 30fps 上限，粒子总数 < 120，页面隐藏时暂停。

## 仓库结构

```
abyssal-pixel-probe.html   成品，单文件（唯一需要部署的东西）
SPEC.md                    两份工单：生成规格 + P0–P5 优化工单
CHANGELOG.md               v1 / v2 改动与根因
```

`SPEC.md` 是这仓库里更值钱的那一半：**带验收断言的工单**可以换模型、换项目复用，代码只有这一份。

## 已知限制

- 环境声设计在 30–55Hz，笔记本与手机扬声器放不出来；v2 补了一层 147–220Hz 的 mains-hum 作为可听锚点。想要完整低频需要用耳机或外接音箱。
- 音频只在参数层与人工试听中验证，没有跨浏览器的自动化听感测试。
