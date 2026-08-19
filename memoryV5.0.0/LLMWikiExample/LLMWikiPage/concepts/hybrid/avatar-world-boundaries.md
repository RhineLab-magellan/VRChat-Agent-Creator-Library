---
title: "Avatar↔World Boundaries — 架构边界总纲"
category: hybrid
knowledge_level: applied
status: active
source: "本地知识库综合整理（基于本地 memory 交叉核对 + 参考层结构整理）"
source_type: community
version: 1.0
last_review: 2026-07-12
confidence: Medium
tags:
  - hybrid
  - avatar
  - world
  - udon
  - tracking
  - contact
  - networking
aliases:
  - "Avatar World Boundaries"
  - "Avatar↔World"
  - avatar-world-boundaries.md
related:
  - index.md
  - ../../entities/api/player-api.md
  - ../../entities/api/dynamics.md
  - ../../entities/api/networking.md
  - ../../entities/world/udon/avatar-events.md
  - ../../entities/world/udon/players/player-positions.md
  - ../../entities/world/udon/players/player-avatar-scaling.md
  - ../../entities/world/udon/players/player-collisions.md
  - ../../entities/avatar/contact.md
  - player-observation-boundaries.md
  - player-world-trigger-bridge.md
  - avatar-world-contact-bridge.md
  - avatar-world-sync-model.md
  - ../../entities/avatar/vrcraycast.md
type: concept
created: 2026-07-12
sources: "本地知识库综合整理（基于本地 memory 交叉核对 + 参考层结构整理）"
updated: 2026-07-12
---
# Avatar↔World Boundaries — 架构边界总纲

> 本文是 `Avatar↔World` 主题的**总纲 / 路由页 / 边界声明页**。
> 目标不是穷举全部 API，而是先回答：
> 1. World 能看到什么？
> 2. World 看不到什么？
> 3. Avatar/Player → World 的正式桥接层是什么？
> 4. 为什么交互触发不等于多人同步状态？

---

## 1. 概述

`Avatar` 与 `World` 在 VRChat 中至少应视为**边界清晰的独立系统**；`Item` 可作为扩展理解，但不应在当前文档中被视为同等强度的已证实并列项。

这意味着：
- 不能把“Avatar 能做什么”直接等同于“World 能读到什么”
- 不能把“World 能观察到玩家外显状态”直接等同于“World 能读取 Avatar 内部参数”
- 不能把“某种交互事件触发了”直接等同于“多人状态已经同步完成”

因此，分析 `Avatar↔World` 边界时，至少要拆成 4 层：

1. **内部参数层**：World 是否能直接读取 Avatar 内部参数
2. **外显观察层**：World 能观察哪些玩家/Avatar 外显状态
3. **正式桥接层**：Avatar/Player → World 通过哪些事件桥发生交互
4. **状态同步层**：交互结果如何变成多人一致、Late Joiner-safe 的世界状态

---

## 2. 核心命题一：World 是否能直接读取 Avatar 内部参数？

**[FACT]** 在当前已核对的本地资料中，尚未发现 World/Udon 直接读取 Avatar Expression Parameters / Animator 内部参数的接口证据。

**[INFERENCE]** 已核对到的 Contact / Dynamics / VRCRaycast 等材料，当前都不能作为“World 直接读取 Avatar 内部参数”的正向证据：
- Contact / Dynamics 给到 World 的是事件与上下文信息
- VRCRaycast 是 Avatar 端组件，结果写入 Avatar Animator 参数，不是 World/Udon 侧读取接口

**[INFERENCE]** 目前更准确的工程表述不是“World 被明令禁止读取 Avatar 内部参数”，而是：

> **World 没有被提供直接访问 Avatar 内部 Expression Parameters / Animator 内部参数的接口。**

**[UNKNOWN]** 当前仍缺一份更直接的官方边界文档，把“无接口”升级成更强的封口式表述。

---

## 3. 核心命题二：World 能观察哪些玩家/Avatar 外显状态？

### 3.1 已稳定成立的观察能力

