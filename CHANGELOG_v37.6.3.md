# ChemAI v37.6.3 更新日志

> 日期：2026-08-17
> 基于：v37.6.2
> 性质：图片精修 + 日间模式修复

---

## 一、图片去重与命名统一（b6f16dd）

- **热重图（TG/DTA）仅保留 1 张**：删除 `配合物的TG和DTA分析曲线2`。
- **操作流程图仅保留 1 张并放大**：删除 `实验步骤2`，`实验步骤1 → 实验步骤`，加 `big` 标记 + `.fig-big{grid-column:1/-1}` 全宽样式。
- **命名统一**：去掉孤立 `1` 后缀——
  - `层析柱1 → 层析柱`
  - `结晶装置示意图1 → 结晶装置示意图`
  - `加草酸钾后浊液1 → 加草酸钾后浊液`
  - `实验步骤1 → 实验步骤`
- 图片总数 80 → 78，同步 `manual.json` / `images.json` / `README`。

## 二、删除两张图 + 图注底部对齐（39c33e8）

- 删除 `二草酸合铜(II)酸钾结构`、`层析柱` 两张图（ch10-s1 清零，78 → 76）。
- **图注底部对齐**：`figure` 改 flex 纵向布局 + `figcaption{margin-top:auto}`，同排图片高度不一时图注底部齐平（`main.html` 与 `assistant.html` 同步）。

## 三、日间模式表格文字修复（41c98a1）

- `.sec-body td` 硬编码浅灰 `#cfd8e6` 未在日间模式覆盖，白底下看不清。
- 补 `.sec-body td{color:#1f2937}` + 行悬停浅色，与正文/加粗文字一致改为深色。

---

## 当前图片状态

- 76 张，全部在 `assets/images/`（48 PNG + 28 JPG，约 4.6 MB）
- 命名统一、无孤立后缀，响应式网格 + 图注底部对齐

## 涉及文件

- `assets/images/*`（删除/重命名）
- `main.html` / `assistant.html`（图注对齐、日间模式、fig-big）
- `data/manual.json` / `data/images.json` / `assets/images/README.md`
