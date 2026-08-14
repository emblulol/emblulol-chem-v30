# ChemAI v37.6 优化日志 — 流畅度提升 + 双库合并

> 日期：2026-08-14
> 基于：v37.5.1
> 性质：性能优化 + 数据整合（无知识丢失）

---

## 一、目标

1. **优化流畅度**：`assistant.html` 原 900 KB（内嵌 750 条 FAQ）导致首屏解析缓慢。
2. **不丧失精确度**：修复双 FAQ 库分叉（内嵌 750 vs `faq_unified.json` 700），合并为单一真相源，逐条保证零丢失。

---

## 二、核心改动

### 1. 双 FAQ 库合并为单一真相源（精确度）

| 源 | 原条数 | 合并后 |
|---|---|---|
| `assistant.html` 内嵌 `const FAQ` | 750 | 已移除，改为动态加载 |
| `data/faq_unified.json` | 700 | **847（唯一真相源）** |

- 以 `q`（问题，双库均唯一）为键去重：**603 条共有 + 147 条仅 html + 97 条仅 json = 847**。
- 字段级并集合并：
  - `answer` / `detail` / `title` / `subfield` 取 **html 版**（v36-v37 修正均落在 html，如「混沌平流」仅在 html 中）。
  - `keys` / `ents` 取 **并集**（603 条共有条目全部扩容，检索召回率不降反升）。
  - `manual_refs` 取自 json（613 条保留原文引用）；`knode` 取自 html（供知识图谱节点跳转）。
- 结果：**847 条、17 分类、9 字段全部无缺失**，无一条知识丢失。

### 2. assistant.html 动态加载 FAQ（流畅度）

- 移除 350 KB 内嵌数组，改为 `loadFAQ()` 异步 `fetch('data/faq_unified.json', {cache:'default'})`。
- 文件体积：**900 KB → 148 KB（−83%）**，首屏解析大幅提速。
- `cache:'default'` 使 1.46 MB 的 FAQ 在重复访问时命中 HTTP 缓存（304），不再每次全量下载。
- 原 `<link rel="preload" href="data/faq_unified.json">` 由「死预载」变为「真实预取」，与 loadFAQ 配合。
- 优雅降级：fetch 失败时 FAQ 为空，自动回退语料检索 + 类比 + 网络（与 `loadCorpus` 同模式）。

### 3. 修复 11 处 LaTeX 宏损坏（精确度）

- `assistant.html` 内嵌 FAQ 中 `\bullet`（自由基电子点号）被 0x08 退格符破坏为 `\x08ullet`。
- 合并过程中定位并替换为 `·`（与全库 1675 处自由基点号写法一致）。
- 修复后：控制字符残留 **0**，`ullet` 残留 **0**。

### 4. 去除 5 个 HTML 文件 UTF-8 BOM

- `assistant/corpus/knowledge/main/prep.html` 全部去 BOM（`index.html` 本无），与 v35 处理的 BOM 乱码同源隐患一并消除。

### 5. README 口径统一

- FAQ 徽章：750（旧）→ 700（v37.5.1 误判）→ **847**（本版，与合并后唯一真相源一致）。
- KP 知识点：正文「12 知识点」→ **10 知识点**（与 `assistant.html` 内嵌 10 个 KP 一致）。
- 项目结构注释标注 `faq_unified.json` 为「847 条，17 分类，单一真相源」。

---

## 三、校验结果

```
faq_unified.json 条数：847（唯一 q：847）
字段完整性：q/answer/subfield/keys/ents/detail/title/knode/manual_refs 缺失均 0
控制字符残留：0   |  'ullet' 残留：0   |  subfield 分类：17
assistant.html：148.4 KB（原 ~900 KB）
合并核对：603 共有 + 147 仅html + 97 仅json = 847 ✅（750 条 html + 700 条 json 全部入账）
```

---

## 四、遗留待办 → 已处置（v37.6.1 追加）

| # | 问题 | 处置 | 状态 |
|---|---|---|---|
| 1 | `data/assessment_kp.json` 8 KP vs `assistant.html` 10 KP | 全仓确认无人引用，`assessment_kp.json` 为死文件，已 `git rm` 删除 | ✅ 已删 |
| 2 | `manual.json` 11 章 vs `categories.json` 12 章 | `categories.json` 移除 phantom「实验报告撰写规范」，`ch12 常见实验故障排查 → ch11`，两文件现均 11 章且逐章一致 | ✅ 已对齐 |
| 3 | 悬挂 worktree `fix-eval` | `git worktree prune` 已清理；分支 `worktree-fix-eval` 有未合并提交（`v29: 评估报告全面修改 (排除#16)`），**保留未删** | ✅ 已清 |

### 仍待处理（本次未动，属数据质量而非机械修复）

1. **FAQ `manual_refs` 章号噪声**：约 32 条引用 `ch12`、19 条引用 `ch11`，且内容与章节名不对齐（`ch11` 引用的 RSD/t 检验实为「实验报告撰写规范」主题，该章已不在 manual.json 中；`ch12` 引用混有光化学与故障排查）。这些 `manual_refs` 仅被 `scripts/fill-detail-from-refs.js` 等**开发脚本**消费、不进入运行时页面，故影响有限，但需按 manual.json 实际章节结构**重新生成映射**，非本次机械修改所能覆盖。

2. **分支 `worktree-fix-eval`**：含独有提交 `4521b68`（v29，标记「排除#16」），如需删除须先人工确认该提交内容是否仍需并入 master。

---

## 五、涉及文件

- `data/faq_unified.json`（合并重写，847 条）
- `assistant.html`（动态加载 + 缓存 + 修复）
- `corpus/knowledge/main/prep.html`（去 BOM）
- `README.md` / `MEMORY.md`（口径）