**[FACT]** World 可通过 `VRCPlayerApi` 读取玩家位置与旋转：
- `GetPosition()`
- `GetRotation()`

**[FACT]** World 可通过 `GetTrackingData(...)` 读取有限枚举的 tracking points：
- `Head`
- `LeftHand`
- `RightHand`
- `Origin`
- `AvatarRoot`

**[FACT]** World 可通过以下 API 读取 Avatar 指定骨骼：
- `GetBonePosition(HumanBodyBones)`
- `GetBoneRotation(HumanBodyBones)`

**[FACT]** World 可观察当前 Eye Height，并响应 Avatar / Eye Height 变化：
- `GetAvatarEyeHeightAsMeters()`
- `OnAvatarChanged(...)`
- `OnAvatarEyeHeightChanged(...)`

### 3.2 观察能力的限制

**[FACT]** “骨骼可读”不等于“可稳定推断真实身体结构/姿态/身高”。文档已明确警告，不同 Avatar 的骨骼位置可能差异极大。

**[FACT]** `Avatar Scaling` 不会改变与环境的碰撞。

**[INFERENCE]** 因此，World 已知能观察的是：
- 玩家是谁
- 玩家在哪里
- 玩家头手/有限 tracking points 在哪里
- Avatar 指定骨骼的位置与旋转
- 玩家当前 Eye Height 与 Avatar 变化
- 玩家是否与某些区域/接触体/物理对象发生交互

而不是：
- Avatar 内部参数本身
- 玩家“真实身体结构”的可靠重建结果
- 已被证实的“全部动骨 / 全量骨骼 / 全量 FBT 原始 tracker”数据

**[UNKNOWN]** 当前仍不能写成强结论的包括：
- World 可直接读取所有 FBT 额外 tracker 原始位置/旋转
- World 已能稳定获得全部动骨 / 全量骨骼信息
- 所有骨骼在所有 Avatar / 平台 / 网络条件下都同等可靠

---

## 4. 核心命题三：Avatar/Player → World 的正式桥接层有哪些？

当前至少已确认 3 类：

### 4.1 Player Trigger Bridge
**[FACT]** World 侧存在：
- `OnPlayerTriggerEnter`
- `OnPlayerTriggerStay`
- `OnPlayerTriggerExit`

其语义是：**玩家胶囊体进入/停留/离开 World Trigger 区域**。

### 4.2 Avatar Contact / Dynamics Bridge
**[FACT]** World 侧存在：
- `OnContactEnter`
- `OnContactExit`
- `OnContactStay`

其语义是：**Avatar Contact / Dynamics 层把接触事实桥接给 World**。

### 4.3 Player Observation Layer
**[FACT]** World 还可通过：
- `VRCPlayerApi`
- Avatar Events
- Tracking / Bone / Eye Height API

观察玩家/Avatar 的外显状态。

**[INFERENCE]** 因此，“Contact 是唯一桥接层”这个说法不成立。更准确的理解是：
- Trigger = 玩家区域进入桥
- Contact = Avatar Contact / Dynamics 桥
- Observation = 玩家/Avatar 外显状态观察层

---

## 5. 核心命题四：为什么交互触发不等于多人同步状态？

**[FACT]** Trigger / Contact / Avatar Event / Tracking / Bone 观察本身都只是**信号**，不自动等于多人一致状态。

**[FACT]** `SendCustomNetworkEvent` 属于事件层，不适合作为 Late Joiner-safe 的持续状态承载。

**[FACT]** `[UdonSynced]` + 反序列化链路，才是多人一致与 Late Joiner 恢复的基础状态承载方式。

**[INFERENCE]** 更稳定的工程模型应是：

```text
Avatar / Player 外显行为
  → World 收到 Trigger / Contact / Tracking / Bone / Avatar Event
  → Owner 或业务权威脚本解释该信号
  → 如有持续状态：写入 [UdonSynced]
  → RequestSerialization()
  → 远端玩家 / Late Joiner 重建表现
```

因此：
- **触发** ≠ **同步**
- **观察到变化** ≠ **所有玩家都已看到同一结果**
- **收到 Event** ≠ **Late Joiner 也能恢复该状态**

