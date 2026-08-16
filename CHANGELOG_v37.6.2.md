# ChemAI v37.6.2 修复日志

> 日期：2026-08-16
> 基于：v37.6.1
> 性质：手册图片加载 + 排版 + 主题适配

---

## 一、手册图片不可见的修复尝试

**症状**：`main.html` 手册正文能正常显示，但图片完全不出现（非裂图、非空白框，而是没有图片区域）。

**排查结论**：线上数据与代码均正确——
- 线上 `manual.json` 80 处图片引用全部存在；
- 图片文件（中文/`.jpg`/含括号）curl 全部 200 OK；
- `main.html` 已含 `sec-figs`/`s.images` 渲染代码。

**修复动作（针对浏览器侧最可能原因）**：
1. 移除 `<img loading="lazy">`（懒加载在 grid 容器内可能不触发，改为立即加载）；
2. 图片 URL 追加 `?v=2` 破坏浏览器/代理缓存。

> ⚠️ 若此版仍不显示，需用户提供截图定位（区分：无图片区域 / 裂图 / 有图但空白）。

## 二、化学式斜体 → 正体

- `main.html` 的 `.fx` 字体由 `"Cambria Math","Times New Roman",...` → `"Times New Roman",Georgia,serif`。
- `assistant.html` 的 `.ans-inline-math` 同样去掉 `"Cambria Math"/"Latin Modern Math"`。
- **原因**：Cambria Math 为数学字体，拉丁字母默认渲染为斜体（数学变量惯例），即使 `font-style:normal` 也无法矫正；改普通衬线字体后字母呈正体（罗马体）。

## 三、日间模式适配

- 新增 `@media (prefers-color-scheme:light)`：
  - `:root` 变量切换为浅色（背景白、文字深）；
  - `.sec-body` 正文由灰 `#cfd8e6` → 深色 `#1f2937`；
  - `.sec-body strong`、`.navbar`、`.sec-figs` 同步适配浅色。
- 效果：系统切浅色主题时，原本黑底灰字自动变为白底黑字。

---

## 涉及文件

- `main.html`（图片渲染、`.fx` 字体、日间模式 CSS）
- `assistant.html`（图片渲染、`.ans-inline-math` 字体）

## 提交

- `c185a81` fix: 手册图片加载 + 化学式正体 + 日间模式
