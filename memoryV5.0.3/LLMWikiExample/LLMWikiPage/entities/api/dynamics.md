---
title: "API: VRChat Dynamics for Worlds (SDK 3.10.0+)"
category: api
knowledge_level: core
status: active
source: "VRChat 官方文档 + VRChat Agent Skills 模板(参考工程) + 2026.2.1 Release Notes + SDK release note 3.10.4/3.10.6 + 官方 Constraints API 页"
source_type: official
version: 1.2
last_review: 2026-11-10
confidence: High
tags:
  - api
  - physbone
  - contact
  - constraint
  - event
aliases:
  - "Dynamics API"
  - 动态系统
  - ../../PhysBone/Contact API
  - Dynamics
related:
  - api-checker.md
  - audio.md
  - events-reference.md
  - not-exposed.md
  - persistence.md
type: entity
created: 2026-06-30
sources: "VRChat 官方文档 + VRChat Agent Skills 模板(参考工程) + 2026.2.1 Release Notes + SDK release note 3.10.4/3.10.6 + 官方 Constraints API 页"
updated: 2026-11-10
---
# API: VRChat Dynamics for Worlds (SDK 3.10.0+)

> SDK Version: 3.10.0+（Dynamics 引入）；本页约束/Contact 增量已核证至 **3.10.6**
> Last Verified: 2026-11-10（3.10.6 发布 2026-10-30 16:24 UTC）

---

## Overview

SDK 3.10.0 引入了 World-side Dynamics：PhysBones、Contacts、VRC Constraints 现在可以在世界对象上使用。

## Contact Events

### OnContactEnter / OnContactExit

> 🔴 **`OnContactStay` 不存在（2026-11-10 定论，不提供）**：World 侧的 Contact Receiver **只有两个 Udon 事件** —— `OnContactEnter(ContactEnterInfo)` 与 `OnContactExit(ContactExitInfo)`。**VRChat 不提供 Stay 回调。**
> 依据：① 官方 Contacts 组件页 https://creators.vrchat.com/common-components/contacts/ （World 侧事件列表只有 Enter/Exit，全文无 "Stay"）；② 3.10.6 release note 亦未引入任何 Stay 类事件。
> **需要"持续接触"语义时**：用 `OnContactEnter` 置位 + `OnContactExit` 复位，自维护一个 bool/计数器状态；或在需要时用 `VRCContactReceiver` 的 Proximity 值轮询。**不要**去找/写 `OnContactStay` —— 写了也不会被调用（编译可能通过，运行时静默无回调）。
> 本页旧标题曾写作 "Enter / Exit / Stay"，已修正；本页内原有 3 处 Stay 措辞亦已在本轮清除。

```csharp
using VRC.Dynamics;

public override void OnContactEnter(ContactEnterInfo info) {
    if (info.isAvatar) {
        Debug.Log($"Touched by avatar: {info.player?.displayName}");
    }
}
```

### ContactEnterInfo
- `info.isAvatar` — bool，是否来自 Avatar
- `info.player` — VRCPlayerApi（可能为 null）
- 需要 `VRCContactReceiver` 组件

### Contacts Box Shape（3.10.4+）

> release note 3.10.4："Contact senders and contact receivers on avatars and worlds can now be box shaped. You can adjust the width, height and depth of the box shape independently. Proximity can be calculated as either distance from the center like other contact types, or distance from a face of the box."

| 事实 | 值 / 说明 |
|---|---|
| Shape Type 可选值 | Sphere / Capsule / **Box** |
| Box 尺寸 | `Size`（Vector3）——**width / height / depth 独立可调** |
| 官方尺寸上限 | 半径最大 3 m；宽/高/深最大 6 m；**挂在缩放过的 GameObject 上时上限在缩放之后计算** |
| Proximity 两种口径 | ① 到接收器中**心**的距离（默认，与其他形状一致）② 到盒**面**的距离 |
| 面接近度的开关名 | Inspector 上的 **`Use Face Proximity`**（仅 Box receiver 可用），计算的是"sender 表面到盒子 **+Z 面**的最近点" |

