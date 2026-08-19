---
title: "Avatar↔World Sync Model — 触发、权威与可恢复状态"
category: hybrid
knowledge_level: applied
status: active
source: "本地知识库综合整理（基于 networking / 本地结构整理 / Late Joiner 模式归纳）"
source_type: inferred
version: 1.0
last_review: 2026-07-12
confidence: Medium
tags:
  - hybrid
  - world
  - avatar
  - networking
  - sync
  - owner
  - late-joiner
aliases:
  - avatar-world-sync-model.md
  - "Sync Model"
  - "Avatar World Sync"
related:
  - avatar-world-boundaries.md
  - player-observation-boundaries.md
  - player-world-trigger-bridge.md
  - avatar-world-contact-bridge.md
  - ../../entities/api/networking.md
  - ../../entities/api/dynamics.md
  - ../../entities/avatar/contact.md
  - ../../entities/world/udon/players/player-collisions.md
type: concept
created: 2026-07-12
sources: "本地知识库综合整理（基于 networking / 本地结构整理 / Late Joiner 模式归纳）"
updated: 2026-07-12
---
# Avatar↔World Sync Model — 触发、权威与可恢复状态

> 本文专门回答：
> - 为什么 Trigger / Contact / Tracking / Bone / Avatar Event 不是同步层？
> - Avatar↔World 交互结果如何变成多人一致状态？
> - Owner / Event / `[UdonSynced]` 在这里分别扮演什么角色？
>
> 本文聚焦“状态建模”，不重复说明 Trigger / Contact 的触发细节。

---

## 1. 先给结论

**[INFERENCE]** Trigger / Contact / Tracking / Bone / Avatar Event 本身更适合作为**信号/观察入口**理解，而不是多人一致状态承载层。

**[FACT]** `SendCustomNetworkEvent` 属于事件层，不适合承担 Late Joiner 需要恢复的持续状态。

**[FACT]** 对于需要多人一致与 Late Joiner 恢复的持续状态，`[UdonSynced]` + `OnDeserialization()` 是核心状态承载机制。

**[INFERENCE]** 更稳定的工程模型应是：

```text
Avatar / Player 外显行为
  → World 收到 Trigger / Contact / Tracking / Bone / Avatar Event
  → 业务权威脚本(通常 owner)解释信号
  → 若需持续状态：写入 [UdonSynced]
  → RequestSerialization()
  → 其他玩家 / Late Joiner 在 OnDeserialization() 中重建表现
```

---

## 2. 为什么“触发”不等于“同步”

### 2.1 触发层的角色

**[INFERENCE]** 以下内容都更像“输入信号”或“事件入口”：
- `OnPlayerTriggerEnter/Stay/Exit`
- `OnContactEnter/Exit/Stay`
- `OnAvatarChanged`
- `OnAvatarEyeHeightChanged`
- `GetTrackingData(...)`
- `GetBonePosition(...)`

### 2.2 触发层的局限

**[INFERENCE]** 它们只能说明：
- 某件事发生了
- 某个状态当前可被读到
- 某个玩家触发了某个条件

但不能自动保证：
- 其他玩家都看到同一结果
- 玩家晚加入时还能恢复该结果
- 状态会以相同方式在不同客户端保持一致

---

## 3. 事件层：`SendCustomNetworkEvent` 的职责边界

### 3.1 它适合什么

**[FACT]** `SendCustomNetworkEvent(...)` 是网络事件机制，可向：
- `All`
- `Owner`

发送无参方法调用。

**[INFERENCE]** 它更适合：
- 命令
- 请求
- 非 Owner → Owner 的更新请求
- 固定 Owner 架构中的操作入口
- 一次性动作触发

### 3.2 它不适合什么

**[FACT]** Network Event 不适合承担 Late Joiner 需要恢复的持续状态。

### 3.3 为什么不适合

**[INFERENCE]** 因为 Event 更像“一次调用”，而不是“可重放状态存储”。

所以：
- 事件发出时在线的人可能看到结果
- 晚加入的人不会因为这次 Event 自动补看过去发生了什么

---

## 4. 状态层：`[UdonSynced]` 才是承载中心

### 4.1 基础原则

**[FACT]** 如果某个 Avatar↔World 交互结果需要：
- 多人一致
- 状态持有
- Late Joiner 恢复

则应落到：
- `[UdonSynced]` 字段

### 4.2 Manual Sync 的关键动作

**[FACT]** 在 Manual Sync 模式下，修改 synced variable 后必须调用：
- `RequestSerialization()`

否则不会真正发出同步。

### 4.3 接收侧恢复

**[FACT]** 当远端收到 synced variable 更新时，会触发：
- `OnDeserialization()`

**[FACT]** Late Joiner 加入时也会触发 `OnDeserialization()`。

### 4.4 正确理解

**[INFERENCE]** 因此：
- Event 更像“告诉别人现在做件事”
- `[UdonSynced]` 更像“把世界当前状态写成共享事实”

---

## 5. Owner 在 Avatar↔World 同步中的角色

### 5.1 为什么需要 Owner

**[FACT]** Networking 中存在 Owner 概念：
- `Networking.GetOwner(...)`
- `Networking.IsOwner(...)`
- `Networking.SetOwner(...)`

### 5.2 典型权威模型

