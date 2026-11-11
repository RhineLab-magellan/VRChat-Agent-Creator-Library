---
title: "Official Documentation Clarifications"
category: api
knowledge_level: core
status: active
source: "VRChat 官方文档 (creators.vrchat.com/worlds/udon/) + release note 3.10.4/3.10.5/3.10.6 + 官方 API 页交叉核证"
source_type: official
version: 1.1
last_review: 2026-11-10
confidence: Low
tags:
  - api
  - reference
  - ownership
  - udonsharp
aliases:
  - "Official Documentation Clarifications"
  - official-doc-clarifications.md
related:
  - not-exposed.md
  - player-api.md
  - ui.md
  - animator.md
  - events-reference.md
type: entity
created: 2026-06-20
sources: "VRChat 官方文档 (creators.vrchat.com/worlds/udon/) + release note 3.10.4/3.10.5/3.10.6 + 官方 API 页交叉核证"
updated: 2026-11-10
---
# Official Documentation Clarifications


---

## 关键澄清与修正

### 1. Player Collision 仅对移动物体有效

> ⚠️ **重要**: `OnPlayerCollisionEnter/Stay/Exit` **仅在移动物体碰撞玩家时触发**。当玩家"走进"静止物体时不会调用这些事件。这与 Unity 标准 Collision 行为不同。

**影响**: 反向设计碰撞检测时必须用 Trigger 替代 Collision。

---

### 2. Trigger 事件的可靠性

> ⚠️ 玩家快速移动或传送时可能跳过 `OnPlayerTriggerEnter/Exit` 事件。

**影响**: 不要仅依赖 Trigger 事件做关键状态变更。考虑周期性位置检查做补充验证。

---

### 3. IsNetworkSettled

| 属性 | 说明 |
|---|---|
| `IsNetworkSettled` | 实例中所有数据是否已反序列化并应用完毕 |

**用途**: Late joiner 准备就绪的判断标志。可用于延迟初始化，等待所有网络数据同步完成。

---

### 4. OnOwnershipRequest 可以拒绝

```csharp
// OnOwnershipRequest 的返回值可以用来批准/拒绝
// 在 UdonSharp 中可能表现为返回 bool 的方法
```

**用途**: 在关键操作期间（turn-based 逻辑、事务中）拒绝 ownership transfer。

---

### 5. OnVariableChanged 事件

- 变量变化时触发，**包括接收同步变量时**
- `SetProgramVariable` 或 `Set Variable + sendChange` 都会触发
- 参数包含 newValue 和 oldValue
- 与 FieldChangeCallback 不同：OnVariableChanged 是更底层的事件

---

### 6. 持久化存储环境差异

| 环境 | 存储位置 | 持久性 |
|---|---|---|
| 上传的世界 | VRChat 服务器（账户关联） | 永久 |
| Build & Test | 本地 | 关闭客户端时删除，重启/重载保留 |
| ClientSim | 项目中 JSON 文件 | 文件持续存在 |
| 压缩 | VRChat 存储约 300KB 压缩为 100KB | — |

**影响**: Build & Test 中测试持久化后，数据不会跨次测试保留。

---

### 7. PlayerData 键名冲突防护

官方建议在 PlayerData 键名前添加唯一前缀：
```
"Momo-PPP-score" 而非 "score"
```

**影响**: 多个 UdonBehaviour 使用同一个 PlayerData key 会互相覆盖。

---

### 8. Video Player 速率限制

- 每个用户每 **5 秒**只能处理一个新的视频播放器 URL（全局限制）
- **Late Joiner 风险**: 多人同时播放时 late joiner 会同时发送多个请求，可能因速率限制而失败

---

### 9. Udon Assembly 类型命名规则

Udon Assembly 中的类型名称：
- 所有 `.` 和 `+` 被移除
- `[]` 用 `Array` 后缀表示
- 例: `VRC.SDKBase.VRCPlayerApi+TrackingData` → `VRCSDKBaseVRCPlayerApiTrackingData`
- 例: `System.Int32[]` → `SystemInt32Array`

