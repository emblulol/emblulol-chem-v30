# ChemAI v37.6 综合更新日志（本次会话全量）

> 范围：v37.5（18f25be）→ v37.6.4（4f66f51）
> 提交：15 次 · 改动文件：122 个
> 日期：2026-08-14 ~ 2026-08-17

---

## 一、总体概览

本次会话围绕「三草酸合铁(III)酸钾制备实验智能教学平台」完成了四大方向的工作：

1. **数据整合**：双 FAQ 库合并为单一真相源，章节 ID 一致性修正。
2. **实验图片**：76 张实验图片接入实验手册与 AI 助手。
3. **排版与主题**：化学式正体、日间模式、图注对齐、响应式。
4. **工程卫生**：部署修复、缓存策略、旧引用清理、文档日志。

---

## 二、按主题分类的变更

### A. 数据整合与修复

| 变更 | 说明 |
|---|---|
| 坏 JSON 修复 | `deep_operation_questions_30.json` 内嵌引号导致非法 JSON，修复 3 处 |
| BOM 清理 | 去除 6 个 HTML + 10 个 JSON 文件的 UTF-8 BOM |
| **双 FAQ 库合并** | 内嵌 750 条 + `faq_unified.json` 700 条 → **847 条单一真相源**（q 为键去重，keys/ents 并集，manual_refs/knode 双字段保留） |
| **assistant.html 动态加载** | 移除 350KB 内嵌数组 → `fetch` 异步加载，**900KB → 148KB（−83%）** |
| 章节 ID 修正 | `manual.json` 最后一章 `ch12→ch11`，section `ch12-s*→ch11-s*` |
| FAQ manual_refs 修正 | 错位章节号 `ch12→ch11`（32 处） |
| 残留 ch12 清理 | 归档/数据/脚本统一改 `ch11`（25 文件约 405 处） |
| 死文件删除 | `data/assessment_kp.json`（8 KP，无人引用） |
| 章节对齐 | `categories.json` 章节 12→11（移除 phantom「实验报告撰写规范」） |

### B. 实验图片（76 张）

| 变更 | 说明 |
|---|---|
| 接入手册 | 80 张图映射到 `main.html` 12 个小节（含图注），`manual.json` 加 `images` 字段 |
| 助手带图 | 新增 `data/images.json`（80 条含关键词），`assistant.html` 按 FAQ/manual_refs + 关键词匹配，回答末尾附「📷 相关实验图片」 |
| 迁移 | `实验图片说明/` → `assets/images/`（英文路径，规避 URL 编码） |
| 压缩 | 29 张照片转 JPEG(q82)、51 张 PNG 无损优化，**16MB → 4.6MB（−72%）** |
| 去重 | 热重图(TG/DTA)、操作流程图各仅留 1 张（80→78） |
| 删除 | 二草酸合铜(II)酸钾结构、层析柱（78→76） |
| 命名统一 | 去掉孤立 `1` 后缀（层析柱1→层析柱 等 4 处） |
| 操作流程图放大 | `big` 标记 + `.fig-big{grid-column:1/-1}` 全宽显示 |
| 说明索引 | `assets/images/README.md`（按章节分类） |

### C. 排版与主题

| 变更 | 说明 |
|---|---|
| 化学式正体 | `.fx`/`.ans-inline-math` 去掉 Cambria Math（数学字体默认斜体）→ Times New Roman 正体 |
| 日间模式 | 新增 `@media(prefers-color-scheme:light)`，正文灰字 `#cfd8e6`→深色 `#1f2937`，含表格单元格 |
| 图注底部对齐 | `figure` 改 flex 纵向布局 + `figcaption{margin-top:auto}` |
| 响应式 | 网格 `repeat(auto-fill,minmax(...))` 自适应手机/电脑 |

### D. 工程卫生与部署

| 变更 | 说明 |
|---|---|
| 部署修复 | `deploy.yml` 漏复制图片目录（曾致图片不部署） |
| 缓存策略 | 数据请求加 `no-store`（manual.json/images.json），图片 URL 加 `?v=2` |
| 图片加载 | 移除 `loading="lazy"`（grid 内可能不触发） |
| 仓库地址 | README 旧地址 `LittleAlety/chemai-8.6-` → `emblulol/emblulol-chem-v30`（6 处） |
| 版本历史 | README 版本徽章 v37→v37.6，补 v37.6/v37.6.x 条目 |
| worktree | `git worktree prune` 清理悬挂 worktree |

---

## 三、完整提交列表

```
4f66f51 docs: v37.6.4 更新日志
202fbba chore: 清理残留 ch12/第12章 引用
ce541f2 fix: 检查并修复需要更新的点
227ca39 docs: v37.6.3 更新日志
41c98a1 fix: 日间模式表格单元格文字颜色
39c33e8 refactor: 删除二草酸合铜/层析柱图片 + 图注底部对齐
b6f16dd refactor: 图片去重与命名统一
1c6045f docs: v37.6.2 修复日志
c185a81 fix: 手册图片加载 + 化学式正体 + 日间模式
5fc1d9b fix: 数据请求加 no-store 防浏览器缓存
c9e1ce7 refactor: 图片迁移到 assets/images/ 并优化体积
a9e360f feat: 助手回答携带相关实验图片
b9c8316 fix: 部署工作流漏复制实验图片说明目录
9db3523 feat: 实验图片接入知识手册 + 图片说明索引
0c53a90 v37.6: 双 FAQ 库合并为单一真相源 + assistant.html 动态加载
```

---

## 四、当前状态

- **FAQ**：847 条（17 分类，单一真相源 `faq_unified.json`）
- **实验图片**：76 张（`assets/images/`，48 PNG + 28 JPG，约 4.6 MB）
- **实验手册**：11 章（`main.html`），章节 ID 一致（ch1-ch11）
- **AI 助手**：动态加载 FAQ + 按匹配携带相关图片
- **仓库**：`emblulol/emblulol-chem-v30`，master 分支，GitHub Pages 自动部署
- **日志**：CHANGELOG_v37.5.1 / v37.6 / v37.6.2 / v37.6.3 / v37.6.4 / 本综合篇

---

## 五、遗留观察（非运行时，供后续参考）

1. `assets/index-B-pT4Snc.js`（React 落地页 minified 构建产物）内仍含 `ch12` 字样，需从 React 源码改后重新构建（手改构建产物已还原）。
2. 分支 `worktree-fix-eval` 含独有提交 `4521b68`（v29，标「排除#16」），未合并未删除。
3. FAQ 中约 19 条 `manual_refs` 原指向已删除的「实验报告撰写规范」章节，现归入 ch11（语义上属数据质量噪声，需逐条重映射）。
