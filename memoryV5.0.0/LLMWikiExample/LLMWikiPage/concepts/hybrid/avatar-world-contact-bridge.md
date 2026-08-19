---
title: "Avatar↔World Contact Bridge — Contact / Dynamics 桥接边界"
category: hybrid
knowledge_level: applied
status: active
source: "本地知识库综合整理（基于 dynamics / vrcraycast / 本地结构整理）"
source_type: community
version: 1.0
last_review: 2026-07-12
confidence: Medium
tags:
  - hybrid
  - avatar
  - world
  - contact
  - dynamics
  - physbone
  - udon
aliases:
  - avatar-world-contact-bridge.md
  - "Contact Bridge"
  - "Avatar Contact Bridge"
related:
  - avatar-world-boundaries.md
  - player-observation-boundaries.md
  - player-world-trigger-bridge.md
  - ../../entities/api/dynamics.md
  - ../../entities/api/networking.md
  - ../../entities/avatar/contact.md
  - ../../entities/avatar/vrcraycast.md
  - avatar-world-sync-model.md
type: concept
created: 2026-07-12
sources: "本地知识库综合整理（基于 dynamics / vrcraycast / 本地结构整理）"
updated: 2026-07-12
---
# Avatar↔World Contact Bridge — Contact / Dynamics 桥接边界

> 本文专门处理 **Avatar Contact / Dynamics → World** 的桥接问题。
>
> 它回答的问题是：
> - Avatar 的 Contact / Dynamics 如何把接触事实桥给 World？
> - World 侧实际拿到的是什么数据？
> - Contact 与 Trigger / Collision / Avatar 内部参数分别有什么区别？
>
> 本文**不处理**：
> - Player Trigger / Collision / Particle 的细节
> - 网络同步建模
> - Avatar 内部参数本身如何工作

---

## 1. 先给结论

**[FACT]** Avatar↔World 之间存在一类正式的 Avatar Contact / Dynamics → World 事件桥。

**[FACT]** World 侧可通过：
- `OnContactEnter`
- `OnContactExit`
- `OnContactStay`

接收来自 Avatar Contact / Dynamics 层的接触事件。

**[FACT]** World 侧拿到的是：
- 事件
- 至少一部分接触上下文
- 玩家来源信息（若可用）

而不是：
- Avatar Expression Parameters
- Avatar Animator 内部参数表
- Avatar 内部逻辑状态的直接读取权限

**[INFERENCE]** Contact 是一座“桥”，但它桥接的是**接触事实**，不是 Avatar 内部参数开放。

---

## 2. World 侧可用的 Contact / Dynamics 事件

### 2.1 Contact 事件

**[FACT]** World-side Dynamics 暴露：
- `OnContactEnter(ContactEnterInfo info)`
- `OnContactExit(...)`
- `OnContactStay(...)`

### 2.2 PhysBone 相关事件

**[FACT]** World-side Dynamics 还暴露：
- `OnPhysBoneGrabbed(PhysBoneGrabbedInfo info)`
- `OnPhysBoneReleased(...)`

这说明 World 不只是能感知 Contact，也能感知部分 PhysBone 相关交互事实。

### 2.3 SDK 版本边界

**[FACT]** 这些 World-side Dynamics 能力属于 SDK 3.10.0+ 引入的世界端 Dynamics 能力。

---

## 3. World 侧实际拿到什么

### 3.1 ContactEnterInfo

**[FACT]** 已核对到的 World-side Contact 信息至少包括：
- `info.isAvatar`
- `info.player`（可能为 null）

### 3.2 参考层补充到的上下文

**[INFERENCE]** 在已整理的 Contact 资料中，World Contact 侧常见还会围绕：
- `player`
- `contactPoint`
- `collisionTags`
- `isAvatar`
- `isLocal`

这些属于“接触上下文”与“来源上下文”。

### 3.3 这说明什么

**[INFERENCE]** Contact / Dynamics 桥给 World 的，是：
- 接触发生了没
- 接触来自 Avatar 还是别的来源
- 大致由哪个玩家触发
- 接触点/标签等上下文

而不是 Avatar 内部控制参数本身。

---

## 4. Contact 桥到底桥接了什么

### 4.1 正确理解

**[INFERENCE]** Contact 桥接的是：

> **Avatar 的外显接触事实 → World 侧可消费的事件信号**

也就是说，World 看到的是“Avatar 与世界发生了某种接触”，而不是“World 打开了 Avatar 内部参数面板”。

### 4.2 这座桥的意义

**[INFERENCE]** 它的真正价值在于：
- 让 Avatar 与 World 不再完全隔绝
- 让 World 能对 Avatar 的接触作出逻辑反应
- 让 Avatar↔World 的交互具备正式事件入口

但它仍然维持了系统边界：
- World 知道“发生了接触”
- 不代表 World 因此获得 Avatar 内部状态读取权

---

