---
title: "World Patterns — Udon 设计模式与典型场景"
category: world
subcategory: patterns
knowledge_level: applied
status: active
source: 本地知识库整理
source_type: community
version: 1.0
last_review: 2026-07-04
confidence: Medium
tags:
  - world
  - patterns
  - udonsharp
  - index
aliases:
  - "World Patterns"
  - "Udon 设计模式"
  - "World 模式库"
related:
  - ../udon/index.md
  - ../../../concepts/patterns/index.md
  - ../scene-components/index.md
type: entity
created: 2026-07-04
sources: 本地知识库整理
updated: 2026-07-04
---

# World Patterns — Udon 设计模式与典型场景

> VRChat World 中常用的 Udon/U# 设计模式、典型实现模式与代码模板。
>
> 本目录是 **World 域子目录**,与顶层 [[concepts/patterns/index]] 互补:
> - [[concepts/patterns/index]]: 跨域通用模式 (Networking、Data Containers、Hash Dispatch)
> - [[entities/world/patterns/index]]: **World 域特定模式** (UI、Player、Scene、Audio)

---

## 文档清单

| 文档 | 内容 | 状态 |
|------|------|------|
| [[entities/world/patterns/dynamic-player-list|dynamic-player-list]] | **动态生成玩家传送按钮** 模式 (Canvas + VRC UI Shape + TextMeshPro) | ⭐NEW 2026-07-04 |

---

## 模式分类

### UI / 交互模式

| 模式 | 文档 | 适用场景 |
|------|------|---------|
| 动态玩家列表 | [[entities/world/patterns/dynamic-player-list|dynamic-player-list]] | RPG World、社交 World、解谜 World |
| ... (待添加) | | |

---

## 与其他模式库的关系

| 模式库 | 范围 | 关系 |
|--------|------|------|
| **[[concepts/patterns/index]]** | 跨域通用 (Networking / Persistence / Data) | 通用基础 |
| **[[entities/world/patterns/index]]** (本目录) | World 域特定 (UI / Player / Scene) | 领域应用 |
| **[[entities/world/udon/index]]** | Udon 官方文档本地化 | 底层 API |
| **源资料归档层（不随发布包分发）** | 优秀开源项目模式提炼 | 案例参考 |

---

## 添加新模式

新模式创建流程:

1. 在本目录创建 `pattern-name.md`
2. 包含 frontmatter (title / category / source / tags)
3. 包含速查表 + 完整代码示例 + 陷阱分析
4. 在本文档"文档清单"中登记
5. 在 [[index]] 顶层索引中引用

---

**最后更新**: 2026-07-04