> 🔴 **authoring 行为，不是 Udon wrapper 方法**：`Use Face Proximity` 是组件字段，官方文档**没有**给出对应的 Udon 读写接口。3.10.6 只改善了"薄 box receiver"的接触检测与 proximity 计算精度（release note 原文），没有新增 API 面。

### Contact 侧 3.10.5 / 3.10.6 变化（World 视角）

| 版本 | 变化 | 对 World Udon 的影响 |
|---|---|---|
| 3.10.5 | 修 "sphere shaped contact sender with an offset" 时 contact point 取错 | 无需改代码 |
| 3.10.5 | PhysBones 的 **Reset When Disabled** 现在同时重置 **animator 参数与 Udon state** | 依赖 PhysBone 复位做状态机的世界要重测 |
| 3.10.5 | Play mode 下 domain reload 不再让 PhysBones / Contacts 停止工作 | Editor 内调试行为变化 |
| 3.10.6 | **Advanced Proximity Parameters**（beta.3 起更名 **Advanced Parameters**）：可拿到最近接触 sender 的类型、局部位置、局部速度等 | Avatar 侧主要能力；World 侧只有 `OnContactEnter/Exit` 两事件 |
| 3.10.6 | **Material Contacts**（写最多 8 个 sender 的变换数据到最多 4 个 MeshRenderer 的材质属性），配 SDK 内 `VRCContacts.cginc` | World 侧可用的 shader 驱动接触 |
| 3.10.6 | Advanced Parameters / Material Contacts **不再只对 Proximity 类型生效**（beta.3 修正） | 早期 beta 文档不可直接引用 |
| 3.10.6 | 新增高级参数 `_SenderLossyScale*` 与 **Point Separation**；两个 sender 同为 proximity=1 时取 **point separation 更小**的那个 | 决定多 sender 竞争时的取值语义 |

> 🔴 事件面再确认：World 侧 Contact Receiver 只触发 `OnContactEnter(ContactEnterInfo)` / `OnContactExit(ContactExitInfo)`（官方 Contacts 组件页），**没有** `OnContactStay`；「持续接触」用 Enter 置位 + Exit 复位自维护状态，详见本页 §Contact Events。

---

## PhysBone Events

### OnPhysBoneGrabbed / OnPhysBoneReleased
```csharp
public override void OnPhysBoneGrabbed(PhysBoneGrabbedInfo physBoneInfo) {
    // Called when the PhysBone is grabbed by a player.
    Debug.Log($"PhysBone grabbed by: {physBoneInfo.player?.displayName}");
}
```

### PhysBoneGrabbedInfo
- `physBoneInfo.player` — VRCPlayerApi（可能为 null）

---

## VRC Constraints

World-side constraints 类似 Unity Joint 系统，但在 VRChat 的约束框架下运行。

> **SDK 3.10.6 起**：6 个 VRC Constraint 类型（`VRCAimConstraint` / `VRCLookAtConstraint` / `VRCParentConstraint` / `VRCPositionConstraint` / `VRCRotationConstraint` / `VRCScaleConstraint`）新增**成组的 source 读写方法与 SmoothingFactor**，见下节。
> 证据：release note 3.10.6 "Added a series of helper methods to VRChat Constraints for adding, removing, getting, and setting sources, similar to Unity's constraints API." + 官方 Constraints API 页 https://creators.vrchat.com/common-components/constraints/constraints-api

### 3.10.6 约束 Source API（**在 Udon 里用方法，不要直接改 Sources 列表**）

命名空间：
- 组件类型 `VRC.SDK3.Dynamics.Constraint.Components`
- 支撑类型 `VRC.Dynamics`：`VRCConstraintSource`（含 `SourceTransform` + `Weight`）、`VRCConstraintSourceKeyableList`

