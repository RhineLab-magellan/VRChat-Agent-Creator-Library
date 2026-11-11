---
title: "VRCBillboard — 原生公告板（Billboard）API 类"
category: api
knowledge_level: applied
status: active
source: "https://ask.vrchat.com/t/developer-update-13-august-2026/48800 (2026-08-13 预告) + release note 3.10.6 + 官方 VRCBillboard 页"
source_type: official
version: 1.1
last_review: 2026-11-10
confidence: High
tags:
  - api
  - udonsharp
  - performance
  - world
  - optimization
aliases:
  - "VRCBillboard"
  - Billboard
  - 公告板
  - 面向玩家
related:
  - index.md
  - player-api.md
  - ../world/scene-components/index.md
  - ../world/udon/players/player-positions.md
  - ../avatar/contact.md
type: entity
created: 2026-08-19
sources: "https://ask.vrchat.com/t/developer-update-13-august-2026/48800 + https://creators.vrchat.com/releases/release-3-10-6/ + https://creators.vrchat.com/worlds/udon/vrc-billboard"
updated: 2026-11-10
---
# VRCBillboard — 原生公告板（Billboard）API 类

> ✅ **已正式发布：SDK 3.10.6（2026-10-30 16:24 UTC）**
> release note 3.10.6 原文："Added the VRCBillboard component for worlds. … You can also register transforms for billboarding directly from Udon."
> 官方 Udon 页：https://creators.vrchat.com/worlds/udon/vrc-billboard
>
> ⚠️ **本页仍有部分内容停留在 2026-08-13 预告口径**（例如"注册对象销毁时自动清理"这类措辞未在正式文档中逐字复现）。下表的**正式签名**以官方 VRCBillboard 页（2026-11-10 抓取）为准；预告期的三方法简表保留在 §"API 签名（预告）"作为历史对照，**不要再用它**。

---

## 概述

**告示板（Billboarding）** 指让某个对象始终面向本地玩家的常见需求（如面板、名字牌、UI 板、图片）。

传统实现需要在 Udon 中用**每帧更新循环（update loop）**旋转 Transform 使其面向玩家，开销较大。VRChat 正在为 World/场景对象添加**原生支持**，让告示板更易实现且更高效。

> **FACT**（官方 2026-08-13 公告）：官方动机即消除 Udon 中昂贵的 update loop —— 与骨骼挂载 API（[[../world/udon/players/player-positions.md|AttachTransformToBone]]）同期宣布，同属"原生替代每帧 Transform 搬运"的性能优化方向。

---

## API 签名（**正式，SDK 3.10.6**）

```csharp
using VRC.SDK3.Components;   // VRCBillboard / VRCBillboardFacing

public static void Register(
    Transform transform,
    VRCBillboardFacing facing = VRCBillboardFacing.Back,
    bool allowTilt = false,
    float smoothingFactor = 0f,
    Transform upFrom = null,
    bool matchViewpointRotation = false);

public static void Unregister(Transform transform);
```

| 参数 | 默认 | 作用 |
|---|---|---|
| `facing` | `VRCBillboardFacing.Back`（本地 -Z） | 用对象**自己的哪个局部轴**指向视点。Back 适配 Canvas / TextMeshPro / Quad / Plane / SpriteRenderer；Forward(+Z) 适配"正面是 +Z"的导入模型；另有 Up / Down / Left / Right |
| `allowTilt` | `false` | false = 只绕 up 轴转（永远直立）；true = 同时俯仰，正对视点 |
| `smoothingFactor` | `0f` | 转向缓动秒数。0 或负数 = 每帧直接吸附。约每 `smoothingFactor` 秒走完到目标旋转的 **63%**；**帧率无关** |
| `upFrom` | `null` = 世界 up | 用另一个 Transform 的 up 作为参考（车辆 / 传送台里保持直立）。**注意**：它同时定义 `allowTilt=false` 时的压平平面 |
| `matchViewpointRotation` | `false` | false = 朝向视点**位置**（旋转随自身位置变化）；true = **复制视点自身旋转**（所有用此设置的对象旋转一致，适合贴屏 UI） |

**注册语义（必须记住）**
- **按键是 Transform**，每个 Transform 只有一组选项。
- **重复注册 = 覆盖**：对同一个 Transform 再次 `Register` 会替换之前的选项，**最后一次调用胜出**。多个脚本碰同一对象时会互相踩。
- `Unregister` **不认来源**：任何脚本都能注销别人注册的 Transform；注销后对象停止 billboard，直到再次被注册。

### 运行时属性（组件形态）

组件 `VRCBillboard` 上对应字段：`Facing` / `AllowTilt` / `SmoothingFactor` / `UpFrom` / `MatchViewpointRotation`。
**赋值任一属性都会触发重新注册**，改动在下一帧 billboard pass 生效。

### 生命周期（组件形态）

组件在 `OnEnable` 自动注册、`OnDisable` 自动注销——所以**开关 GameObject 就能正确保持 billboard 状态**。
脚本注册形态则需要自己配对 `OnEnable` / `OnDisable`。

### Udon Graph 的坑（官方明列）

Udon Graph 里 `Register` 节点的 **Transform 端口不连**，会被解析为"图所在的 GameObject"而**不是参数默认值**。
- 目标 == 图所在对象：官方自动识别为自引用 → 当作"世界 up"
- 目标 != 图所在对象：**必须手动连 `upFrom`**（想用世界 up 就连一个 Const null）

### 与约束的分工

