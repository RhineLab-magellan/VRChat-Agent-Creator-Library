---
title: "CharacterJoint — 世界物理组件"
category: world
subcategory: scene-components
knowledge_level: applied
status: active
source: "Unity Manual 2022.3 + VRChat 官方文档"
source_type: official
version: 1.0
last_review: 2026-08-17
confidence: medium
tags:
  - world
  - unity
  - physics
  - joint
aliases:
  - CharacterJoint
  - 角色关节
  - 布娃娃关节
related:
  - ../whitelisted-world-components.md
  - configurablejoint.md
  - rigidbody.md
  - ../../api/pickups.md
  - vrc-objectsync.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-CharacterJoint.html | https://docs.unity3d.com/2022.3/Documentation/ScriptReference/CharacterJoint.html | https://docs.unity3d.com/2022.3/Documentation/Manual/RigidbodiesOverview.html | https://creators.vrchat.com/worlds/whitelisted-world-components/"
updated: 2026-08-17
---
# CharacterJoint — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无本组件独立帧时间数据，详见性能小节）

---

## 核心功能（官方，[FACT]）

Character Joint 是**扩展的球窝（ball-socket）关节**，允许你在每个轴上限制关节运动。它主要用于 **ragdoll（布娃娃）效果**——官方指引配合 Ragdoll Wizard 使用。

- Ragdoll Wizard 生成的 CharacterJoint 轴约定：**twist 轴对应肢体最大的 swing 轴，Swing 1 对应较小的 swing 轴，Swing 2 用于肢体扭转**。
- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-CharacterJoint.html

## 关键属性/行为

| 属性 | 说明（[FACT]） |
|---|---|
| **Connected Body / Connected Articulation Body** | 关节依赖的 Rigidbody / ArticulationBody；不设则连接世界。 |
| **Anchor / Axis** | 关节旋转中心的本地锚点；twist 轴（橙色 gizmo 锥）。 |
| **Auto Configure Connected Anchor / Connected Anchor** | 自动/手动配置连接锚点。 |
| **Swing Axis** | 摆动轴（绿色 gizmo 锥）。 |
| **Twist Limit Spring / Swing Limit Spring** | 沿 twist/swing 轴限制的弹性。Spring=0 表示限制**不可穿越**；非 0 则弹性。 |
| **Low/High Twist Limit、Swing 1/2 Limit** | 各轴限制角度（相对起始位置）。swing 对称（±30），twist 分低/高限。Limit 属性含 Limit/Bounciness/Contact Distance。 |
| **Enable Projection / Projection Distance / Projection Angle** | 关节越界时物理引擎如何将其"投影"回约束（None 或 Position and Rotation）。 |
| **Break Force / Break Torque** | 使关节断裂所需力/扭矩。 |
| **Enable Collision** | 是否允许被关节连接的两体相互碰撞。 |
| **Enable Preprocessing** | 关闭可稳定某些"不可能"配置。 |
| **Mass Scale / Connected Mass Scale** | 反转质量/惯性张量缩放，连接质量差异大的刚体时用于稳定。 |

- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-CharacterJoint.html
- 源：https://docs.unity3d.com/2022.3/Documentation/ScriptReference/CharacterJoint.html

## VRChat 特定限制/注意

**官方地位**：CharacterJoint 是 VRChat World 白名单组件（Unity Components → Physics/Colliders 段，见 [[../whitelisted-world-components.md|Allowlisted World Components]]）。**VRChat 官方无 CharacterJoint 专属页**；地位仅见于官方白名单清单。
- 源：https://creators.vrchat.com/worlds/whitelisted-world-components/

**[INFERENCE] World 实用场景（ragdoll）**：
- Ragdoll 效果：World 中做可被物理推倒并散架的 NPC/物体（需配合多段 Rigidbody + Collider + CharacterJoint + VRC Object Sync/所有权管理）。与 [[vrc-objectsync.md|VRC Object Sync]]、[[../../api/pickups.md|Pickups]] 的协同见两页。
- **重要限制**：关节连接的是 **Rigidbody**——而 VRChat 中动态物理对象通常会带 VRC Object Sync（同步 position/rotation/kinematic/gravity）。若用关节把多个同步刚体连成 ragdoll，**同步与关节解算可能冲突**（非拥有端/remote 端 kinematic 强制），需谨慎设计。此为结合 Rigidbody 同步已知问题的推断，官方无 ragdoll 专页。

**[FACT] Unity 物理学通用约束**（Kinematic 刚体 + Joint）：
- Kinematic Rigidbody 与 Joint 连接时，关节**无法对 kinematic body 施力**，只能调整非 kinematic 端以满足关节限制；非 kinematic body 也无法推动 kinematic body（RigidbodiesOverview）。详见 [[rigidbody.md|Rigidbody]]。
- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/RigidbodiesOverview.html

**[FACT] Quest 限制**：白名单页 Quest 脚注承认 Android 端存在例外清单，但 Quest 缺失列表**不含** Physics/Colliders 段组件——CharacterJoint 未列入已知 Quest 缺失清单；最终依据以官方 Quest 限制页为准。
- 源：https://creators.vrchat.com/platforms/android/quest-content-limitations#components

## 性能数据（unityvrchat 或 [UNKNOWN]）

`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）` **无 CharacterJoint 独立帧时间数据** → **`[UNKNOWN]`，无该组件独立性能数据**。

相关邻近事实（[FACT]，**非本体数据**）：Cloth 的 Collider 使布料帧时间约翻倍（`unityvrchat.md` 第 198 行）；PhysBones 无关节概念。无关节专用性能基准。

## 社区佐证

官方无专页。社区（VRChat World）有少量用 Unity 关节做可动机关/ragdoll 的教程，但官方并未专门背书。仅作普通佐证，不作为唯一来源。[COMMUNITY]

## 相关页面

[[../whitelisted-world-components.md|Allowlisted World Components]] · [[configurablejoint.md|ConfigurableJoint]] · [[rigidbody.md|Rigidbody]] · [[../../api/pickups.md|Pickups]] · [[vrc-objectsync.md|VRC Object Sync]]

## 源清单

- Unity Manual: Character Joint component reference (2022.3) — https://docs.unity3d.com/2022.3/Documentation/Manual/class-CharacterJoint.html
- Unity Scripting API: CharacterJoint — https://docs.unity3d.com/2022.3/Documentation/ScriptReference/CharacterJoint.html
- Unity Manual: Introduction to rigid body physics — https://docs.unity3d.com/2022.3/Documentation/Manual/RigidbodiesOverview.html
- VRChat: Allowlisted World Components — https://creators.vrchat.com/worlds/whitelisted-world-components/
- 本地性能源（无本组件数据）：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`