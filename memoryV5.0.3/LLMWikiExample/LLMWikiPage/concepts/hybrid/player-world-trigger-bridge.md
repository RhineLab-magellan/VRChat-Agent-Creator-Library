---
title: "Player↔World Trigger Bridge — 区域/碰撞/粒子观测桥"
category: hybrid
knowledge_level: applied
status: active
source: "本地知识库综合整理（基于 player-collisions / player-api / 本地结构整理）"
source_type: community
version: 1.0
last_review: 2026-07-12
confidence: Medium
tags:
  - hybrid
  - player
  - world
  - trigger
  - collision
  - particle
  - udon
aliases:
  - player-world-trigger-bridge.md
  - "Trigger Bridge"
  - "玩家世界触发桥"
related:
  - avatar-world-boundaries.md
  - player-observation-boundaries.md
  - ../../entities/world/udon/players/player-collisions.md
  - ../../entities/api/player-api.md
  - ../../entities/api/dynamics.md
  - ../../entities/api/networking.md
  - avatar-world-contact-bridge.md
  - avatar-world-sync-model.md
type: concept
created: 2026-07-12
sources: "本地知识库综合整理（基于 player-collisions / player-api / 本地结构整理）"
updated: 2026-07-12
---
# Player↔World Trigger Bridge — 区域/碰撞/粒子观测桥

> 本文专门处理 Player↔World 侧的 **Trigger / Physics Collision / Particle Collision**。
>
> 它回答的问题是：
> - World 如何检测玩家进入区域？
> - World 如何检测玩家与移动物理对象碰撞？
> - World 如何检测粒子与玩家碰撞？
>
> 本文**不处理** Avatar Contact / Dynamics → World 桥；那属于 `avatar-world-contact-bridge.md` 的范围。

---

## 1. 先给结论

**[FACT]** Player↔World 这一支至少包含 3 类不同机制：
1. `Trigger`
2. `Physics Collision`
3. `Particle Collision`

**[INFERENCE]** 这三类机制不宜混写成一个笼统的“碰撞系统”。

**[INFERENCE]** 更准确的理解应是：
- Trigger = 玩家胶囊体进入/停留/离开某个世界区域
- Physics Collision = 移动物理对象与玩家发生碰撞
- Particle Collision = 粒子系统与玩家发生碰撞

---

## 2. Trigger 层：玩家进入/离开区域

### 2.1 事件

**[FACT]** World 侧存在：
- `OnPlayerTriggerEnter(VRCPlayerApi player)`
- `OnPlayerTriggerStay(VRCPlayerApi player)`
- `OnPlayerTriggerExit(VRCPlayerApi player)`

### 2.2 语义

**[FACT]** 这些事件的语义是：

> 玩家胶囊体进入 / 停留在 / 离开某个 Trigger Collider 区域。

### 2.3 适用场景

**[INFERENCE]** Trigger 更适合处理：
- 进房间
- 进区域
- 进判定带
- 本地玩家进入某功能区
- 站在某范围内持续生效的机制

### 2.4 前提配置

**[FACT]** 典型前提是：
- 世界物体上有 Collider
- `Is Trigger = true`
- 同一物体上有 UdonBehaviour

### 2.5 边缘情况

**[FACT]** 文档已提醒，以下情况可能跳过 Trigger 事件：
- 玩家传送进出 collider
- 玩家移动过快

**[INFERENCE]** 因此，不应把 Trigger Enter 当成“绝不会漏触发”的绝对机制。

---

## 3. Physics Collision 层：物理对象与玩家碰撞

### 3.1 事件

**[FACT]** World 侧存在：
- `OnPlayerCollisionEnter(VRCPlayerApi player)`
- `OnPlayerCollisionStay(VRCPlayerApi player)`
- `OnPlayerCollisionExit(VRCPlayerApi player)`
- `OnControllerColliderHitPlayer()`

### 3.2 关键限制

**[FACT]** `OnPlayerCollision*` 事件**仅对移动的物理对象**生效。

**[FACT]** 当玩家“走进静止物体”时，这些事件不会触发；这种场景应改用 Trigger。

### 3.3 正确适用场景

**[INFERENCE]** Physics Collision 更适合：
- 弹球击中玩家
- 子弹/飞行物击中玩家
- 移动机关撞到玩家
- 被物理对象碰飞或推开

### 3.4 错误适用场景

**[INFERENCE]** Physics Collision 不适合：
- 玩家走进一个固定房间
- 玩家踏入静止陷阱区
- 玩家进入静止门框或边界区

这些都应优先用 Trigger。

---

## 4. Particle Collision 层：粒子与玩家碰撞

### 4.1 事件

**[FACT]** World 侧存在：
- `OnPlayerParticleCollision(VRCPlayerApi player)`

### 4.2 典型用途

