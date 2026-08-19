---
title: "Networking Patterns"
category: world
knowledge_level: applied
status: active
source: udonsharp-udon.md + 本地知识库整理
source_type: community
version: 1.0
last_review: 2026-07-13
confidence: Medium
tags:
  - world
  - udon
  - networking
  - patterns
aliases:
  - "Networking Patterns"
  - "网络模式"
  - "同步模式"
  - patterns.md
related:
  - architecture.md
  - index.md
  - performance.md
  - variables.md
type: entity
created: 2026-07-13
sources: udonsharp-udon.md + 本地知识库整理
updated: 2026-07-13
---

# Networking Patterns

> 本页记录的是可复用的工程技巧 / 模式，不等同于 API 主规则页。

## 1. 变量变化主动通知订阅者
- 相比让多个行为在 `Update` 中轮询某个状态，优先考虑“状态变化时主动通知订阅者”。
- 这样更容易把网络真相层与本地表现层分离，也能减少无意义轮询开销。

## 2. PlayerId 作为轻量传输载体
- 某些场景下，跨行为 / 跨网络边界传 `PlayerId`，再本地反查 `VRCPlayerApi`，比直接试图长期持有复杂引用更稳妥。
- 但前提是：你已经设计好了玩家离开 / 重进 / id 失效时的恢复路径。

## 3. 参数压缩 / bit-pack 思路
- 对多组布尔或离散小状态，可优先考虑压缩为更紧凑的表示，再在本地展开。
- 这是一种**带宽与复杂度交换**：节省同步负担，但要求更严格的编码 / 解码一致性。

## 4. 模式使用边界
- 不要为了“看起来高级”而提前做 bit-pack；先确认同步预算真的紧张。
- 不要把 `PlayerId` 传递误当成所有玩家引用问题的万能解法。
- 不要把“主动通知”实现成另一个全局耦合中心；重点是减少无意义轮询，不是制造新单点。

---

## 相关页面

[[architecture.md]] · [[index.md]] · [[performance.md]] · [[variables.md]]
