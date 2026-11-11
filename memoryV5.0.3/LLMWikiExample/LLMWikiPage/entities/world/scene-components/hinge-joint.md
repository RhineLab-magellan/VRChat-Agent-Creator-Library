---
title: "HingeJoint — 世界物理组件"
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
  - HingeJoint
  - 铰链关节
  - 合页关节
related:
  - ../whitelisted-world-components.md
  - characterjoint.md
  - configurablejoint.md
  - rigidbody.md
  - index.md
type: entity
created: 2026-08-19
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-HingeJoint.html | https://docs.unity3d.com/ScriptReference/HingeJoint.html | https://creators.vrchat.com/worlds/whitelisted-world-components/"
updated: 2026-08-19
---
# HingeJoint — VRChat 原生 Unity 组件

> 调研日期：2026-08-19 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS

---

## 核心功能（官方，[FACT]）

HingeJoint 把 **两个 Rigidbody** 连接起来，约束它们像被铰链（合页）连接一样运动。官方明确示例用途：**门（doors）**，也用于链（chains）、摆（pendulums）等铰链式运动物体。

- 单个 Hinge Joint 挂在承载它的 GameObject 上；其旋转发生在 **Anchor**（锚点）位置、绕 **Axis**（轴）方向转动。
- 除了基础连接，HingeJoint 带有 **Spring（弹簧）**、**Motor（电机）**、**Limits（限位）** 三组可选行为，用于精细控制关节运动。
- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-HingeJoint.html

## 关键属性/行为

| 属性 | 说明（[FACT]） |
|---|---|
| **Connected Body / Connected Articulation Body** | 关节依赖的 Rigidbody / ArticulationBody；不设则连接世界。可借连接体把多个 HingeJoint 串成一条链（每节加一个关节、把下一节设为 Connected Body）。 |
| **Anchor / Axis** | 自带组件本体的摆动中心（局部空间点）与摆动轴（局部空间方向）。例：普通门的铰链在 Y 轴，猫门/宠物门在相对 X 轴。 |
| **Auto Configure Connected Anchor / Connected Anchor** | 自动（默认）按全局 Anchor 位置算 Connected Anchor，或手动配置。 |
| **Use Spring → Spring / Damper / Target Position** | 让刚体向某个**目标角度**（度）被拉回。Spring=牵引力大小，Damper=越大减速越明显，Target Position=目标角度。**官方明示：Spring 与 Motor 互斥，同时使用结果不可预测。** |
| **Use Motor → Target Velocity / Force / Free Spin** | 让物体绕关节轴转动。Target Velocity=目标角速度（度/秒），Force=达到该速度施加的力，Free Spin=开启则电机永不降低加速度。 |
| **Use Limits → Min / Max / Bounciness / Contact Distance** | 限制铰链角度在 Min~Max 范围。Bounciness=撞到限位时的回弹，Contact Distance=限位附近避免抖动的最小接触容差。 |
| **Extended Limits** | 把角度范围扩展到 [-360, 360] 度。 |
| **Use Acceleration** | 让 Spring 输出加速度而非力。 |
| **Break Force / Break Torque** | 使关节断裂所需的力/扭矩。 |
| **Enable Collision / Enable Preprocessing / Mass Scale / Connected Mass Scale** | 是否允许被连接两体互相碰撞；关闭 Preprocessing 可稳定"不可能满足"的配置；质量差异大时用 Mass Scales 施加假质量以稳定求解。 |

- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-HingeJoint.html
- 源（Scripting API，`useSpring`/`useMotor`/`useLimits` 默认关闭，`angle`/`velocity` 只读）：https://docs.unity3d.com/ScriptReference/HingeJoint.html

## Rigidbody 前件关系（[FACT]/[INFERENCE]）

- **前件**：关节连接的是 **Rigidbody**——被铰链连接的两个物体都需有 Rigidbody（至少要有一端是动态刚体）。[FACT] 源：Unity Manual（Joints 概览、class-HingeJoint）
- **Kinematic 约束**：关节无法对 kinematic（运动学）刚体施力，只能调整非 kinematic 端以满足关节限制；非 kinematic 刚体也无法推动 kinematic 刚体。详见 [[rigidbody.md|Rigidbody]]。[FACT]（Unity RigidbodiesOverview 泛化自关节类，与 [[characterjoint.md|CharacterJoint]]/[[configurablejoint.md|ConfigurableJoint]] 同一条约束）

## VRChat 特定限制/注意

**[FACT] 白名单地位**：HingeJoint 在官方 Allowlisted World Components「Unity Components → Physics/Colliders」段内，**World 可用**（见 [[../whitelisted-world-components.md|Allowlisted World Components]]）。**VRChat 官方无 HingeJoint 专属页**，地位仅见于官方白名单清单。

**[UNKNOWN] 边界**：官方未就 HingeJoint 在 VRChat 的许可范围给出更多说明，以下事项**无法从官方来源确认**：
- 铰链式机关在多人与 VRC Object Sync 下，关节解算与对象状态同步（position/rotation 同步、remote/非拥有端 kinematic 强制）是否冲突——参照 [[characterjoint.md|CharacterJoint]]/[[configurablejoint.md|ConfigurableJoint]] 页的同类保留意见，属推断（[INFERENCE]），非官方结论。
- Quest（Android）端是否存在额外差异——白名单页脚注仅声明"Android 端有例外清单"，Physics/Colliders 段组件不在已知缺失清单内，最终以官方 Quest 限制页为准。

**[INFERENCE] World 实用场景**：门/舱门、可开合机关、链条、起重机吊臂等旋转类机关；结合 Use Limits 限制开门角度、Motor 做自动门/转轴。与 [[vrc-objectsync.md|VRC Object Sync]]、[[../../api/pickups.md|Pickups]] 的协同需像其他关节一样谨慎（同步 vs 关节解算）。

## 性能数据

**无 HingeJoint 独立性能基准 → `[UNKNOWN]`。** 官方与可获取源均未提供铰链关节在 VRChat 世界的独立帧时间数据；关节解算对物理步进的定性影响见 [[configurablejoint.md|ConfigurableJoint]]/[[characterjoint.md|CharacterJoint]]，无定量结论可引用。

## 相关页面

[[../whitelisted-world-components.md|Allowlisted World Components]] · [[characterjoint.md|CharacterJoint]] · [[configurablejoint.md|ConfigurableJoint]] · [[rigidbody.md|Rigidbody]] · [[vrc-objectsync.md|VRC Object Sync]] · [[../../api/pickups.md|Pickups]] · [[index.md|Scene Components 总览]]

## 源清单

- Unity Manual: Hinge Joint component reference (2022.3) — https://docs.unity3d.com/2022.3/Documentation/Manual/class-HingeJoint.html
- Unity Scripting API: HingeJoint — https://docs.unity3d.com/ScriptReference/HingeJoint.html
- VRChat: Allowlisted World Components — https://creators.vrchat.com/worlds/whitelisted-world-components/
