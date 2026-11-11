---
title: "Rule Set: VRChat API Exposure"
category: rules
knowledge_level: applied
status: active
source: "VRChat 官方文档 + UdonSharp 文档 + SDK release note 3.10.4/3.10.5/3.10.6"
source_type: official
version: 1.1
last_review: 2026-11-10
confidence: Low
tags:
  - rules
  - rules
  - animator
  - networking
  - audio
  - pickup
aliases:
  - "Rule Set： VRChat API Exposure"
  - "Rule Set: VRChat API Exposure"
related:
  - performance-rules.md
  - networking-rules.md
  - ../patterns/slot-parameter-passing.md
  - multi-vm-rules.md
  - udon-vm-architecture.md
type: concept
created: 2026-06-20
sources: "VRChat 官方文档 + UdonSharp 文档 + SDK release note 3.10.4/3.10.5/3.10.6"
updated: 2026-11-10
---
# Rule Set: VRChat API Exposure


---

## RULE-API-01: Verify Before Use

### Rule
任何 API 使用前必须判断：
1. 是否被 VRChat Udon 暴露
2. 是否能在 UdonSharp 中编译
3. 是否适合在 Udon VM 热路径中调用
4. 是否会带来网络或多 VM 协作成本

### Reason
Unity 提供大量 API，但只有一小部分被 VRChat 暴露给 Udon。使用未暴露 API 会导致编译错误或运行时无效果。

---

## RULE-API-02: Known Exposed API Categories

### 确认暴露
- `UdonSharpBehaviour` 基类
- `[UdonSynced]`, `[FieldChangeCallback]`
- `Networking.SetOwner()`, `Networking.LocalPlayer`, `Networking.GetOwner()`
- `RequestSerialization()`, `OnDeserialization()`
- `SendCustomEvent()`, `SendCustomNetworkEvent()`, `SendCustomEventDelayedSeconds()`
- `Interact()`, `OnPickup()`, `OnDrop()`, `OnPickupUseDown()`, `OnPickupUseUp()`
- `OnPlayerTriggerEnter/Exit/Stay`
- `OnPlayerJoined`, `OnPlayerLeft`
- `OnOwnershipTransferred`
- `OnPreSerialization`, `OnPostSerialization`
- `VRCPlayerApi`, `TrackingData`
- `VRCObjectPickup`, `VRCObjectSync`
- `GameObject`, `Transform`, `Rigidbody`（基本属性和方法）
- `Animator`（基本参数设置）
- `AudioSource`（基本播放控制）
- `ParticleSystem`（基本控制）
- `Time.time`, `Time.deltaTime`
- `Vector3`, `Quaternion`（基本运算）
- `DataList`, `DataDictionary`, `DataToken`
- `VRCPlayerApi.TeleportTo()`

### 不确定或需要验证
- `Physics.Raycast` — 需要确认，部分项目可用
- `Material.SetColor` — 需要确认在不同 VRChat SDK 版本的表现
- `Renderer.material` — 可能受安全沙箱限制
- `Shader.SetGlobalFloat` — 通常受限

### 3.10.4–3.10.6 新增确认暴露（逐条有 release note 锚点）

| API | 版本 | 来源 |
|---|---|---|
| `VRCTween` / `VRCTweenHandle`（含 `DelayedCall` / `DelayedSetActive`） | 3.10.4 | release note 3.10.4；[[entities/api/vrctween]] |
| World 侧 `VRCPhysBoneCollider` 属性 + `ApplyConfigurationChanges()` | 3.10.4 | release note 3.10.4；[[entities/api/dynamics]] |
| `DataList(int)` / `DataDictionary(int)` / `DataDictionary.EnsureCapacity()` | 3.10.4 | release note 3.10.4；[[entities/world/udon/data-containers/data-dictionaries]] |
| `VRCQualitySettings.RealtimeReflectionProbes` / `.ShadowmaskMode`（**可写**） | 3.10.5 | release note 3.10.5；[[entities/api/vrchat-advanced-apis]] |
| 6 个 VRC Constraint 的 `AddSource/RemoveSource/GetSource/SetSource/GetSources/SetSources/ClearSources/SourceCount/SmoothingFactor` | 3.10.6 | release note 3.10.6 + 官方 Constraints API 页；[[entities/api/dynamics]] |
| `VRCBillboard.Register/Unregister`（含命名参数） | 3.10.6 | release note 3.10.6；官方 VRCBillboard 页；[[entities/api/vrc-billboard]] |

> ⚠️ **本次未能确认（不要当"已暴露"用）**：
> - `UdonBehaviour.LastErrorReason` —— **公开 C# 属性，不在任何 release note 中，也没有 Udon wrapper**。属于**编辑期/工具侧诊断**面，Udon 脚本里不可依赖。
> - 3.10.6 的 **Outline Renderers**（Pickup 与 Interact）—— 纯 authoring 字段，**无 Udon accessor**。
> - `VRCVelocityTracker` / `VRCPlacementZone` 的 Udon 方法面 —— 归 world 侧组件页，本页不展开。
> - 本页列的是**新增的 Udon 面**，不代表 3.10.3 导出快照（[[entities/api/udon-type-exposure]]）被更新；该快照仍是 3.10.3 时点。

### 确认不可用
- 任何 `System.IO` 文件操作
- 网络请求（HTTP、WebSocket）
- `PlayerPrefs`（跨平台表现可能不一致）
- `Camera` 操作的大部分方法

---

## RULE-API-03: API Heat Assessment

### 热路径安全 API（可在 Update/高频事件中使用）
- `Time.time`, `Time.deltaTime`
- `Vector3` 基本运算
- int/float 比较和赋值
- `Transform.position`（读）
- 数组索引读写

### 避免在热路径使用的 API
- `GameObject.Find()`
- `GetComponent<T>()`
- `Debug.Log()`
- `SendCustomEvent()` — 每次调用有帧延迟
- `RequestSerialization()` — 不应每帧调用
- 字符串操作
- 复杂的 `Transform` 链式访问

---

## RULE-API-04: When Uncertain

当 API 可用性不确定时：
1. 明确告知用户"需要验证"
2. 将 API 列入 `pending/uncertain-api.md`
3. 提供替代方案
4. 建议在 Editor 中测试编译