| 成员 | 签名 / 类型 | 说明 |
|---|---|---|
| `SourceCount` | `int`（属性） | 当前 source 数量 |
| `AddSource` | `AddSource(VRCConstraintSource source)` | 追加到列表末尾 |
| `RemoveSource` | `RemoveSource(int index)` / `bool RemoveSource(VRCConstraintSource source)` | 按索引删；按 source 删时返回是否找到并删除 |
| `GetSource` | `VRCConstraintSource GetSource(int index)` | 越界会**报错并返回空 source** |
| `SetSource` | `SetSource(int index, VRCConstraintSource source)` | 越界同样报错并返回空 source |
| `GetSources` | `int GetSources(VRCConstraintSource[] sources)` | 填入你提供的数组，返回**实际拷贝的数量**（数组不够大就截断） |
| `SetSources` | `SetSources(VRCConstraintSource[] sources)` | 用给定数组**完全替换**现有列表；null / 空数组 = 清空 |
| `ClearSources` | `ClearSources()` | 清空全部 source |
| `SmoothingFactor` | `float`（属性） | 0 = 不平滑；公式见下 |

推荐写法（官方示例改写）：

```csharp
using VRC.SDK3.Dynamics.Constraint.Components;
using VRC.Dynamics;

// 逐个读改写权重
for (int i = 0; i < constraint.SourceCount; i++)
{
    VRCConstraintSource s = constraint.GetSource(i);
    s.Weight = 0.5f;
    constraint.SetSource(i, s);
}

// 新增一个指向某 Transform、权重 1 的 source
constraint.AddSource(new VRCConstraintSource(targetTransform, 1.0f));

// 批量取（数组必须够大）
VRCConstraintSource[] buf = new VRCConstraintSource[constraint.SourceCount];
int n = constraint.GetSources(buf);
```

> ⚠️ **不存在的成员**（旧参考资料里出现过，属**错误**）：`GetSourceCount()`、`SetSourceWeight()`。
>
> **证据锚点（可直接复核）**：
> - **官方 Constraints API 页**：https://creators.vrchat.com/common-components/constraints/constraints-api —— 该页在源列表里**只有** `SourceCount` **属性** + `AddSource` / `RemoveSource` / `GetSource` / `SetSource` / `GetSources` / `SetSources` / `ClearSources` 这 8 个成员；全文检索 **`GetSourceCount` = 0 处、`SetSourceWeight` = 0 处**（2026-11-10 抓取核验，页面正常 200，非 404）。
> - 交叉锚点：**3.10.6 release note**（https://creators.vrchat.com/releases/release-3-10-6/ ）只描述 "a series of helper methods … for adding, removing, getting, and setting sources"，未提任何 count/weight 专用方法。
>
> **正确写法**：按索引改权重必须 `GetSource` → 改 `source.Weight` → `SetSource`（`VRCConstraintSource` 是含 `SourceTransform` + `Weight` 的 struct，权重在 struct 上，不在约束上的独立 setter）。

> ⚠️ **raw `Sources` 列表**：仍然对 Udon 开放（向后兼容），但官方原文 "VRChat strongly recommends you use the methods listed below instead. This is because Udon can have issues applying changes when modifying the list directly (since it's a struct type)."
> 另有限制：`Sources` 列表**只有前 16 个元素**能被 Animator 定位（Unity 对数组元素动画的限制），写 SDK 工具时要注意。

### ApplyConfigurationChanges() —— 改完必须调

```csharp
// 约束（3.10.6）
VRCPositionConstraint c = ...;
c.GlobalWeight = 0.5f;
c.SmoothingFactor = 0.2f;
c.ApplyConfigurationChanges();   // ← 改完必须调

// PhysBone Collider（3.10.4 起 world 侧可从 Udon 改，见本页 §World-side VRCPhysBoneCollider）
collider.radius = 0.3f;
collider.ApplyConfigurationChanges();
```

官方口径：**批量修改多个属性后只调用一次**，以减少性能开销（"aim to only call this method once after all of your changes"）。
> 约束/PhysBone 的属性改动**不会**自动回写组件——不调这个方法，改动可能"看起来赋值成功但没生效"。

### SmoothingFactor（3.10.6）

> release note 3.10.6："Added a Smoothing Factor option for VRChat Constraints. When set to a value above zero, this allows a constraint to move smoothly towards its target over time rather than immediately snapping to it."

