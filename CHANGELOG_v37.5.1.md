# ChemAI v37.5.1 修复日志

> 日期：2026-08-14
> 基于：v37.5（commit `18f25be`）
> 性质：数据修复 + 文档口径统一（不含新功能）

---

## 一、摘要

对 v37.5 全量巡检发现的 4 类问题做定向修复：

| # | 问题 | 严重性 | 处理 |
|---|------|:------:|------|
| 1 | `deep_operation_questions_30.json` 为非法 JSON，无法解析 | 高 | 已修复 |
| 2 | README FAQ 数量口径混乱（750 / 700+ / 实际 700） | 中 | 已统一为 700 |
| 3 | 10 个 JSON 数据/归档文件带 UTF-8 BOM | 中 | 已全部去除 |
| 4 | v37.5 的 30 题 + 评分结果未接入任何页面/脚本 | 低 | 记录待办（见文末） |

---

## 二、修改明细

### 1. 修复 `deep_operation_questions_30.json` 非法 JSON（高危）

**根因**：v37.5 自学习循环生成的答案/详情字符串中，混入了**未转义的 ASCII 直双引号** `"`，导致 JSON 解析器报
`Expecting ',' delimiter: line 203 column 396`，整个文件（31 题）无法加载。

**修复**：将 3 处字符串值内的 ASCII 直引号改写为中文全角引号 `“ ”`，共 8 对：

| 行 | 位置 | 修复前 → 修复后 |
|----|------|----------------|
| 203 | `这就是"热水洗比冷水好"的` | → `这就是“热水洗比冷水好”的` |
| 252 | `如"Spectral Workbench"）` | → `如“Spectral Workbench”）` |
| 266 | `"A/B/C"`、`"试剂自查卡"`、`"草酸→…"`、`"草酸替草酸钾"`、`"30%替6% H₂O₂"`、`"五分钟预警信号"`（6 对） | → 全部改为 `“ ”` |

**校验**：`json.load` 纯 utf-8 解析通过，得到 31 题。

### 2. 统一 FAQ 数量口径（中）

`data/faq_unified.json` 实际为 **700 条 / 17 分类**，但 README 徽章写 `750条`、正文注释写 `700+ 条`，git 提交信息写 `750+`，三处互相矛盾。

**修复**（README.md 两处）：
- 顶部徽章 `FAQ-750条` → `FAQ-700条`
- 项目结构注释 `FAQ 知识库（700+ 条，17 分类）` → `FAQ 知识库（700 条，17 分类）`

### 3. 去除 JSON 文件的 UTF-8 BOM（中）

以下 10 个文件带 `EF BB BF` 前缀，用标准 `utf-8` 解码会直接报 `Unexpected UTF-8 BOM`（与 v35 处理过的 BOM 乱码同源隐患）：

**data/（运行期数据，3 个）**
- `data/faq_unified.json`
- `data/kb.json`
- `data/kg.json`

**结构化输出/（归档产物，7 个）**
- `compact_output.json`、`structured_output.json`、`structured_output_compact.json`
- `structured_output_compact_r19.json`、`structured_output_data.json`
- `structured_output_payload.json`、`structured_output_payload_r19.json`

**处理**：字节级删除前 3 字节 BOM，其余内容逐字节保留，无二次编码风险。

---

## 三、校验结果

```
全量 JSON 扫描（**/*.json，共 71 个）：
  可解析（纯 utf-8）：71 / 71  ✅
  坏文件数：0
关键文件条数：
  data/faq_unified.json        = 700 条（17 分类）
  deep_operation_questions_30.json = 31 题
  deep_questions_scoring.json  = 30 题
  data/kb.json                 = 1335 条
  data/kg.json                 = 2 节点（97 节点 / 136 关联）
```

`git diff --stat`：12 个文件，仅 3 处内容行 + 10 处 BOM 删除，无整文件重写、无行尾符（CRLF/LF）污染。

---

## 四、遗留观察与待办（未在本次改动，避免引入回归）

1. **条数不一致**：`deep_operation_questions_30.json` 实际含 **31 题**，而文件名、README（“30 条”）及评分文件 `deep_questions_scoring.json`（30 题）均记为 30。疑为生成过程中多出 1 题或评分时漏掉 1 题，需人工核对后再统一口径，**本次未擅自删题**。

2. **孤儿产物未接入**：`deep_operation_questions_30.json` 与 `deep_questions_scoring.json` 仍未被 `assistant.html` 或任何脚本引用。git 历史显示对抗评分后仅手工插入了 **3 条修正 FAQ**。由于 30 题中有 7 题被判“错误”，**不应全量自动注入**（否则会重新引入自学习循环本要消除的错误）。建议：将这两个文件明确标注为“v37.5 自学习原始产物/评分底稿”归档，仅保留已修正的 3 条进入正式 FAQ。

3. **`assistant.html` 内嵌 FAQ 与 `faq_unified.json` 的同步关系**：页面内嵌 FAQ 数组（`const FAQ`，单引号 JS 对象字面量）与 `data/faq_unified.json`（双引号 JSON）是两套数据，README 的“750+”口径可能曾试图合并两者。建议在文档中明确二者边界，避免再次出现数量口径漂移。