> 官方原文："A billboard always tracks the **local player's viewpoint**… To aim an object at anything else, use a **VRCAimConstraint** or **VRCLookAtConstraint**."
> 即：**只追本地视点**用 VRCBillboard；要追任意目标用约束（[[entities/api/dynamics]]）。

---

## API 签名（**预告期历史对照，勿再使用**）

将提供三个新方法（静态调用，需通过 `VRCBillboard` 类访问）：

| 方法 | 行为 | 说明 |
|------|------|------|
| `VRCBillboard.Register(Transform target)` | **直立（upright）**，仅跟踪**偏航角（yaw，左右）** | 简单告示板场景；对象保持垂直不倾斜 |
| `VRCBillboard.Register(Transform target, bool allowTilt)` | 在 yaw 基础上**允许俯仰（pitch，上下）** | `allowTilt = true` 时对象可随视角倾斜（如抬头/低头看板） |
| `VRCBillboard.Unregister(Transform target)` | **取消注册** | 注册对象销毁时**自动清理**，无需手动调用；也可手动触发以提前解绑 |

> ⚠️ 上表是 **2026-08-13 预告**的三方法简表，**与 3.10.6 正式签名不符**（正式版有 6 个参数且全部可选，第二参数是 `VRCBillboardFacing` 而**不是** `bool`）。保留仅为追溯；实际写代码请用上一节。
> 另：预告里"注册对象销毁时自动清理"在正式文档中的对应说法是"客户端会一直旋转已注册的 Transform，**直到你 unregister 或销毁它的 GameObject**"——语义接近但以正式文档为准。

---

## U# 用法示例（**正式 API，3.10.6**）

```csharp
using UdonSharp;
using UnityEngine;
using VRC.SDK3.Components;    // VRCBillboard / VRCBillboardFacing

public class BillboardLabel : UdonSharpBehaviour
{
    public Transform Vehicle;

    void OnEnable()
    {
        // 命名参数：只改需要的，其余保持默认
        VRCBillboard.Register(transform, allowTilt: true, smoothingFactor: 0.25f);
    }

    void OnDisable()
    {
        VRCBillboard.Unregister(transform);
    }
}
```

直立（仅偏航）+ 指定 up 参考（载具 / 传送台上保持直立）：

```csharp
VRCBillboard.Register(transform, upFrom: Vehicle);
```

贴屏 UI（复制视点旋转而不是朝向视点）：

```csharp
VRCBillboard.Register(transform, matchViewpointRotation: true);
```

> 🔴 **命名参数在此处是硬依赖**：官方文档推荐的写法全是 `allowTilt: true` / `smoothingFactor: 0.25f` 这类**跳参**调用。UdonSharp 的"命名参数跳过可选参数"bug **直到 3.10.6 才修**（见 [[concepts/rules/udonsharp-language-limits]] RULE-LL-15 §15.2）——在 3.10.4/3.10.5 上这些调用会**静默丢参数**。3.10.6 之前务必用位置参数写全。

> 注册为一次性调用，无需 Update 循环 —— 这正是本能力的性能优势所在（对比旧式每帧 `transform.LookAt(localPlayer)`）。

---

## 与 Unity 原生 `BillboardRenderer` 的区别

> ⚠️ 避免混淆：`BillboardRenderer`（Unity 原生组件，已并入 scene-components 索引的 [unity-rendering-aux.md](../world/scene-components/unity-rendering-aux.md) 合并页）是**面向摄像机的十字片渲染**组件，属于渲染管线；本页 `VRCBillboard` 是 **VRChat SDK 层面向本地玩家的 Transform 朝向** API 类，二者机制与用途不同。

---

## 性能动机（对比旧式做法）

| 方案 | 开销 | 说明 |
|------|------|------|
| 旧式：Udon Update 循环 + `transform.LookAt` | ⚠️ 每帧 Udon 指令成本高 | Udon VM 中每帧矩阵/四元数运算昂贵（官方强调） |
| 原生：`VRCBillboard.Register()` | ✅ 一次性注册，运行时原生处理 | 无需每帧脚本执行，替换 update loop |

> **FACT**（官方 2026-08-13 公告）：官方动机即"用 transforms 实现需要 Udon 更新循环，开销很大"，原生支持使其"更易用且更高效"。

---

## 适用场景

- 始终面向玩家的信息面板 / 名字牌 / 提示 UI
- 图片、视频面片（配合 VideoPlayer）朝向本地玩家
- 与 [[player-api.md|Player API]] 本地玩家相关逻辑配合（依赖本地玩家位置）

> 📌 该能力针对 **World/场景对象**（官方措辞为 world creator 视角）；Avatar 端公告板需求与 [[../avatar/contact.md|Contact]] 等 Avatar 机制分属不同域。

---

## 跨页引用

- **Scene Components（World 组件索引）**：[[../world/scene-components/index.md]]
- **骨骼挂载 API（同期预告）**：[[../world/udon/players/player-positions.md]]
- **VRC Contact / 接近度参数（同期预告）**：[[../avatar/contact.md]]
- **Player API**：[[player-api.md]]
- **API 总入口**：[[index.md]]

---

## 引用

- 官方 release note 3.10.6（正式发布）：https://creators.vrchat.com/releases/release-3-10-6/
- 官方 Udon 文档页（正式签名 / Options / Functions）：https://creators.vrchat.com/worlds/udon/vrc-billboard
- 预告来源：https://ask.vrchat.com/t/developer-update-13-august-2026/48800
- 关联 API：[[player-api.md]]（VRCPlayerApi / Networking.LocalPlayer）
- 同期 SDK 预告：骨骼挂载（[[../world/udon/players/player-positions.md]]）与接触接近度参数（[[../avatar/contact.md]]）
