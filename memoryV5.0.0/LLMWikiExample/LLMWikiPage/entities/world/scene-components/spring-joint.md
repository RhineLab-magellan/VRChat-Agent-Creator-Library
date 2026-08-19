---
title: "SpringJoint — 世界物理组件"
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
  - SpringJoint
  - 弹簧关节
related:
  - ../whitelisted-world-components.md
  - rigidbody.md
  - hinge-joint.md
  - configurablejoint.md
  - index.md
type: entity
created: 2026-08-19
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-SpringJoint.html | https://docs.unity3d.com/ScriptReference/SpringJoint.html | https://creators.vrchat.com/worlds/whitelisted-world-components/"
updated: 2026-08-19
---
# SpringJoint — VRChat 原生 Unity 组件

> 调研日期：2026-08-19 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS

---

## 核心功能（官方，[FACT]）

SpringJoint 把 **两个 Rigidbody** 连在一起，但允许它们之间的距离像被弹簧连接一样变化——弹簧像一段有弹性的绳，试图把两个锚点拉回同一位置。拉力大小与当前距离成正比。

- 官方描述：连接两个刚体、允许距离变化，如同"一段弹性材料试着把两个锚点拉拢到一个位置"。
- 默认会**维持其起始距离**（rest position 保持初始间距），minDistance/maxDistance 基于该初始距离叠加。
- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-SpringJoint.html

## 关键属性/行为

| 属性 | 说明（[FACT]） |
|---|---|
| **Connected Body / Connected Articulation Body** | 连接的刚体；不设则接连世界（空间固定点）。官方也提到可用连接体串成链（说明处示例沿用 Hinge 链，但连接体机制通用）。 |
| **Anchor / Auto Configure Connected Anchor / Connected Anchor** | 本体与连接体上的锚点；Auto 保持场景内设定的起始距离。 |
| **Spring** | 单位距离的弹簧力——弹力按当前两点距离成比例，把两锚点拉回。 |
| **Damper** | 阻尼：抑制弹簧振荡，越高振荡更快衰减。与相对速度成正比地削减弹簧力。 |
| **Min Distance / Max Distance** | 在该距离区间内弹簧不施力；超过区间才施力拉回。 |
| **Tolerance** | 允许的误差/改变静止长度（rest length）。 |
| **Break Force / Break Torque** | 断裂力/扭矩。 |
| **Enable Collision / Enable Preprocessing / Mass Scale / Connected Mass Scale** | 连接两体是否碰撞、预处理稳定、质量差异稳定。 |

- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-SpringJoint.html
- 源（Scripting API：`spring`/`damper`/`minDistance`/`maxDistance`/`tolerance`）：https://docs.unity3d.com/ScriptReference/SpringJoint.html

## 典型用途（[FACT] 官方 + [INFERENCE] 应用）

- **模拟绳索/弹性**：官方把 Spring 比作"弹性材料拉拢锚点"，[FACT]。
- **荡秋千 / 摆绳 / 吊桥 / 弹性机关**：World 中做摆动吊桥、弹性缆绳、拉簧机关等。[INFERENCE——官方未列 VRChat 专属用例，属应用推断]

## Rigidbody 前件关系（[FACT]/[INFERENCE]）

- **前件**：两端都需 Rigidbody；不设 Connected Body 则连到空间固定点。
- **Kinematic 约束**：弹簧关节同样无法对 kinematic 刚体施力（Unity 刚体物理学通则），被弹簧拉动的应为非 kinematic 动态刚体，详见 [[rigidbody.md|Rigidbody]]。

## VRChat 特定限制/注意

**[FACT] 白名单地位**：SpringJoint 在官方 Allowlisted World Components「Unity Components → Physics/Colliders」段内，**World 可用**（见 [[../whitelisted-world-components.md|Allowlisted World Components]]）。**VRChat 官方无 SpringJoint 专属页**，地位仅见于官方白名单清单。

**[UNKNOWN] 边界**：官方未就 SpringJoint 在 VRChat 补充说明，以下**无法从官方来源确认**：
- 弹性/摆动对象在多人与 VRC Object Sync 下，弹簧持续施力与对象状态同步（position 同步、remote/非拥有端 kinematic）是否稳定——[INFERENCE]，非官方结论。
- Quest 端差异同其他 Physics/Colliders 段组件（不在已知缺失清单，最终以官方 Quest 限制页为准）。

**[INFERENCE] World 实用场景**：绳索、荡秋千、弹性吊桥等需要"距离弹簧力"的机关；与 [[vrc-objectsync.md|VRC Object Sync]]、[[../../api/pickups.md|Pickups]] 的协同需谨慎。

## 性能数据

**无 SpringJoint 独立性能基准 → `[UNKNOWN]`。** 官方与可获取源均未提供弹簧关节在 VRChat 世界的独立帧时间数据；无定量结论可引用。

## 相关页面

[[../whitelisted-world-components.md|Allowlisted World Components]] · [[rigidbody.md|Rigidbody]] · [[hinge-joint.md|HingeJoint]] · [[configurablejoint.md|ConfigurableJoint]] · [[vrc-objectsync.md|VRC Object Sync]] · [[../../api/pickups.md|Pickups]] · [[index.md|Scene Components 总览]]

## 源清单

- Unity Manual: Spring Joint component reference (2022.3) — https://docs.unity3d.com/2022.3/Documentation/Manual/class-SpringJoint.html
- Unity Scripting API: SpringJoint — https://docs.unity3d.com/ScriptReference/SpringJoint.html
- VRChat: Allowlisted World Components — https://creators.vrchat.com/worlds/whitelisted-world-components/
