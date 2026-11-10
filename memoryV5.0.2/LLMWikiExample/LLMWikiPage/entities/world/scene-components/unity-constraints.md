---
title: "Unity Constraints 组件参考（VRChat 白名单）"
category: world
subcategory: scene-components
knowledge_level: applied
status: active
source: "creators.vrchat.com/worlds/whitelisted-world-components/ + docs.unity3d.com (Unity 2022.3 Manual)"
source_type: official
version: 1.0
last_review: 2026-08-19
confidence: High
tags:
  - world
  - scene-components
  - constraint
  - unity-native
  - whitelist
aliases:
  - "Unity Constraints"
  - "Constraint 组件"
  - unityconstraint
related:
  - ../../avatar/vrc-constraints.md
  - ../../avatar/animator-system.md
  - ../whitelisted-world-components.md
  - index.md
  - ../../api/ui.md
type: entity
created: 2026-08-19
sources: "https://creators.vrchat.com/worlds/whitelisted-world-components/ (2026-11 官方核实 77 项) | Unity Manual 2022.3"
updated: 2026-08-19
---
# Unity Constraints 组件参考（VRChat 白名单）

> **白名单地位**: Unity 的 6 个 Constraint 组件均在官方 [Allowlisted World Components](../whitelisted-world-components.md)「Unity Components」内，**World 可用**。
> **关系**: VRChat 有自定义约束系统 → 见 [[../../avatar/vrc-constraints.md|VRChat Constraints]]。Avatar 上 Unity Constraint **上传时自动转换为 VRChat Constraints**；World 可任选一方。

---

## 6 类约束总览

| 组件 | Unity 类 | 用途 |
|------|---------|------|
| **AimConstraint** | `UnityEngine.Animations.AimConstraint` | 物体旋转，使指定轴瞄准一个或多个源 |
| **LookAtConstraint** | `UnityEngine.Animations.LookAtConstraint` | 类似 Aim 但用于模拟"注视"行为 |
| **ParentConstraint** | `UnityEngine.Animations.ParentConstraint` | 按多个源的加权组合做位置+旋转跟随 |
| **PositionConstraint** | `UnityEngine.Animations.PositionConstraint` | 位置跟随（可加权多个源） |
| **RotationConstraint** | `UnityEngine.Animations.RotationConstraint` | 旋转跟随（可加权多个源） |
| **ScaleConstraint** | `UnityEngine.Animations.ScaleConstraint` | 缩放跟随（可加权多个源） |

> 均在 `UnityEngine.Animations` 命名空间。它们实现 Unity 的 [Timeline/Constraint System](https://docs.unity3d.com/2022.3/Documentation/Manual/Constraints.html)。

---

## 与 VRChat Constraints 的关系（FACT）

| 方面 | Unity Constraints（本页） | VRChat Constraints |
|------|--------------------------|--------------------|
| Avatar | **自动转换**为 VRChat Constraints | ✅ 推荐直接使用 |
| World | 白名单内可直接用 | 也可用（SDK 3.10.0+） |
| 性能 | 一般 | 专为 VRChat 优化 |
| 功能 | 基础 | 更多特性（Freeze To World 等） |

> **FACT**: Avatar 若包含 Unity Constraints，**上传时自动转为 VRChat Constraints**。社区经验仍建议直接在 Avatar 上用 VRChat Constraints（[[../../avatar/vrc-constraints.md]]）。

---

## VRChat 特定注意

- **执行顺序**: 官方建议 **同一 GameObject 不要同时挂 Constraint 和 [VRCPhysBone](../../avatar/physbones.md)**（会造成执行顺序问题）——把 Constraint 放到父级 GameObject。
- **16 源上限**: VRChat Constraints API 中 Sources 有 16 个源的上限（见 [[../../avatar/vrc-constraints.md]] §Constraints API）。
- **Aim 轴配置**: AimConstraint 的 `AimAxis`/`UpAxis`/`WorldUpObject` 是高级用法关键；LookAt 通常兼顾上方向摆正。

---

## 相关

- [[../../avatar/vrc-constraints.md|VRChat Constraints]] — 6 种 VRChat 约束类型详解 + API
- [[../../avatar/animator-system.md|Animator 系统]]
- [[../whitelisted-world-components.md|Allowlisted World Components]] — 完整白名单
- [[index.md|Scene Components 总览]]
