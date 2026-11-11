---
title: "Network Sync Architecture"
category: world
knowledge_level: applied
status: active
source: networking.md + 本地知识库整理
source_type: community
version: 1.0
last_review: 2026-07-13
confidence: Medium
tags:
  - world
  - udon
  - networking
  - architecture
aliases:
  - "Network Sync Architecture"
  - "网络同步架构"
  - "同步架构"
  - architecture.md
related:
  - index.md
  - network-details.md
  - ownership.md
  - variables.md
  - performance.md
type: entity
created: 2026-07-13
sources: networking.md + 本地知识库整理
updated: 2026-07-13
---

# Network Sync Architecture

> ⚠️ 本页只吸收**架构模式与设计问题清单**。与现有 memory 冲突的旧数字、旧语义、未验证反外挂段落不纳入。

## 1. 常见架构分型
### 1.1 单文件同步
- 把核心同步状态集中在少量 UdonBehaviour / 少量同步变量中。
- 适合小型系统、状态面有限的玩法。
- 风险是后期容易演化成“万能同步脚本”。

### 1.2 同步隔离
- 把不同职责拆到不同同步单元：
  - 玩家状态
  - 交互物件状态
  - UI / 本地展示状态
- 目的是减少无关状态耦合，避免每次序列化都带上所有东西。

### 1.3 同步管理模式
- 由一个中心管理器负责关键同步决策，而把局部表现下放给局部对象。
- 适合需要统一裁决、统一恢复 late joiner 状态的场景。
- 风险是中心管理器过载，因此只应让它持有“必要真相”，而不是所有表现细节。

### 1.4 同步对象池模式
- 对可重复生成 / 回收的对象，不要只关注“生成逻辑”，也要提前设计对象重置与所有权重置。
- 池化对象若带同步状态，必须明确：回收时谁清理、复用时谁初始化、late joiner 如何看见正确状态。

## 2. 设计前先回答的问题
1. **哪些状态是真正需要跨客户端一致的？**
2. **哪些只是本地表现，不应占用同步预算？**
3. **状态是离散的还是连续的？**
4. **late joiner 必须看到什么，允许错过什么？**
5. **谁拥有写权限 / 裁决权？**
6. **对象回收 / 复用后如何保证旧状态不会泄漏？**
7. **这个系统真的需要更多同步变量，还是应该先重构职责边界？**

## 3. 设计边界提醒
- 不要把“高频变化”自动等同于 Continuous；先问中间帧是否真的重要。
- 不要把网络事件当成 late joiner 状态恢复机制。
- 不要让中心管理器同时承担“真相源 + 全部表现逻辑 + 全部 UI 转发”。
- 不要在缺少状态边界设计时就堆更多同步变量。

---

## 相关页面

[[index.md]] · [[network-details.md]] · [[ownership.md]] · [[variables.md]] · [[performance.md]]