**[INFERENCE]** 适合：
- 粒子爆炸伤害
- 喷雾/毒雾/火焰类区域效果
- 粒子命中反馈

### 4.3 前置条件

**[FACT]** 文档列出的前置条件包括：
- Particle System 开启 Collision 模块
- `Send Collision Messages` 开启
- Collision 模式为 `World` + `3D`

### 4.4 性能注意

**[FACT]** 大量粒子 + 碰撞消息会产生显著性能开销。

**[INFERENCE]** 尤其在 Quest 或多人场景中，粒子碰撞应谨慎使用。

---

## 5. Trigger / Collision / Particle 三者的区别

| 机制 | 关注对象 | 典型问题 | 不适合什么 |
|---|---|---|---|
| Trigger | 玩家胶囊体进入区域 | 玩家是否进入某区域 | 精确表达 Avatar Contact 语义 |
| Physics Collision | 移动物理对象与玩家 | 玩家是否被移动物体击中 | 玩家走入静止物体 |
| Particle Collision | 粒子与玩家 | 玩家是否被粒子命中 | 大规模高频稳定区域逻辑 |

**[INFERENCE]** 这三类机制都是 Player↔World 的“观测桥”，但桥接语义不同，不应互相替代使用。

---

## 6. 与 Avatar Contact 的区别

**[FACT]** Trigger / Collision / Particle 主要围绕：
- 玩家胶囊体
- 物理对象
- 粒子系统

而不是 Avatar Contact / Dynamics 的语义层。

**[INFERENCE]** 因此：
- Trigger ≠ Contact
- 玩家走进区域 ≠ Avatar Contact 命中
- 移动物体击中玩家 ≠ Avatar PhysBone / Contact 交互

如果问题是“Avatar 的 ContactReceiver / Dynamics 怎么桥给 World”，应转到：
- `avatar-world-contact-bridge.md`

---

## 7. 与观察层、同步层的关系

### 7.1 与观察层的关系

**[INFERENCE]** Trigger / Collision / Particle 都属于：
- Player 外显状态 / 外显交互在 World 侧的事件化观测

它们与：
- `GetPosition()`
- `GetTrackingData()`
- `GetBonePosition()`

不同，后者更偏“读取状态”，前者更偏“接收事件”。

### 7.2 与同步层的关系

**[INFERENCE]** 这些事件本身不等于多人同步状态。

**[INFERENCE]** 若某次 Trigger / Collision / Particle 命中需要让多人一致可见，通常仍应把结果写入 `[UdonSynced]` 状态层，而不是只依赖本次事件触发。

---

## 8. 常见误判警告

### 8.1 误判：玩家走进静止物体会触发 `OnPlayerCollisionEnter`
错误。静止物体场景应使用 Trigger。

### 8.2 误判：Trigger 就是 Contact
错误。Trigger 主要描述玩家胶囊体进入区域；Contact 主要描述 Avatar Contact / Dynamics 层。

### 8.3 误判：`Stay` 事件可以放心做重逻辑
错误。`Stay` 是每帧每玩家调用，容易成为性能热点。

### 8.4 误判：事件触发了，Late Joiner 也会自动看到结果
错误。事件本身不是 Late Joiner-safe 状态承载。

---

## 9. 当前可稳定沉淀的分层结论

### [FACT]
1. Player↔World 侧至少存在 Trigger、Physics Collision、Particle Collision 三类不同机制。
2. `OnPlayerTriggerEnter/Stay/Exit` 用于玩家胶囊体进入/停留/离开 Trigger 区域。
3. `OnPlayerCollisionEnter/Stay/Exit` 主要适用于移动物理对象与玩家碰撞。
4. 玩家走进静止物体时，不应依赖 `OnPlayerCollision*`，应使用 Trigger。
5. `OnPlayerParticleCollision` 用于粒子与玩家碰撞。

### [INFERENCE]
1. Trigger 更像区域观测桥。
2. Physics Collision 更像移动物体击中玩家的桥。
3. Particle Collision 更像特效/伤害粒子的桥。
4. Trigger / Collision / Particle 都属于 Player↔World 侧的外显事件观测层，不等于同步层。

### [UNKNOWN]
1. 不同平台/人数/高速度条件下，这三类机制的完整鲁棒性边界。
2. 更复杂嵌套 collider / 粒子密度 / 高频传送场景的稳定性矩阵。

---

## 10. 相关资料

- `avatar-world-boundaries.md`
- `player-observation-boundaries.md`
- ``
- ``

---

## 相关页面

[[avatar-world-boundaries.md]] · [[player-observation-boundaries.md]] · [[../../entities/world/udon/players/player-collisions.md]] · [[../../entities/api/player-api.md]] · [[../../entities/api/dynamics.md]] · [[../../entities/api/networking.md]] · [[avatar-world-contact-bridge.md]] · [[avatar-world-sync-model.md]]
