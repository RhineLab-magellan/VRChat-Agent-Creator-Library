---
title: "Player Observation Boundaries — 玩家/Avatar 外显观察边界"
category: hybrid
knowledge_level: applied
status: active
source: "本地知识库综合整理（基于 player-positions / player-avatar-scaling / avatar-events / 本地补证整理）"
source_type: community
version: 1.0
last_review: 2026-07-12
confidence: Medium
tags:
  - hybrid
  - world
  - avatar
  - player
  - tracking
  - bone
  - scaling
  - eye-height
aliases:
  - "Player Observation Boundaries"
  - player-observation-boundaries.md
  - "玩家观察边界"
related:
  - avatar-world-boundaries.md
  - ../../entities/api/player-api.md
  - ../../entities/api/dynamics.md
  - ../../entities/world/udon/players/player-positions.md
  - ../../entities/world/udon/players/player-avatar-scaling.md
  - ../../entities/world/udon/avatar-events.md
  - ../../entities/avatar/full-body-tracking.md
  - ../../entities/avatar/vrcraycast.md
  - player-world-trigger-bridge.md
  - avatar-world-contact-bridge.md
  - avatar-world-sync-model.md
type: concept
created: 2026-07-12
sources: "本地知识库综合整理（基于 player-positions / player-avatar-scaling / avatar-events / 本地补证整理）"
updated: 2026-07-12
---
# Player Observation Boundaries — 玩家/Avatar 外显观察边界

> 本文专门回答：**World 到底能观察玩家/Avatar 的哪些外显状态？**
>
> 它是 `avatar-world-boundaries.md` 的下游细分文件，聚焦：
> - 位置 / 旋转
> - Tracking Data
> - 骨骼位置 / 旋转
> - Eye Height / Avatar Scaling
> - FBT 对观测解释的影响
>
> 本文不处理：
> - Contact 事件桥接细节
> - Trigger / Collision 细节
> - 网络同步模型

---

## 1. 先给结论

**[FACT]** World 已知可以观察到以下玩家/Avatar 相关能力：
- 玩家位置与旋转
- 有限枚举的 tracking points
- Avatar 指定骨骼的位置与旋转
- 当前 Eye Height
- Avatar / Eye Height 变化事件

**[FACT]** 这些能力并不等于：
- World 可读取 Avatar 内部 Expression Parameters / Animator 参数
- World 可稳定重建玩家真实身体结构
- World 已被证实能读取全部 FBT tracker 原始数据
- Avatar Scaling 会改变与环境碰撞

**[INFERENCE]** 更准确的理解应是：

> World 观察到的是玩家/Avatar 的**外显、可投影到世界交互层的状态**，不是 Avatar 内部逻辑状态本身。

---

## 2. 位置与旋转层

### 2.1 基础能力

**[FACT]** World 可通过：
- `GetPosition()`
- `GetRotation()`

读取玩家的位置与旋转。

### 2.2 这一层能说明什么

**[INFERENCE]** 这意味着 World 至少能回答：
- 玩家当前在哪里
- 玩家整体朝向大致是什么
- 玩家是否移动、是否发生空间位移

### 2.3 这一层不能说明什么

**[INFERENCE]** 仅凭 `GetPosition()` / `GetRotation()`：
- 不能得到手部细节
- 不能得到骨骼细节
- 不能得到 Avatar 内部参数
- 不能直接推出真实身体比例

---

## 3. Tracking Data 层

### 3.1 当前稳证支持的 TrackingDataType

**[FACT]** 当前可读的 tracking points 为：
- `Head`
- `LeftHand`
- `RightHand`
- `Origin`
- `AvatarRoot`

通过：
- `GetTrackingData(TrackingDataType)`

返回 `position` 与 `rotation`。

### 3.2 `Origin` 与 `AvatarRoot` 不同

**[FACT]** `Origin` 与 `AvatarRoot` 语义不同：
- `Origin`：本地 VR 用户为 playspace 中心；本地桌面用户为玩家位置；所有远程用户为玩家位置
- `AvatarRoot`：Avatar 根 Transform（与玩家胶囊体附着的同一 Transform）

### 3.3 本地玩家 vs 远程玩家

**[FACT]** 文档明确区分：
- 本地玩家 Tracking 数据来自 `TrackingManager`（直接来自头显/追踪器）
- 远程玩家 Tracking 数据来自 Avatar 的头/手骨骼

### 3.4 对 FBT 的一个关键限制

**[FACT]** 对全身追踪玩家，`GetTrackingData(AvatarRoot).rotation` **不会**随头部朝向旋转。

