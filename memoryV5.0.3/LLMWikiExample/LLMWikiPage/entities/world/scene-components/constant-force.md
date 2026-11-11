---
title: "ConstantForce — 世界物理组件"
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
aliases:
  - ConstantForce
  - 恒定力
related:
  - ../whitelisted-world-components.md
  - rigidbody.md
  - ../../api/dynamics.md
  - index.md
type: entity
created: 2026-08-19
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-ConstantForce.html | https://docs.unity3d.com/ScriptReference/ConstantForce.html | https://creators.vrchat.com/worlds/whitelisted-world-components/"
updated: 2026-08-19
---
# ConstantForce — VRChat 原生 Unity 组件

> 调研日期：2026-08-19 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS

---

## 核心功能（官方，[FACT]）

ConstantForce 给一个 **Rigidbody** 持续施加恒定力，适合"随时间加速"的物体运动。它区别于 `Rigidbody.AddForce`（只作用一帧、需每帧调用）：**ConstantForce 每帧持续施加力/扭矩**，直到你改值为止。

- 官方明示：若把 ConstantForce 加到**没有 Rigidbody** 的 GameObject 上，Unity 会自动添加一个 Rigidbody。
- 继承自 Behaviour（可 `enabled` 开关）。
- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-ConstantForce.html
- 源（Scripting API）：https://docs.unity3d.com/ScriptReference/ConstantForce.html

## 关键属性/行为（[FACT]）

四个字段，区分**世界/局部空间**；值越大、力越强、初始速度越快：

| 字段 | 空间 | 说明 |
|---|---|---|
| **Force** | **世界**（scene 全局轴） | 线性力方向/大小（XYZ 对应全局轴）。 |
| **Relative Force** | **局部**（Rigidbody 本地轴） | 线性力，跟随刚体自身朝向。 |
| **Torque** | **世界**（全局轴） | 刚体绕其旋转的扭矩。 |
| **Relative Torque** | **局部**（Rigidbody 本地轴） | 绕刚体自身局部轴旋转的扭矩。 |

- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-ConstantForce.html
- 源（Scripting API：`force`/`relativeForce`/`torque`/`relativeTorque` 每帧施加）：https://docs.unity3d.com/ScriptReference/ConstantForce.html

## VRChat 特定限制/注意

**[FACT] 白名单地位**：ConstantForce 在官方 Allowlisted World Components「Unity Components → Physics/Colliders」段内，**World 可用**（见 [[../whitelisted-world-components.md|Allowlisted World Components]]）。**VRChat 官方无 ConstantForce 专属页**，地位仅见于官方白名单清单。

**[FACT] 前件：需要附着刚体才生效**——ConstantForce 只作用于 Rigidbody；无刚体时 Unity 自动补一个，但若目标是**不跟随物理的静态物体则力不产生可见效果**（力只影响动态刚体）。详见 [[rigidbody.md|Rigidbody]]。

**[INFERENCE] 对 Kinematic 刚体无效**：恒定力作用于 kinematics 刚体时不产生运动学响应（Kinematic 刚体忽略力），这类用法无效果——推断，非官方专述。

**[UNKNOWN] 边界**：官方未就 ConstantForce 在 VRChat 补充说明，以下**无法从官方来源确认**：
- World 中恒力推进器/漂浮物在多人与 VRC Object Sync 下的同步稳定性（持续施力 vs 对象状态同步）。
- Quest 端差异同其他 Physics/Colliders 段组件（不在已知缺失清单，最终以官方 Quest 限制页为准）。

**[INFERENCE] World 实用场景**：推进器（喷气/飞船加速）、漂浮物（恒定上升气流、漂浮平台）、持续旋转装置（扭矩）。与 [[../../api/dynamics.md|Dynamics API]]、[[vrc-objectsync.md|VRC Object Sync]] 的协同需谨慎。

## 性能数据

**无 ConstantForce 独立性能基准 → `[UNKNOWN]`。** 官方与可获取源均未提供恒力组件在 VRChat 世界的独立帧时间数据；无定量结论可引用。

## 相关页面

[[../whitelisted-world-components.md|Allowlisted World Components]] · [[rigidbody.md|Rigidbody]] · [[vrc-objectsync.md|VRC Object Sync]] · [[../../api/dynamics.md|Dynamics API]] · [[../../api/pickups.md|Pickups]] · [[index.md|Scene Components 总览]]

## 源清单

- Unity Manual: Constant Force component reference (2022.3) — https://docs.unity3d.com/2022.3/Documentation/Manual/class-ConstantForce.html
- Unity Scripting API: ConstantForce — https://docs.unity3d.com/ScriptReference/ConstantForce.html
- VRChat: Allowlisted World Components — https://creators.vrchat.com/worlds/whitelisted-world-components/