---

## 6. 当前可稳定沉淀的结论分层

### [FACT]
1. 在当前已核对的本地资料中，未发现 World/Udon 直接读取 Avatar 内部参数的接口证据。
2. World 可读取玩家位置、旋转、有限 tracking points。
3. World 可读取 Avatar 指定骨骼的位置与旋转。
4. World 可读取目标玩家当前 Eye Height，并响应 Avatar / Eye Height 变化事件。
5. Avatar Scaling 不会改变与环境的碰撞。
6. World 存在 Player Trigger、Physics Collision、Particle Collision、Contact 等正式交互/观测机制。
7. Event 不等于 Late Joiner-safe 持续状态；`[UdonSynced]` 才是状态承载核心。

### [INFERENCE]
1. Avatar 与 World 至少应按边界清晰的独立系统理解；Item 可作为扩展理解，但当前直接支撑较弱。
2. “无接口证据”比“明令禁止”更准确地描述当前 World 无法直接读取 Avatar 内部参数的原因。
3. Trigger / Contact / Observation / Sync 可视为不同分析层次，讨论时不宜混写。
4. 骨骼可读不等于真实身体结构可可靠还原。
5. 远程玩家 tracking 更接近由 Avatar 骨骼回映出的可观测结果，而不是本地硬件原始追踪流。

### [UNKNOWN]
1. World 是否可直接读取所有 FBT 额外 tracker 的原始位置/旋转。
2. “全部动骨 / 全量骨骼信息”是否真的可稳定观测。
3. 是否存在更直接的官方封口文档来描述 Avatar 内部参数边界。
4. Pickup / Station 等其他机制在 Avatar↔World 桥接总图中的最终位置。

---

## 7. 常见误判警告

### 7.1 误判：能看到骨骼 = 能准确判断真实身体姿态
错误。骨骼可读，但不同 Avatar 骨架差异会显著影响解释。

### 7.2 误判：Avatar 变大/变小 = 碰撞体一起变
错误。Avatar Scaling 不改变与环境碰撞。

### 7.3 误判：Contact 触发了 = 状态已经网络同步
错误。Contact 是桥接信号，不是状态承载层。

### 7.4 误判：VRCRaycast 证明 World 可以读取 Avatar 内部状态
错误。VRCRaycast 是 Avatar 端组件，结果进入 Avatar Animator 参数，不是 World/Udon 读取接口。

---

## 8. 建议的下游文件

本文件是总纲。后续建议逐步拆出：

1. `player-observation-boundaries.md`
   - 位置 / tracking / bone / eye height / scaling 细节

2. `player-world-trigger-bridge.md`
   - Trigger / Collision / Particle 的细节与对照

3. `avatar-world-contact-bridge.md`
   - Contact / Dynamics → World 的桥接机制

4. `avatar-world-sync-model.md`
   - Trigger / Contact / Observation 如何转成多人一致状态

---

## 9. 相关资料

- [[concepts/hybrid/index]]
- [[entities/api/player-api]]
- [[entities/world/udon/avatar-events]]
- [[entities/world/udon/players/player-positions]]
- [[entities/world/udon/players/player-avatar-scaling]]
- [[entities/world/udon/players/player-collisions]]
- [[entities/avatar/vrcraycast]]
- `（来源：本地网络搜索存档）
- `（来源：本地网络搜索存档）
- `（来源：本地网络搜索存档）

---

## 相关页面

[[index.md]] · [[../../entities/api/player-api.md]] · [[../../entities/api/dynamics.md]] · [[../../entities/api/networking.md]] · [[../../entities/world/udon/avatar-events.md]] · [[../../entities/world/udon/players/player-positions.md]] · [[../../entities/world/udon/players/player-avatar-scaling.md]] · [[../../entities/world/udon/players/player-collisions.md]] · [[../../entities/avatar/contact.md]] · [[player-observation-boundaries.md]] · [[player-world-trigger-bridge.md]] · [[avatar-world-contact-bridge.md]] · [[avatar-world-sync-model.md]] · [[../../entities/avatar/vrcraycast.md]]