**[FACT]** 当需要“玩家大致朝向”时，应优先用：
- `GetRotation()`

而不是：
- `GetTrackingData(AvatarRoot).rotation`

### 3.5 这一层的正确边界

**[INFERENCE]** 这说明：
- World 能获取一组**有限的官方定义 tracking points**
- 但这不等于 World 可读取“全身所有追踪器原始流”
- 尤其远程玩家 tracking 的直接事实基础，是其数据来自 Avatar 的头/手骨骼；因此它不应等同于对方本地硬件追踪原始值

### 3.6 当前仍属未知的部分

**[UNKNOWN]** 当前不能强写为 FACT 的说法包括：
- World 可直接读取所有 FBT tracker 原始位置/旋转
- World 可读取完整追踪器拓扑
- 所有 tracking points 在所有平台和网络条件下都同等可靠

---

## 4. 骨骼观察层

### 4.1 已确认可用的骨骼 API

**[FACT]** World 可通过以下 API 读取 Avatar 指定骨骼：
- `GetBonePosition(HumanBodyBones)`
- `GetBoneRotation(HumanBodyBones)`

### 4.2 这意味着什么

**[FACT]** “World 可观察骨骼”已经可以写成稳定结论：

> World 确实可以读取 Avatar 指定骨骼的位置与旋转。

这比“只能看到头手 tracking”更强一层。

### 4.3 但骨骼可读不等于可稳定解释身体真相

**[FACT]** 文档明确警告：
- 不同 Avatar 的骨骼位置可能差异极大
- 不要基于骨骼位置强推身高或姿态
- 不同 Avatar 的 `Head` 骨骼不一定在预期位置

### 4.4 正确理解

**[INFERENCE]** 因此更准确的边界是：
- 骨骼**可读**
- 但“真实身体结构/姿态/身高”**不可直接稳定反推**

这两件事必须严格区分。

### 4.5 当前仍未知的部分

**[UNKNOWN]** 目前仍缺更强材料来确认：
- 所有 `HumanBodyBones` 是否同样可靠
- Quest / 桌面 / VR / FBT 状态下骨骼数据差异矩阵
- 远程骨骼更新频率与同步精度边界
- “全部动骨 / 全量骨骼信息”是否稳定可观测

---

## 5. Eye Height / Avatar Scaling 层

### 5.1 可观察能力

**[FACT]** World 可通过：
- `GetAvatarEyeHeightAsMeters()`

读取目标玩家当前 Eye Height。

**[FACT]** 该函数对：
- 本地玩家有效
- 远程玩家也有效

### 5.2 变化事件

**[FACT]** World 可响应：
- `OnAvatarChanged(...)`
- `OnAvatarEyeHeightChanged(...)`

来观察 Avatar / Eye Height 变化。

### 5.3 时序限制

**[FACT]** `avatar-events.md` 已明确提醒：
- 在 `OnAvatarChanged` 中，不应依赖远程玩家新 Eye Height 已经同步完成
- 远程玩家的 `OnAvatarEyeHeightChanged` 可能多次触发
- 远程玩家的 `OnAvatarEyeHeightChanged` 可能早于 `OnAvatarChanged`

### 5.4 控制函数边界

**[FACT]** 大部分 Avatar Scaling 控制函数仅对本地玩家有效，例如：
- `SetManualAvatarScalingAllowed`
- `SetAvatarEyeHeightMinimumByMeters`
- `SetAvatarEyeHeightMaximumByMeters`
- `SetAvatarEyeHeightByMeters`
- `SetAvatarEyeHeightByMultiplier`

### 5.5 关键硬边界

**[FACT]** Avatar Scaling **不会改变与环境的碰撞**。

### 5.6 正确理解

**[INFERENCE]** 因此应把这一层理解为：
- World 可以观察和部分控制**缩放语义层**（尤其 Eye Height）
- 但不能把“变大/变小”直接等价为“碰撞体同步变化”
- 也不能把“可读当前眼高”误解成“任意远程缩放完全可控”

### 5.7 当前未知部分

**[UNKNOWN]** 当前仍未稳证：
- 是否存在统一稳定的远程玩家“缩放倍率”读取接口（不同于 Eye Height）
- 极端网络条件下 Avatar / Eye Height 事件的完整时序保证矩阵

---

## 6. FBT 对观察层的影响

### 6.1 已知事实

**[FACT]** FBT（Full-Body Tracking）说明 VRChat 的 Avatar / 用户侧存在更丰富的追踪器生态。

