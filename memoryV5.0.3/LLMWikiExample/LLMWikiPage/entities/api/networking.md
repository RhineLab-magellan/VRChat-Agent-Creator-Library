---
title: API: Networking
category: api

knowledge_level: core
status: active

tags:
  - api
  - networking
  - sync
  - serialization
  - ownership

aliases:
  - Networking
  - "网络 API"
  - 网络同步
  - "Networking API"

related:
  - ../world/udon/networking/index.md
  - ../world/udon/networking/compatibility.md
  - ../world/udon/networking/events.md
  - ../world/udon/networking/variables.md
  - ../world/udon/networking/ownership.md
  - ../world/udon/networking/late-joiners.md
  - ../world/udon/networking/network-components.md
  - ../world/udon/networking/debugging.md
  - ../world/udon/networking/performance.md
  - ../world/udon/networking/network-details.md

source: VRChat + UdonSharp 官方文档
source_type: official
version: 1.0
last_review: 2026-06-04
confidence: High
type: entity
created: 2026-06-04
sources: VRChat + UdonSharp 官方文档
updated: 2026-06-04
---
# API: Networking

---

## Networking (UdonBehaviour 成员)

### Networking.SetOwner(VRCPlayerApi player, GameObject obj)
- **暴露**: ✅
- **热路径**: ❌ (涉及 ownership transfer network message)
- **说明**: 将 obj 的 ownership 转移给 player。**当前 SDK 语义应以 [[entities/world/udon/networking/ownership]] 与 `rules/networking-rules.md` 为准**：本地调用后通常会先反映为本地 ownership 变化，再由网络与服务器裁决收敛最终结果。
- **注意**:
  - 不应再把它简化成“只有当前 owner 可以转移，非 owner 调用无效果”的旧说法。
  - **本地效果可能先立即可见**，但最终 ownership 仍受网络传播与服务器 / durable owner 裁决影响。
  - 与 `OnOwnershipTransferred()` 的关系应理解为：该事件反映一次 ownership 变更已完成传播 / 应用，而不是说明 `SetOwner` 在本地调用瞬间绝对没有效果。

### Networking.LocalPlayer
- **暴露**: ✅
- **热路径**: ✅
- **说明**: 返回本地玩家的 VRCPlayerApi。在 Start 中缓存引用。

### Networking.GetOwner(GameObject obj)
- **暴露**: ✅
- **热路径**: ✅ (轻量查询)
- **说明**: 返回 obj 的当前 owner。

### Networking.IsOwner(GameObject obj)
- **暴露**: ✅
- **热路径**: ✅
- **说明**: 检查本地玩家是否是 obj 的 owner。常用于 guard clause。

## UdonBehaviour 同步成员

### RequestSerialization()
- **暴露**: ✅
- **热路径**: ⚠️ (会触发序列化，不适合每帧调用)
- **说明**: Manual Sync 模式下，显式请求序列化 synced variable。修改后必须调用。

### OnDeserialization()
- **暴露**: ✅
- **热路径**: ✅ (网络事件驱动)
- **说明**: 收到远端 synced variable 更新时调用。Late joiner 加入时也会调用。

### OnPreSerialization()
- **暴露**: ✅
- **热路径**: ⚠️ (每次序列化前调用)
- **说明**: 序列化前准备数据。可以在这里更新 synced variable 为最新值再序列化。

### OnPostSerialization()
- **暴露**: ✅
- **热路径**: ⚠️
- **说明**: 序列化完成后调用。少见使用场景。

### SendCustomNetworkEvent(NetworkEventTarget target, string eventName)
- **暴露**: ✅
- **热路径**: ⚠️ (产生网络消息)
- **说明**: 向指定目标发送网络事件。eventName 必须是无参方法名。
- **Target 选项**: `NetworkEventTarget.All`, `NetworkEventTarget.Owner`

### BehaviourSyncMode
- `NoVariableSync` — 不同步任何变量
- `Manual` — 手动调用 RequestSerialization 同步
- `Continuous` — 自动持续同步

### UdonSyncMode（同步变量插值模式）
用于 `[UdonSynced]` 属性的可选参数，控制网络同步时的插值行为：

| Mode | 说明 | 适用场景 |
|------|------|---------|
| `None` | 无插值（默认） | 离散状态、开关、枚举值 |
| `Linear` | 线性插值 (lerp) | 连续位置、速度 |
| `Smooth` | 平滑插值 | 需要更自然过渡的值 |
| `NotSynced` | 显式不同步 | 本地计算或手动同步 |

```csharp
[UdonSynced] public bool toggle;                    // 默认 None
[UdonSynced(UdonSyncMode.Linear)] public float progress;  // 线性插值
[UdonSynced(UdonSyncMode.NotSynced)] public int localOnly; // 不同步
```

**注意**: `NotSynced` 用于需要本地计算后手动同步的场景，与 `BehaviourSyncMode.None` 不同。

---

## Bandwidth Limits ⚠️

| 限制类型 | 数值 | 说明 |
|---------|------|------|
| **总带宽** | ~11 KB/s | Udon scripts 可发送的总带宽 |
| **Manual Sync** | 280,496 bytes/serialization | 每次序列化最大数据量 |
| **Continuous Sync** | ~200 bytes/serialization | 每次序列化最大数据量 |