**[INFERENCE]** 在 Avatar↔World 交互里，常见模型是：
- 某个世界对象由一个 owner 作为“状态权威”
- 其他玩家通过事件向 owner 发请求
- owner 决定是否写入 `[UdonSynced]` 状态

### 5.3 为什么这样更稳

**[INFERENCE]** 这样做的收益是：
- 避免多人同时写同一状态
- 更容易控制序列化时机
- 更容易为 Late Joiner 保留单一事实源

### 5.4 风险提醒

**[FACT]** `SetOwner` 不是即时完成的，存在网络延迟；transfer 完成后才会触发 `OnOwnershipTransferred()`。

**[FACT]** 非 owner 调用某些 owner-only 写入逻辑会无效或静默失败。

---

## 6. Avatar↔World 的推荐建模链路

### 6.1 模式 A：Trigger / Contact 触发世界状态切换

```text
玩家进入 Trigger / Avatar Contact 命中
  → 业务脚本判断条件成立
  → owner 写入 [UdonSynced] 状态（如门已开、机关已激活）
  → RequestSerialization()
  → 所有在线玩家更新表现
  → Late Joiner 加入后通过 OnDeserialization() 恢复门已开状态
```

### 6.2 模式 B：非 Owner → Owner 更新请求

```text
非 owner 玩家触发某个交互
  → SendCustomNetworkEvent(Owner, "RequestChange")
  → owner 收到请求并校验
  → owner 修改 [UdonSynced] 状态
  → RequestSerialization()
```

### 6.3 模式 C：仅本地反应，不需要同步

```text
本地玩家观察到某个 Avatar / Tracking / Eye Height 信号
  → 本地只更新 UI / 摄像机 / 本地特效
  → 不写 [UdonSynced]
```

**[INFERENCE]** 不是所有 Avatar↔World 信号都要同步；只有“其他人也必须看到”或“晚加入者必须恢复”的结果，才应该进入状态层。

---

## 7. Late Joiner 视角下的关键原则

### 7.1 核心问题

**[INFERENCE]** Late Joiner 不关心“过去发生过什么事件”，更关心“当前世界状态是什么”。

### 7.2 因此要怎么设计

**[INFERENCE]** 对 Late Joiner 有意义的结果，通常应优先表示为：
- 当前开关状态
- 当前机关状态
- 当前同步数值
- 当前布局 / 当前阶段

并通过 `[UdonSynced]` 表达。

### 7.3 不要做的事

**[FACT]** 不要只依赖：
- `SendCustomNetworkEvent`
- 某次 Trigger Enter
- 某次 Contact Enter

来表达一个需要恢复的长期状态。

---

## 8. 与其他层的关系

### 8.1 与 Observation 层

**[INFERENCE]** Observation 层回答：
- World 当前能读到什么

Sync 层回答：
- 哪些结果要写成共享状态

### 8.2 与 Trigger / Contact 层

**[INFERENCE]** Trigger / Contact 层回答：
- 信号从哪里进入

Sync 层回答：
- 这些信号的结果如何被共享、恢复、保持一致

---

## 9. 常见误判警告

### 9.1 误判：事件发给 All 了，所以 Late Joiner 也会知道
错误。Event 不是状态存储层。

### 9.2 误判：Trigger / Contact 已经触发，所以状态已经同步
错误。触发只是入口，不代表 `[UdonSynced]` 状态已经写入。

### 9.3 误判：所有 Avatar↔World 信号都应该同步
错误。只同步真正需要多人一致或可恢复的结果。

### 9.4 误判：Owner 切换后可以立刻当作稳定完成
错误。Ownership transfer 有延迟。

---

## 10. 当前可稳定沉淀的分层结论

### [FACT]
1. `SendCustomNetworkEvent` 更适合事件/命令，不适合 Late Joiner 需要恢复的持续状态。
2. `[UdonSynced]` + `OnDeserialization()` 是 Late Joiner-safe 状态承载核心。
3. Manual Sync 修改后必须 `RequestSerialization()`。
4. Owner 是多人状态写入模型中的关键权威角色。

### [INFERENCE]
1. Trigger / Contact / Tracking / Bone / Avatar Event 更适合作为信号/观察入口理解，而不是多人一致状态承载层。
2. Avatar↔World 最稳定的工程模型是：信号进入 → owner 解释 → `[UdonSynced]` 持久化状态。
3. Event 更像命令层；`[UdonSynced]` 更像事实层。
4. 只有真正需要多人一致或可恢复的结果，才应进入同步层。

### [UNKNOWN]
1. 特定复杂项目中，不同 Owner 架构（固定 owner / 动态 owner / master-like）哪种最优。
2. 某些高频 Avatar↔World 交互在极端网络负载下的最佳分层策略。

---

## 11. 相关资料

- `avatar-world-boundaries.md`
- `player-observation-boundaries.md`
- `player-world-trigger-bridge.md`
- `avatar-world-contact-bridge.md`
- `../api/networking.md`

---

## 相关页面

[[avatar-world-boundaries.md]] · [[player-observation-boundaries.md]] · [[player-world-trigger-bridge.md]] · [[avatar-world-contact-bridge.md]] · [[../../entities/api/networking.md]] · [[../../entities/api/dynamics.md]] · [[../../entities/avatar/contact.md]] · [[../../entities/world/udon/players/player-collisions.md]]