**[FACT]** 但目前已知能直接进入 World 侧观察接口并被本轮稳证支持的，仍主要是：
- TrackingData 的有限枚举点
- Bone API
- Eye Height / Avatar Events

### 6.2 FBT 给我们的真正启示

**[INFERENCE]** FBT 在本主题中的价值，更像是：
- 说明 Avatar / 用户端可能有更复杂追踪结构
- 说明 `AvatarRoot` 的解释不能偷懒
- 说明“世界里能看见的姿态结果”和“底层硬件追踪拓扑”不是一回事

### 6.3 当前仍未知的部分

**[UNKNOWN]** 还不能从 FBT 文档直接推出：
- World 可直接读取全部 FBT tracker 原始数据
- World 可完整掌握玩家所有追踪节点

---

## 7. VRCRaycast 为什么不能算作 World 观察层证据

**[FACT]** `VRCRaycast` 是 Avatar 端组件。

**[FACT]** 它的结果写入 Avatar Animator 参数，而不是 World/Udon 侧读取接口。

**[INFERENCE]** 所以它更适合作为一个反证：
- Avatar 自己可以感知世界并驱动自身动画
- 但这不等于 World 被开放读取 Avatar 内部状态的能力

---

## 8. 当前可稳定沉淀的分层结论

### [FACT]
1. World 可读取玩家位置与旋转。
2. World 可读取有限枚举 tracking points：`Head` / `LeftHand` / `RightHand` / `Origin` / `AvatarRoot`。
3. World 可读取 Avatar 指定骨骼的位置与旋转。
4. World 可读取目标玩家当前 Eye Height。
5. World 可响应 Avatar / Eye Height 变化事件。
6. Avatar Scaling 不会改变与环境的碰撞。

### [INFERENCE]
1. World 观察到的是玩家/Avatar 的外显状态，而非 Avatar 内部参数。
2. 远程 tracking 的直接事实基础，是其数据来自 Avatar 头/手骨骼；因此它不应等同于对方硬件原始追踪值。
3. 骨骼可读不等于真实身体结构可可靠还原。
4. Eye Height 是比骨骼位置更适合作为缩放语义层的观测量，但仍受事件时序约束影响。

### [UNKNOWN]
1. World 是否可直接读取所有 FBT 额外 tracker 原始位置/旋转。
2. “全部动骨 / 全量骨骼信息”是否真的可稳定观测。
3. 所有骨骼在不同 Avatar / 平台 / 网络条件下是否同等可靠。
4. 是否存在稳定统一的远程玩家缩放倍率读取方式。

---

## 9. 常见误判警告

### 9.1 误判：tracking 可读 = 全部追踪器原始数据可读
错误。当前稳证只到有限枚举 tracking points。

### 9.2 误判：骨骼可读 = 真实身体结构可还原
错误。骨骼可读，但 Avatar 骨架差异会严重影响解释。

### 9.3 误判：Eye Height 可读 = 任意远程缩放完全可控
错误。大部分控制函数仅本地玩家有效。

### 9.4 误判：Avatar 变大/变小 = 碰撞体也一起变化
错误。Avatar Scaling 不改变与环境碰撞。

---

## 10. 与其他文档的关系

- `avatar-world-boundaries.md`：本主题总纲
- `player-world-trigger-bridge.md`：未来专门承载 Trigger / Collision / Particle 细节
- `avatar-world-contact-bridge.md`：未来专门承载 Contact / Dynamics → World 桥接
- `avatar-world-sync-model.md`：未来专门承载同步建模

---

## 11. 相关资料

- [[entities/api/player-api]]
- [[entities/world/udon/players/player-positions]]
- [[entities/world/udon/players/player-avatar-scaling]]
- [[entities/world/udon/avatar-events]]
- [[entities/avatar/full-body-tracking]]
- [[entities/avatar/vrcraycast]]
- `（来源：本地网络搜索存档）

---

## 相关页面

[[avatar-world-boundaries.md]] · [[../../entities/api/player-api.md]] · [[../../entities/api/dynamics.md]] · [[../../entities/world/udon/players/player-positions.md]] · [[../../entities/world/udon/players/player-avatar-scaling.md]] · [[../../entities/world/udon/avatar-events.md]] · [[../../entities/avatar/full-body-tracking.md]] · [[../../entities/avatar/vrcraycast.md]] · [[player-world-trigger-bridge.md]] · [[avatar-world-contact-bridge.md]] · [[avatar-world-sync-model.md]]