### 超出限制后果
- 网络拥塞
- `IsClogged` 返回 true
- 数据丢失或延迟

### 优化建议
- Manual Sync 用于离散状态（棋盘位置、游戏状态）
- Continuous Sync 用于高频变化（位置、进度条）
- 避免频繁序列化

---

## Outgoing Queue Drop Policy (2026.2.1+ 拥塞优化) ⚠️

> **FACT** (2026.2.1 Build 1833):VRChat 客户端在出站队列过大时**主动丢弃**不可靠网络数据。

### 受影响的数据类型

- **`VRCObjectSync`**(位置/旋转同步)
- **Udon Behaviour 设为 `Continuous` 的变量**(高频变化)

### 行为

| 状态 | 行为 |
|------|------|
| 队列正常 | 所有数据正常发送 |
| 队列过大 | **客户端丢弃中间的旧数据** |
| 最新数据 | ⚠️ **永远传输**(保证不 desync) |

### 官方原话

> "Unreliable network data (e.g. VRC Object Sync or Udon Behaviors set to Continuous mode) will now be dropped client-side if the outgoing queue gets too large. The latest state will still always be transmitted, so there should be no desyncs, but under extreme network load in-between data may be dropped in favor of maintaining local FPS and RAM footprints."

### 对创作者的影响

| 场景 | 影响 |
|------|------|
| **设计高频 Continuous Sync(50+ Hz)** | 极重负载下中间数据丢失,但最终状态一致 |
| **依赖中间帧动画插值** | 可能在拥塞时跳变(因为中间帧丢弃) |
| **游戏状态同步(Manual)** | **不受影响**(Manual 是可靠的) |
| **VRCObjectSync 平滑** | 默认仍然平滑,只是"追赶"间隔变长 |

### 缓解策略

- 降低 Continuous 频率(从 50Hz 降到 20Hz)
- 优先用 `[UdonSynced(Linear)]` 让 VRChat 客户端插值
- 关键状态用 Manual Sync + 显式 RequestSerialization
- 避免在 Update 中每帧 SetFloat

---

## Network Events Rate Limiting

### [NetworkCallable] (SDK 3.8.1+)
```csharp
[NetworkCallable(maxEventsPerSecond: 50)]
public void MyNetworkedMethod() { }
```

- 限制方法每秒可被调用的次数
- 范围: 1-100
- 默认: 无限制（但有隐含限制）

### NetworkCalling API
```csharp
// 获取指定事件排队数
int queued = NetworkCalling.GetQueuedEvents(target, eventName);

// 获取所有排队事件
int allQueued = NetworkCalling.GetAllQueuedEvents();
```

### 安全建议
- 私有方法或以下划线 `_` 开头的方法不会被网络调用
- `[NetworkCallable]` 显式标记网络可调用方法

---

## 常见错误
- Manual Sync 忘记 RequestSerialization（最常见 bug）
- 非 owner 侧对 `SetOwner` 语义的理解停留在旧版本经验，未考虑当前 SDK 下本地先变化、最终仍受网络 / 服务器裁决收敛的行为
- Continuous Sync 用于离散状态
- SendCustomNetworkEvent 用于 late joiner 需要的状态
- 超出带宽限制导致网络拥塞
- 依赖高频 Continuous 帧间插值(2026.2.1+ 中间帧可能丢失,见 Outgoing Queue Drop Policy)

---

## 深入阅读(应用层详解)

> 本文档为 **API 速查表**。完整的应用层详解、模式、案例、风险分析见 [[entities/world/udon/networking/index]] 子分类。

| 子页面 | 主题 |
|--------|------|
| [[entities/world/udon/networking/index]] | Networking 概述、带宽限制速查、Manual vs Continuous 决策树 |
| [[entities/world/udon/networking/compatibility]] | 跨版本兼容、降级策略、Serialization 限制 |
| [[entities/world/udon/networking/events]] | 完整 Network Events 列表、`[NetworkCallable]`、速率限制(5-100/s) |
| [[entities/world/udon/networking/variables]] | 完整 `[UdonSynced]` 字段类型表、序列化大小、FieldChangeCallback |
| [[entities/world/udon/networking/ownership]] | 完整所有权转移流程、Master vs InstanceOwner、SetOwner 安全 |
| [[entities/world/udon/networking/late-joiners]] | Late Joiner 同步策略、Buffer Events、OnPlayerJoined 模式 |
| [[entities/world/udon/networking/network-components]] | VRCObjectSync、VRCPickup、VRCObjectPool、Networking 属性/事件 |
| [[entities/world/udon/networking/debugging]] | World Debug Views、ClientSim 调试、Stats API |
| [[entities/world/udon/networking/performance]] | 10 条带宽优化规则、对象池、合并序列化 |
| [[entities/world/udon/networking/network-details]] | 内部字节细节、Manual vs Continuous 字节差异、数组陷阱 |

> 数据来源:VRChat 官方 Creator Docs(2025-11-13 更新版本),本地化日期 2026-06-15。
