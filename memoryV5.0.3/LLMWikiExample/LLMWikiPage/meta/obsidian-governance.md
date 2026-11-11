---
title: Obsidian Governance - LLMWikiPage 分层治理规则
category: meta
knowledge_level: governance
status: active
source: 本地知识库治理
source_type: local
version: 2.0
last_review: 2026-08-20
confidence: High
tags:
  - meta
  - obsidian
  - governance
  - knowledge-base
aliases:
  - 知识库治理规则
  - LLMWikiPage Governance
  - Vault Governance
related:
  - "[[../FACT.md]]"
  - "[[../index.md]]"
  - "[[../meta/kb-protocol.md]]"
  - "[[../meta/auxiliary-scripts.md]]"
type: governance
created: 2026-07-14
sources: 本地知识库治理
updated: 2026-08-20
---
# Obsidian Governance - LLMWikiPage 分层治理规则

> 本文件定义本知识发布包的 Obsidian Vault 治理边界与质量要求。目标是让知识层完整 Obsidian 化、可被 Agent 机器解析，同时避免工具、备份、历史报告污染知识图谱或被误判为当前事实。
> **Vault Root = 包内知识根（`LLMWikiPage/`）**。本文描述的目录结构、统计与路径均以发布包内实际分布为准，不从外部工作区硬编码任何路径。

## Vault Root

```text
LLMWikiPage/           # 本包 Obsidian Vault Root（知识页根）
```

Obsidian 写入、索引、链接治理默认限定在该根目录内。包外内容（源材料、工具、备份）不参与本库知识图谱。

## 治理层级

### A. 核心知识层（完整 Obsidian 化）

范围（当前物理目录即域边界）：

```text
entities/    # 实体域：api / avatar(含 shader) / platform / vrchatsdk / world
concepts/    # 概念域：hybrid / misc / patterns / rules
comparisons/ # 跨实体对比
queries/     # 问答归档
FACT.md / index.md / _always-load.md   # 顶层治理与导航
```

治理要求：

- 每页必须有可解析的 YAML frontmatter（title/type/category/tags/aliases/related/confidence/sources）。
- `tags` 遵循标签分类法；`aliases`、`related` 用于检索与图谱。
- 每页应可由 `index.md` 或所属域 `index.md` 索引可达。
- 页间关系使用库内 wikilink（见「推荐 Wikilink 规则」）。
- 参与死链、孤页、索引死链、frontmatter 合规、可解析性审查。

### B. 来源 / 参考层（包外，不随发布包分发）

- 源材料（raw 源、参考文献快照）以哈希账本锚定存放于包外，**不随本包分发**。
- 知识页 `sources` 字段只允许 `http(s)://` URL 或可追溯的文字描述，**不得写本地文件路径**。
- 参考资料晋升为核心知识前，须经验证、拆解与风险标注。

### C. 治理报告层（证据归档，不进入主知识网络）

- 审查报告、检查 JSON、审计记录存于包外审计目录，可作证据保留。
- 不要求与核心知识相同的 `aliases` / `related` 完整度；不参与主知识图谱质量评分；不作为当前知识事实来源。

## 自包含硬性要求（v5.0.0 起）

- 库内**只允许两种引用**：`http(s)://` URL 与**指向本包 `LLMWikiPage/` 内部**的文件链接/wikilink。
- 禁止任何指向包外或本机文件系统的硬编码路径（例如带盘符的绝对路径、file 方案链接、跨层跳转相对路径、指向包外 raw 源材料层的路径文本等）。
- 正文中的路径**示例**必须参数化（如 `<用户目录>\...`、`<安装盘>:\...`），不得出现具体盘符/用户名/工作区路径。
- 死链（wikilink / 相对文件链接）与孤页必须为 0；站点相对 URL（缺主机）不得出现。

## 禁止事项

- 禁止把 `entities/`、`concepts/` 等知识层与治理/源层无差别套用同一套质量规则。
- 禁止无基线执行批量改名、批量删除、批量链接重写。
- 禁止把旧报告、备份文件或机器日志作为当前知识事实。
- 禁止把所有路径无脑替换为裸 `[[index]]` 等可能歧义的 wikilink。
- 禁止在 `sources` / `related` 中写入本地文件路径或包外路径。

## Properties 字段语义

### `related`

只表示 **Vault 内部知识页关系**，优先使用路径型 Wikilink：

```yaml
related:
  - "[[entities/world/index.md]]"
  - "[[entities/api/networking.md]]"
```

规则：

- 只放当前 `LLMWikiPage/` 内真实存在的页面。
- 跨目录关系使用路径型 Wikilink；避免裸 `[[index]]`、`[[overview]]`。
- `related` 不承载来源路径、外部 URL、历史参考目录或包外文件。

### `sources`

- 只允许 `http(s)://` URL（用 `|` 分隔多个）或文字来源描述。
- 不写带盘符的绝对路径、跨层跳转相对路径文本或指向包外源材料层的路径；源材料层不随包分发。

## 推荐 Wikilink 规则

由于库内存在多个同名文件（`index.md`、`overview.md`、`installation.md` 等），**优先使用库根相对路径型 Wikilink**：

```md
[[entities/avatar/index.md]]
[[entities/world/udon/index.md]]
[[entities/api/networking.md]]
```

同一目录短链接可在试点保留，但新增关系一律写库根相对路径。

## 质量审查门（发布前）

每次发布前必须复跑并满足：

- C1 自包含：死链文件链接 0 / 死链 wikilink 0 / 站点相对 URL 0 / 畸形 wikilink 0 / 包外路径 0
- C2 置信度：high/medium/low 缺失或非法 0
- C3 Fact Index：孤页 0 / index 死链 0
- C4 标签：tags 缺失 0 / 分类法外复用（≥2 次）0

## 当前状态摘要（2026-08-20，发布复扫）

> 精确计数以发布审查报告为准（包外审计目录），本文件不长期硬编码漂移。

- 知识页总数：414（entities 351 / concepts 47 / comparisons 6 / queries 1 / meta 6 + 顶层 3）
- frontmatter 合规 100%
- 死链 0 / 孤页 0（发布口径）
- 分类法 150；无 ≥2 次分类法外复用
- 本文件 v2.0（2026-08-20）起，治理规则与现行 `LLMWikiPage/` 拓扑对齐
