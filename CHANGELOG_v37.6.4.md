# ChemAI v37.6.4 更新日志

> 日期：2026-08-17
> 基于：v37.6.3
> 性质：一致性检查修复 + 章节 ID 清理

---

## 一、一致性检查修复（ce541f2）

| # | 问题 | 修复 |
|---|---|---|
| 1 | README 仍指向旧仓库 `LittleAlety/chemai-8.6-`（6 处） | 全部改为 `emblulol/emblulol-chem-v30` |
| 2 | `manual.json` 最后一章 id=`ch12` 但 number=`11`（章节 ID 错位） | `ch12→ch11`，section `ch12-s*→ch11-s*` |
| 3 | FAQ `manual_refs` 引用错位章节 `ch12`（32 处） | `ch12-*→ch11-*` |
| 4 | README 版本徽章 `v37`、版本历史缺 v37.6 系列 | 徽章改 `v37.6`，历史补 v37.6 / v37.6.x 条目 |

**修复后**：章节 ID 与 number 一致（ch1-ch11），FAQ/images 引用无无效章节。

## 二、清理残留 ch12 引用（202fbba）

- 残留 `ch12` / `第12章` 统一改为 `ch11` / `第11章`（25 个文件、约 405 处）：
  - 运行时数据：`manual.json`（章节 ID + quiz 的 `chapterId`/`q12-→q11-`）、`faq_unified.json`（正文「Manual ch12-s*」文字）、`kb.json`（168 处）
  - 归档：`Agent工作区/`（12 个）、`试题迭代记录/`（3 个）、`结构化输出/`（4 个）
  - 脚本：`fill-detail-from-refs.js`、`训练管道/evaluate.js`、`训练管道/gen_round3.js`
- **保留不改**：README/CHANGELOG 中的「ch12→ch11」修正说明；`assets/index-*.js`（minified 构建产物，已还原）。

---

## 涉及文件

- `data/manual.json` / `data/faq_unified.json` / `data/kb.json`
- `README.md`
- 归档目录（Agent工作区 / 试题迭代记录 / 结构化输出）
- 脚本（fill-detail-from-refs.js / evaluate.js / gen_round3.js）

## 校验

- 三个 JSON 数据文件均有效
- 改过的 JS 脚本 `node --check` 语法通过
- 主项目无 `ch12`/`第12章` 残留（说明文档与构建产物除外）