**用途**: 调试时识别 Extern 签名中的类型。

---

### 10. Udon Assembly 代码结构

```assembly
.data_start
    # 变量定义，可 .export 标记为公开
    # 可附加 .sync variableName, interpolationMode
.data_end

.code_start
    .export _start          # 标记入口点
    _start:
        PUSH, message
        EXTERN, "UnityEngineDebug.__Log__SystemObject__SystemVoid"
        JUMP, 0xFFFFFFFC    # 结束（栈为空时 VM 自动停止）
.code_end
```

**程序结束条件**: 栈为空并尝试 POP 时自动停止。

---

### 11. 持久化存储限制事件

| 事件 | 说明 |
|---|---|
| `OnPlayerDataStorageExceeded` | 玩家超出 PlayerData 存储限制 |
| `OnPlayerDataStorageWarning` | 玩家接近 PlayerData 存储限制 |
| `OnPlayerObjectStorageExceeded` | 玩家超出 PlayerObject 存储限制 |
| `OnPlayerObjectStorageWarning` | 玩家接近 PlayerObject 存储限制 |

**用途**: 主动通知用户存储告警，防止数据丢失。

---

### 12. OnScreenUpdate 事件

- 移动设备进入世界或设备方向变化时触发
- 输出 `ScreenUpdateData`: type, orientation, resolution

---

### 13. Player 其他属性

| 属性/方法 | 说明 |
|---|---|
| `IsInstanceOwner` | 邀请/好友+实例返回 true，公开实例始终 false |
| `InstanceOwner` | 返回实例 owner 的 VRCPlayerApi 或 null |
| `IsSuspended` | 玩家设备是否被暂停（睡眠/切换应用） |
| `OnPlayerSuspendChanged` | 设备暂停/唤醒事件，检查 isSuspended 区分 |
| `IsVRCPlus` | 是否有 VRC+ 订阅 |
| `GetPickupInHand(Hand)` | 获取手持拾取物（仅本地玩家有效） |
| `PlayHapticEventInHand(Hand, duration, amplitude, frequency)` | 手柄震动（0-1） |
| `UseAttachedStation()` | 使玩家进入同 GameObject 上的 Station |
| `SimulationTime` | 玩家的模拟时间 |

---

### 14. SendCustomEvent 事件通信规则

> ⚠️ **仅对 public 事件有效**。Udon Graph 中自定义事件始终 public。UdonSharp 中确保使用 `public` 方法。

| 节点 | 说明 |
|---|---|
| `SendCustomEvent` | 立即运行 |
| `SendCustomEventDelayedFrames` | 延迟 N 帧（最少 1 帧） |
| `SendCustomEventDelayedSeconds` | 延迟 N 秒（0 秒可能在本帧或下一帧） |
| `SendCustomNetworkEvent` | 远程触发（All/Owner/Others） |

> ⚠️ `SendCustomEventDelayedFrames` 在 Update 之前调用（如 Start、Input 事件）可能少 1 帧

---

### 15. EventTiming（3.10.2+）

```
SendCustomEventDelayedSeconds(nameof(Method), 1.0f, EventTiming.FixedUpdate);
SendCustomEventDelayedFrames(nameof(Method), 1, EventTiming.PostLateUpdate);
```

支持: `FixedUpdate`, `PostLateUpdate`

---

### 16. 3.10.4–3.10.6 期间发现的官方文档自相矛盾 / 缺口（2026-11-10）

> 这一节只记录**对照过程中发现的文档问题**，每条都给出「按哪个为准」的判定。判定原则：**release note（时效性最强）> API 参考页正文 > 页面导言措辞**。