- 0 = 不平滑（瞬时贴合）
- 数值越大，追上目标越慢；**约每 `SmoothingFactor` 秒走完到目标的 63%**
- 官方实现公式（`deltaTime` = 距上一帧时间）：`result = lerp(current, target, 1 - exp(-deltaTime / smoothingFactor))`
- 这是**帧率无关**的指数趋近，与朴素 `Mathf.Lerp` 每帧固定系数写法不同

### 其他约束成员（3.10.6 前已有，速查）

| 成员 | 说明 |
|---|---|
| `IsActive` (bool) | 是否正在求值（等价 Unity `constraintActive`） |
| `GlobalWeight` (float) | 全局权重，叠在各有权重的 source 之上（等价 Unity `weight`） |
| `Locked` (bool) | 锁定态；**play mode 下约束永远视为 locked** |
| `TargetTransform` (Transform) | 受约束影响的 Transform；为 null 时作用于自身（3.10.6 起 play mode 下改它也会响应） |
| `SolveInLocalSpace` (bool) | 局部空间求解（Unity 约束恒为世界空间） |
| `FreezeToWorld` (bool) / `RebakeOffsetsWhenUnfrozen` (bool) | 冻结到世界位姿 / 解冻时是否重算 offset |
| `ActivateConstraint()` / `ZeroConstraint()` | 等价 Inspector 的 Activate / Zero 按钮（3.10.5 起可 Undo，且会标记场景为未保存） |
| `PositionAtRest`/`PositionOffset`、`RotationAtRest`/`RotationOffset` 等 | 各约束类型自己的 at-rest / offset；Parent 约束的 offset 在**每个 source 上** |

---

## World-side VRCPhysBoneCollider（3.10.4+）

> release note 3.10.4："Exposed PhysBone colliders in worlds to Udon, allowing their properties to be changed dynamically via Udon scripts. After modifying a VRCPhysBoneCollider, you need to call ApplyConfigurationChanges() on it to apply the changes, similar to how PhysBone components behave."

- 3.10.4 起，**world 里的 `VRCPhysBoneCollider` 暴露给 Udon**，可运行时读写属性。
- 官方点名的可改属性：`height` / `position` / `radius` / `rotation` / `shapeType`（另有 `ApplyConfigurationChanges()`）。
- 🔴 **改完必须调 `ApplyConfigurationChanges()`**，否则改动可能不生效；一次改多个属性后**只调一次**。

```csharp
using VRC.SDK3.Dynamics.PhysBone.Components;   // VRCPhysBoneCollider

public VRCPhysBoneCollider collider;

public void Enlarge() {
    collider.radius = 0.5f;
    collider.height = 1.2f;
    collider.ApplyConfigurationChanges();   // ← 必须
}
```

> 与 Avatar 侧的关系：Avatar 的 PhysBone Collider 早已可在 Udon 侧操作（[[entities/avatar/physbones]] 已有记录）；3.10.4 把这条路径**延伸到 world**。
> 3.10.4 同时（Avatar 侧）加入**全局 PhysBone Collider**（每个 Avatar 最多额外 4 个可标记为 global），并明确 **plane 形状的 collider 不支持 global 碰撞**。
> 3.10.5 又给 global PhysBone Collider 加了 **Allow Self / Allow Others**。

---

## Interact Outline Renderers（3.10.5 / 3.10.6，**无 Udon accessor**）

| 版本 | 事实 | 来源 |
|---|---|---|
| 3.10.5 | 给 **VRCPickup** 加 "Outline Renderers" 属性：手动指定 hover 时高亮的部件 | release note 3.10.5 |
| 3.10.6 | 同样能力加到**实现 `Interact()` 的 UdonBehaviour**：可指定 hover 时用哪些子 Renderer 生成 outline | release note 3.10.6 |
| client 2026.4.1 起 | Pickup 与 Interactable 在 **Outline Renderers 列表为空**时**不再默认高亮 SkinnedMeshRenderer**（因为有存量内容依赖旧行为） | release note 3.10.6 |

> 🔴 **纯 authoring 字段**：官方 release note 与组件页都**没有**给出 Outline Renderers 的 Udon 读写接口，**无 Udon accessor**。想程序化控制 outline 只能改这个列表以外的路径（换材质 / 自己画）。
> 注意区分：本项是 **Pickup / Interactable 的 hover 描边**，与 [[entities/world/udon/vrctween]] 无关，也不等于 Shader 描边。

