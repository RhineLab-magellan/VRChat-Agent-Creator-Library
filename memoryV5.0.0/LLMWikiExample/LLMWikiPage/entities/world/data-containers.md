---
title: Data Containers & VRCJson ⚠️ 已迁移
category: world

knowledge_level: applied
status: deprecated

tags:
  - world
  - deprecated
  - udon
  - data-containers
  - vrcjson

aliases:
  - "Data Containers"
  - 数据容器
  - VRCJson
  - DataToken

related:
  - udon/data-containers/index.md
  - udon/data-containers/byte-and-bit-operations.md
  - ../../concepts/patterns/bit-packed-flags.md

source: 本地知识库整理
source_type: community
version: 1.2
last_review: 2026-07-01
confidence: Medium
type: entity
created: 2026-07-01
sources: 本地知识库整理
updated: 2026-07-01
---

> ⚠️ **DEPRECATED — 已迁移**
> 本文件内容已迁移到 [`udon/data-containers/`](udon/data-containers/index.md)。
> 保留此文件仅为防止断链。请使用新位置。

---

## 📚 权威文档位置

| 主题 | 权威文档 |
|------|---------|
| **Data Containers 总览** | [[entities/world/udon/data-containers/index]] |
| **Byte/Bit Operations（位域压缩）** | [[entities/world/udon/data-containers/byte-and-bit-operations]] |
| **位域压缩 Pattern** | [[concepts/patterns/bit-packed-flags]] |

## 关键要点（已覆盖在权威文档中）

- **DataToken 类型**: Null/Boolean/Byte/Int/Float/String/DataList/DataDictionary/Reference/DataError
- **DataList / DataDictionary**: 基础操作、克隆、JSON 序列化
- **JSON 限制**: 仅字符串键、根只能是 List/Dict、不支持 Reference、不支持 NaN/Infinity
- **位域压缩**: 8 个 bool → 1 个 byte，节省 87.5% 序列化空间
- **核心工具**: `DataToken.Bitcast()` / `System.BitConverter` / `System.Buffer`
