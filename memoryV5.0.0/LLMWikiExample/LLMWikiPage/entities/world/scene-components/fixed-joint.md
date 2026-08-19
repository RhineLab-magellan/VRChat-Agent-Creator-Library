---
title: "FixedJoint — 世界物理组件"
category: world
subcategory: scene-components
knowledge_level: applied
status: active
source: "Unity Manual 2022.3 + VRChat 官方文档"
source_type: official
version: 1.0
last_review: 2026-08-19
confidence: high
tags:
  - world
  - unity
  - physics
  - joint
aliases:
  - FixedJoint
  - 固定关节
related:
  - ../whitelisted-world-components.md
  - rigidbody.md
  - hinge-joint.md
  - configurablejoint.md
  - index.md
type: entity
created: 2026-08-19
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-FixedJoint.html | https://docs.unity3d.com/ScriptReference/FixedJoint.html | https://creators.vrchat.com/worlds/whitelisted-world-components/"
updated: 2026-08-19
---
# FixedJoint — VRChat 原生 Unity 组件

> 调研日期：2026-08-19 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS

---

## 核心功能（官方，[FACT]）

FixedJoint **把两个 Rigidbody 在绑定位置上刚性锁死**（"making them stick together in their bound position"，Scripting API），使本体的运动完全依赖另一物体的运动——本质上**无自由度**，类似父子（Parenting）关系，但走物理而非 Transform 层级。

- 官方定位：用于想"连接两个物体运动但不做父子"、或想方便地**让它们可断裂分开**（Break Force）的场景。
- 权衡：无需改 Hierarchy 层级即可达到连接效果，但**所有用到 FixedJoint 的物体都必须有 Rigidbody**。
- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-FixedJoint.html

## 关键属性/行为

| 属性 | 说明（[FACT]） |
|---|---|
| **Connected Body / Connected Articulation Body** | 关节依赖的 Rigidbody / ArticulationBody；不设则连接世界。 |
| **Break Force / Break Torque** | 使关节断裂所需力/扭矩。 |
| **Enable Collision** | 被连接的两物理体是否互相碰撞。 |
| **Enable Preprocessing** | 关闭可稳定"不可能"的配置。 |
| **Mass Scale / Connected Mass Scale** | 反转质量/惯性张量缩放（0.00001~∞）；质量差异大时用假质量让两刚体质量近似，以提升求解稳定度。 |

- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-FixedJoint.html
- 源（Scripting API：连在 bound position，无额外可配置自由度）：https://docs.unity3d.com/ScriptReference/FixedJoint.html

## Rigidbody 前件关系（[FACT]/[INFERENCE]）

- **前件**：被 FixedJoint 锁死的物体都需 Rigidbody。[FACT] 源：Unity Manual。
- **无自由度**：FixedJoint 提供的是**全刚性锁定**（六个自由度全部约束），与 [[configurablejoint.md|ConfigurableJoint]]（逐轴 Free/Locked/Limited）不同——这也是"父约束、但物理实现"。
- **与 kinematic 关系**：关节无法对 kinematic 刚体施力（Unity 刚体物理学通则）；因此用 FixedJoint 把动态物体锁到 kinematic/世界物体上时，实际由非 kinematic 端跟随，详见 [[rigidbody.md|Rigidbody]]。[FACT 通则 + INFERENCE 关节应用]

## 常见陷阱（[INFERENCE]）

- **移动父级**：若被 FixedJoint 锁定的物体同时挂在会变换位置的父级下，父级 Transform 运动与关节约束可能叠加/冲突，导致抖动或"拉扯"。建议要么走父子、要么走关节，避免两种约束同时叠加。
- **父子约束叠加**：对某物体同时做 Parent 和挂 FixedJoint，等于同一连接被约束两次，物理求解更易不稳定。此为社区常见经验与引擎约束叠加的推断，官方未就此给出专项说明。
- **断裂**：利用 Break Force 可实现"受足够大冲击后散架"的机关/部件（脆断机关），若需逐段可动应改用 [[hinge-joint.md|HingeJoint]]/CharacterJoint/ConfigurableJoint。

## VRChat 特定限制/注意

**[FACT] 白名单地位**：FixedJoint 在官方 Allowlisted World Components「Unity Components → Physics/Colliders」段内，**World 可用**（见 [[../whitelisted-world-components.md|Allowlisted World Components]]）。**VRChat 官方无 FixedJoint 专属页**，地位仅见于官方白名单清单。

**[UNKNOWN] 边界**：官方未就 FixedJoint 在 VRChat 的许可范围进一步说明，以下**无法从官方来源确认**：
- 多人与 VRC Object Sync 下，用 FixedJoint 把同步物体刚性锁定时，关节解算与对象状态同步（position/rotation 同步、remote/非拥有端 kinematic 强制）是否冲突——[INFERENCE]，非官方结论。
- Quest 端差异同其他 Physics/Colliders 段组件（不在已知缺失清单，最终以官方 Quest 限制页为准）。

**[INFERENCE] World 实用场景**：不动的"合体"机关、可冲击断裂的结构件、把动态物体附着到另一个动态物体上而不改 Hierarchy。与 [[vrc-objectsync.md|VRC Object Sync]]、[[../../api/pickups.md|Pickups]] 的协同需谨慎（同步 vs 关节约束）。

## 性能数据

**无 FixedJoint 独立性能基准 → `[UNKNOWN]`。** 官方与可获取源均未提供固定关节在 VRChat 世界的独立帧时间数据；无定量结论可引用。

## 相关页面

[[../whitelisted-world-components.md|Allowlisted World Components]] · [[rigidbody.md|Rigidbody]] · [[hinge-joint.md|HingeJoint]] · [[configurablejoint.md|ConfigurableJoint]] · [[vrc-objectsync.md|VRC Object Sync]] · [[../../api/pickups.md|Pickups]] · [[index.md|Scene Components 总览]]

## 源清单

- Unity Manual: Fixed Joint component reference (2022.3) — https://docs.unity3d.com/2022.3/Documentation/Manual/class-FixedJoint.html
- Unity Scripting API: FixedJoint — https://docs.unity3d.com/ScriptReference/FixedJoint.html
- VRChat: Allowlisted World Components — https://creators.vrchat.com/worlds/whitelisted-world-components/
