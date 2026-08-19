---
title: "Queries — 查询归档索引"
category: misc
knowledge_level: applied
status: active
source: 本地知识库整理
source_type: community
version: 1.0
last_review: 2026-08-09
confidence: High
tags:
  - qa
  - navigation
aliases:
  - 查询归档
  - Queries
related:
  - index.md
type: entity
created: 2026-08-09
sources: 本地知识库整理
updated: 2026-08-09
---
# Queries — 查询归档索引

> LLM Wiki Query 分区：收录值得保留的优质查询结果（实质对比、深挖、新颖综合）。琐碎查询不归档。

## 归档规则

- **何时归档**：回答需要综合 3+ 页面、或包含新颖综合/对比结论
- **何时不归档**：单页即答的琐碎问题、纯事实速查
- **命名**：`YYYY-MM-DD-<主题>.md` 或 `<主题>-query.md`
- **结构**：问题 → 答案 → 引用页面列表（wikilink）→ 结论

## 已归档查询

| 日期 | 问题 | 答案入口 | 综合度 |
|------|------|----------|--------|
| 2026-07 | Avatar Shader 怎么选：lilToon / Poiyomi / SCSS / ORL / Filamented / UnlitWF？ | [[../entities/avatar/shader/index.md]] | 多页对比 |
| 2026-07 | Poiyomi Pro 与 Poiyomi Toon 选哪个？ | [[../comparisons/poiyomi-pro-vs-toon.md]] | 对比结论 |
| 2026-07 | Filamented / ORL / lilToon 功能差异矩阵？ | [[../comparisons/filamented-comparison.md]] · [[../comparisons/orl-comparison.md]] | 多页矩阵 |
| 2026-06 | UdonSharp 能否使用 List / Dictionary 等集合类型？ | [[../concepts/rules/udonsharp-language-limits.md]] | 规则速查 |
| 2026-06 | 世界持久化数据如何设计方案？ | [[../entities/world/udon/persistence/index.md]] | 架构综合 |

## 维护规则

- 新增查询页必须在本表登记
- 归档后 log.md 追加 query 条目