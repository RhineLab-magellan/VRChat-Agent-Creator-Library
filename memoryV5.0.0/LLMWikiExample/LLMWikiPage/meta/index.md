---
title: Meta Index - 知识库治理入口
category: meta
knowledge_level: governance
status: active
source: 本地知识库整理
source_type: local
version: 1.0
last_review: 2026-07-14
confidence: High
tags:
  - meta
  - index
  - navigation
  - governance
aliases:
  - Meta Index
  - 元协议索引
  - 知识库治理入口
related:
  - [[../index.md]]
  - "[[kb-protocol]]"
  - "[[working-modes]]"
  - "[[network-search-protocol]]"
  - "[[auxiliary-scripts]]"
  - "[[obsidian-governance]]"
type: governance
created: 2026-07-14
sources: 本地知识库整理
updated: 2026-07-14
---
# Meta Index - 知识库治理入口

> `meta/` 存放知识库治理、写入协议、工作模式、网络搜索准入与 Obsidian 分层治理规则。这里是 Curator / Agent 维护知识库时的协议入口，不存放领域事实正文。

## 协议入口

| 文档 | 用途 |
|---|---|
| [[kb-protocol]] | 写入流程、Curator 审计、文件职责边界 |
| [[working-modes]] | Architect / Engineer / Reviewer / Teacher / Researcher / Curator 等工作模式 |
| [[network-search-protocol]] | 网络来源分级、验证流程、禁止事项与入库标注规则 |
| [[auxiliary-scripts]] | 本地治理脚本与使用边界 |
| [[obsidian-governance]] | Vault Root、治理层、备份/工具/配置排除规则 |

## 使用规则

- 修改核心知识库结构前，先检查本目录协议。
- 网络搜索结果不得直接写入核心事实层，应按 [[network-search-protocol]] 处理。
- Obsidian 化改造应遵守 [[obsidian-governance]] 的分层边界。
- 本目录文件属于治理协议，不应与 `api/`、`avatar/`、`world/` 等领域知识混写。