## 5. Contact 与 Trigger 的区别

### 5.1 Trigger 的对象

**[FACT]** Trigger 更偏：
- 玩家胶囊体进入/离开区域
- 关注的是玩家整体进入某个世界判定区

### 5.2 Contact 的对象

**[FACT]** Contact 更偏：
- Avatar Contact / Dynamics 层与世界发生接触
- 关注的是 Avatar 触发出来的接触事实

### 5.3 正确区分

**[INFERENCE]** 因此：
- 玩家走进一个房间 → 更像 Trigger 问题
- Avatar 的 ContactReceiver 命中世界对象 → 更像 Contact 桥问题

两者不是同一层，也不应互相替代。

---

## 6. Contact 与 Avatar 内部参数的区别

### 6.1 当前稳证边界

**[FACT]** 当前已核对到的资料中，未发现通过 Contact 直接读取 Avatar Expression Parameters / Animator 参数的接口证据。

**[FACT]** VRCRaycast 也不能作为这种证据，因为：
- 它是 Avatar 端组件
- 结果写入 Avatar Animator 参数
- 不是 World/Udon 侧读取接口

### 6.2 正确理解

**[INFERENCE]** 因此，Contact 的角色是：
- 提供“接触事实桥接”

而不是：
- 开放 Avatar 内部参数访问

### 6.3 当前仍未知的部分

**[UNKNOWN]** 目前仍缺更直接的官方封口文档，把“Contact 不等于参数访问”表述成更强的官方边界句。

---

## 7. Contact 与 Observation 层的关系

**[INFERENCE]** Contact 与 Observation 的区别可粗略理解为：
- Observation 层：World 主动读取玩家/Avatar 外显状态
- Contact 层：World 被动收到 Avatar 接触事件

例如：
- `GetBonePosition(...)` → 读取状态
- `OnContactEnter(...)` → 接收事件

两者都属于“Avatar↔World 边界”的一部分，但角色不同。

---

## 8. Contact 与 Sync 层的关系

**[FACT]** Contact 事件本身不是多人一致状态承载层。

**[INFERENCE]** 如果某次 Contact 命中后要让：
- 其他玩家都看到结果
- Late Joiner 也恢复结果

则仍需把结果转进：
- `[UdonSynced]`
- Owner 权威逻辑
- 反序列化恢复链路

因此：
- Contact = 信号入口
- Sync = 状态承载

这两者必须分开。

---

## 9. 常见误判警告

### 9.1 误判：Contact = Trigger
错误。Trigger 更偏玩家胶囊体区域事件；Contact 更偏 Avatar Contact / Dynamics 接触事实。

### 9.2 误判：Contact 命中 = World 已能读取 Avatar 参数
错误。Contact 桥接的是接触事实，不是参数读取权限。

### 9.3 误判：有了 Contact，就不需要同步建模
错误。Contact 只是信号，不是多人一致状态承载层。

### 9.4 误判：VRCRaycast 证明 World 可以读 Avatar 内部状态
错误。VRCRaycast 是 Avatar 端组件，结果在 Avatar Animator 内部消费。

---

## 10. 当前可稳定沉淀的分层结论

### [FACT]
1. Avatar↔World 之间存在正式的 Contact / Dynamics 桥。
2. World 侧可通过 `OnContactEnter/Exit/Stay` 接收接触事件。
3. World-side Dynamics 还可接收部分 PhysBone 交互事件，如 `OnPhysBoneGrabbed`。
4. World 侧拿到的是事件与上下文；当前已核对资料未提供通过 Contact 直接读取 Avatar 内部参数的接口证据。
5. Contact 事件本身不是多人状态承载层。

### [INFERENCE]
1. Contact 桥接的是 Avatar 外显接触事实，而不是 Avatar 内部状态。
2. Contact 是 Avatar↔World 正式连接的关键桥，但仍然保留系统边界。
3. Contact 更适合作为逻辑入口，再由 World 同步层决定是否持久化和广播结果。

### [UNKNOWN]
1. Contact 相关上下文在不同实现/版本中的完整字段矩阵。
2. 是否存在更直接的官方封口文档，专门声明 Contact 不等于参数访问。
3. Contact 与更多高级 Avatar 机制（除 VRCRaycast 外）的边界说明是否还有未覆盖部分。

---

## 11. 相关资料

- `avatar-world-boundaries.md`
- `player-observation-boundaries.md`
- `player-world-trigger-bridge.md`
- `../api/dynamics.md`
- `../avatar/vrcraycast.md`

---

## 相关页面

[[avatar-world-boundaries.md]] · [[player-observation-boundaries.md]] · [[player-world-trigger-bridge.md]] · [[../../entities/api/dynamics.md]] · [[../../entities/api/networking.md]] · [[../../entities/avatar/contact.md]] · [[../../entities/avatar/vrcraycast.md]] · [[avatar-world-sync-model.md]]