---

## SDK Version Requirement

- ⚠️ `OnPhysBoneGrabbed`、`OnContactEnter` 等在 SDK < 3.10.0 **编译通过但运行时静默忽略**
- 始终检查 `#if UDONSHARP` 或 SDK 版本条件

---

## SDK Version Feature Timeline

| SDK | Key Features |
|---|---|
| 3.7.1 | StringBuilder, Regex, System.Random |
| 3.7.4 | **Persistence API** (PlayerData / PlayerObject) |
| 3.7.6 | Multi-platform Build & Publish |
| 3.8.0 | PhysBone dependency sorting, Drone API |
| 3.8.1 | **`[NetworkCallable]`** (parameterized network events) |
| 3.9.0 | Camera Dolly API, Auto Hold pickup |
| 3.10.0 | **Dynamics for Worlds** (PhysBones, Contacts, VRC Constraints) |
| 3.10.1 | Bug fixes, stability improvements |
| 3.10.2 | EventTiming.PostLateUpdate/FixedUpdate, PhysBones fixes |
| 3.10.3 | VRCPlayerApi.isVRCPlus, VRCRaycast (avatar), Mirror fix |
| 3.10.4 | **VRCTween / VRCTweenHandle**（DOTween 封装，见 [[vrctween.md]]）；**Contacts Box shape**；World 侧 **VRCPhysBoneCollider 暴露给 Udon**（改属性后必须 `ApplyConfigurationChanges()`）；DataList/DataDictionary 支持自定义 capacity + `EnsureCapacity()`；UdonSharpBehaviour 实现 `IUdonEventReceiver`（省一次 cast） |
| 3.10.5 | **VRCQualitySettings.RealtimeReflectionProbes / ShadowmaskMode**（Udon 可读写）；U# 支持 **Assembly Version Defines**；重新支持在 `Assets/` 与 `Packages/` 建 U# 脚本；ClientSim 在 Play→Edit 时**不再**跑 `OnDisable` 等 Unity 事件；PhysBone "Reset When Disabled" 也重置 animator 参数与 Udon state；VRCJson 嵌套花括号 / 字符串内括号修复；DataToken int→long 转换修复 |
| 3.10.6 | **Constraints source API + SmoothingFactor**（见下节）；U# **enum 比较**与**命名参数跳过可选参数**两处编译器修复；sync null VRCUrl（persistence）异常修复；U# Interact 新增 **Outline Renderers** 字段（**无 Udon accessor**）；VRCBillboard / VRCVelocityTracker / VRCPlacementZone 三个新组件（见 world 侧页面） |

> SDK < 3.9.0 已于 2025-12-02 弃用。新 world 上传需要 3.9.0+。
> 3.10.6 仍基于 **Unity 2022.3**，release note 顶部明确警告 "Do NOT upgrade your projects to Unity 6!"。

---

## VRCRaycast (Avatar 端)

> **🔴 注意**:VRCRaycast 是 **Avatar 组件**,**不属于 World Dynamics**。但因同属 SDK 3.10.3 引入,在此处简要说明。
>
> **完整文档**:[[entities/avatar/vrcraycast]]

### 速查

- 作用:Avatar 骨骼向下投射 raycast,结果写入 Animator 参数(默认 `IsRaycastHit`)
- 用途:踩地检测、遮挡检测、Avatar 端世界交互
- 文档:https://creators.vrchat.com/avatars/avatar-components/raycast

### 2026.2.1 关键更新

| 改进 | 详情 |
|------|------|
| **自动剥离** | 玩家本地屏蔽 Poor/Very Poor 时,从其他玩家 Avatar 上剥离 VRCRaycast |
| **无实时光照修复** | Animator 移动 raycast 时参数正确返回 |
| **第一人称对称** | 头部 raycast 在头部隐藏**之前**应用 |

详见 [[entities/avatar/vrcraycast]] 完整文档。

---

## 相关页面

[[api-checker.md]] · [[audio.md]] · [[events-reference.md]] · [[not-exposed.md]] · [[persistence.md]]