| # | 文档问题 | 证据 | 判定 |
|---|---|---|---|
| 16.1 | **VRCQualitySettings 被写成 "read-only"** | 官方页导言："A thin **read-only** layer over properties of UnityEngine.QualitySettings."，但同一页把 `RealtimeReflectionProbes`(bool) 与 `ShadowmaskMode`(UnityEngine.ShadowmaskMode) 列进 "Currently exposed"，且 release note 3.10.5 明说这两个**可从 Udon 启用/禁用** | **按 release note**：这两个是 3.10.5 新增的**可写**项；其余属性仍视为只读。见 [[vrchat-advanced-apis]] §2 |
| 16.2 | **DataDictionary 的 `EnsureCapacity()` 不在 API 页** | release note 3.10.4 明确 "VRCDataDictionary now exposes an EnsureCapacity() method…"，但官方 data-dictionaries 页（Last updated 2024-12-11）的函数表里查不到 | **按 release note**：方法存在（3.10.4+），但属"只在 release note 里"的文档缺口。见 [[entities/world/udon/data-containers/data-dictionaries]] §2.3 |
| 16.3 | **U# 文档 FAQ 与 Supported Features 自相矛盾（Properties）** | 同一页既列 "User defined properties" 为**支持**，又在 FAQ 把 "Properties" 列为常见**不可用** | 官方未澄清；**实践以 SDK 内实测为准**，不要拿 FAQ 那条反证自定义属性不可用。见 [[concepts/rules/udonsharp-language-limits]] §15.4 |
| 16.4 | **VRCBillboard 预告签名 ≠ 正式签名** | 2026-08-13 开发者更新给的是 `Register(Transform)` / `Register(Transform, bool)`；正式 3.10.6 是 6 参数（第二参数为 `VRCBillboardFacing` 枚举） | **按正式文档**：预告版第二参数是 bool 的说法**会让代码编译失败或语义错位**。见 [[vrc-billboard]] |
| 16.5 | **VRCTween 文档页时间戳早于 stable 发布** | 四页文档 `Last updated: May 28, 2026`，早于 3.10.4 stable（2026-06-17），但正文已含 beta.3 的 `DelayedSetActive` / `TweenPitch` | **按正文**：以正文列出的方法为准，不要因为时间戳早就认为不含新项。见 [[vrctween]] |
| 16.6 | **World Contact Receiver 没有 `OnContactStay`** | 官方 Contacts 组件页 World 侧只列 `OnContactEnter` / `OnContactExit`，全文无 "Stay" | **按组件页**：不存在第三个事件。**KB 曾把 `OnContactStay` 误记为真实回调**（8 处），2026-11-10 已全部纠正为「Enter 置标志 / Exit 清标志 / Update 读标志」的实现路径。见 [[dynamics]] §Contact Events 与 [[entities/avatar/contact]] §World 端 Udon 事件 |
| 16.7 | **`Use Face Proximity` 只有 authoring，没有 Udon 接口** | 官方 Contacts 页在 Box receiver 段描述该选项，但全文未给出对应 Udon 属性/方法 | 视为**纯 authoring 字段**；不要把"能改"写进 Udon API 表。见 [[dynamics]] §Contacts Box Shape |
| 16.8 | **Class Exposure Tree 没有可抓取的静态导出** | 官方页（Last updated 2024-08-06）只说明它是编辑器窗口 "VRChat SDK" > "Udon Sharp" > "Class Exposure Tree" | 无法从公开来源核实 3.10.6 曝光面；KB 快照保留 3.10.3 并标注未重新导出。见 [[udon-type-exposure]] |
| 16.9 | **3.10.6 beta 期的 Advanced Parameters 语义在 beta.3 改过** | beta.1/2 时 "Advanced Proximity Parameters" **只对 Proximity 类型生效**；beta.3 起改名为 "Advanced Parameters" 且**对所有 receiver 类型生效** | 引用 beta 期文档必须标注版本；以 stable 3.10.6 为准。见 [[dynamics]] §Contact 侧变化 |

> 这些不是"KB 与官方冲突"，而是**官方文档内部/跨文档的不一致**。记录它们的价值：下次有人只读官方 API 页就断言"某成员不存在"时，能立刻定位到这里的版本锚点。

---

## 相关页面

[[not-exposed.md]] · [[player-api.md]] · [[ui.md]] · [[animator.md]] · [[events-reference.md]]
