---
title: "MeshCollider — 世界物理组件"
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
  - collider
  - mesh
aliases:
  - MeshCollider
  - 网格碰撞体
related:
  - ../whitelisted-world-components.md
  - boxcollider.md
  - rigidbody.md
  - ../../api/pickups.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-MeshCollider.html | https://docs.unity3d.com/2022.3/Documentation/ScriptReference/MeshCollider.html | https://creators.vrchat.com/worlds/whitelisted-world-components/"
updated: 2026-08-17
---
# MeshCollider — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无本组件独立帧时间数据，详见性能小节）

---

## 核心功能（官方，[FACT]）

Mesh Collider 取一个 **Mesh Asset** 并构建与该网格几何一致的碰撞体。对复杂 Mesh，它比 primitive（Box/Sphere/Capsule）更精确，是精确碰撞场景的更优选择。

- **标记为 Convex 的 MeshCollider 才能与其他 MeshCollider 碰撞**：非凸（Non-Convex）MeshCollider 只能对静态/半静态几何有效；Convex 模式有 **255 三角形上限**。
- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-MeshCollider.html

## 关键属性/行为

| 属性 | 说明（[FACT]） |
|---|---|
| **Convex** | 勾选后 MeshCollider 可与其他 MeshCollider 碰撞；**Convex 上限 255 三角形**。 |
| **Is Trigger** | 作触发器，触发 `OnTriggerEnter` / `OnTriggerStay` / `OnTriggerExit`。 |
| **Provides Contacts** | 开启后恒定生成接触信息，资源密集，默认关闭。 |
| **Cooking Options** | 控制物理引擎处理 Mesh 的方式（Cook for Faster Simulation / Enable Mesh Cleaning / Weld Colocated Vertices / Use Fast Midphase 等）。非默认设置要求 Mesh 的 `isReadable=true`；默认 Everything 开启。 |
| **Material** | 指定 Physic Material（摩擦/弹性）。 |
| **Mesh** | 用于碰撞的 Mesh 引用。 |
| **Layer Overrides** | 覆盖项目级 Layer 碰撞矩阵（Include/Exclude Layers + Priority）。 |

- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-MeshCollider.html
- 源：https://docs.unity3d.com/2022.3/Documentation/ScriptReference/MeshCollider.html

## VRChat 特定限制/注意

**官方地位**：MeshCollider 是 VRChat World 白名单组件（Unity Components → Physics/Colliders 段，见 [[../whitelisted-world-components.md|Allowlisted World Components]]）。**VRChat 官方无 MeshCollider 专属页**；地位仅见于官方白名单清单。
- 源：https://creators.vrchat.com/worlds/whitelisted-world-components/

**[INFERENCE] World 实用场景**：
- MeshCollider 常见于**自定义静态关卡几何**（地形/雕塑），需要精确碰撞时使用——静态背景下非凸（Non-Convex）MeshCollider 即可工作。
- 若要作为**动态可拾取/可推物体**：非凸 MeshCollider 无法与动态 Rigidbody 正确交互（只能对静态物体），通常需勾选 **Convex**（受 255 三角形限制）或用多个 primitive collider 近似——这是 Unity 物理学通用约束，World 上传时尤其要注意。与 [[rigidbody.md|Rigidbody]]、[[../../api/pickups.md|Pickups]] 的协同见两页。
- 与 VRC Contact **无关**（Contact 是独立组件，官方明确 "Contacts are separate from standard Unity colliders"），不要把 MeshCollider 与 VRC Contact 混淆。

**[INFERENCE/谨慎] 性能注意**：MeshCollider 的碰撞计算成本通常高于 primitive（Box/Sphere/Capsule 被官方明确标注 "low resource requirements"，而 MeshCollider 无此标注）。World 制作中建议对几何尽量用 primitive/Convex 近似。*（此为对比官方表述的推断，非单独性能数据。）*

**[FACT] Quest 限制**：白名单页 Quest 脚注承认 Android 端存在例外清单，但 Quest 缺失列表**不含** Physics/Colliders 段组件——MeshCollider 未列入已知 Quest 缺失清单；最终依据以官方 Quest 限制页为准。
- 源：https://creators.vrchat.com/platforms/android/quest-content-limitations#components

## 性能数据（unityvrchat 或 [UNKNOWN]）

`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）` **无 MeshCollider 独立帧时间数据** → **`[UNKNOWN]`，无该组件独立性能数据**。

相关邻近事实（[FACT]，**非本体数据**）：Cloth 的 Collider 使布料帧时间约翻倍，每 10 个 Collider 每个额外约 7%（`unityvrchat.md` 第 198 行）——间接提示碰撞体数量抬高物理/布料成本。

## 社区佐证

官方无专页。社区（VRChat World 教程）普遍提示「World 地面/大几何用 MeshCollider、动态物用 Convex 或 primitive」以减少成本。仅作 secondary 佐证，不作为唯一来源。[COMMUNITY]

## 相关页面

[[../whitelisted-world-components.md|Allowlisted World Components]] · [[boxcollider.md|BoxCollider]] · [[rigidbody.md|Rigidbody]] · [[../../api/pickups.md|Pickups]]

## 源清单

- Unity Manual: Mesh collider component reference (2022.3) — https://docs.unity3d.com/2022.3/Documentation/Manual/class-MeshCollider.html
- Unity Scripting API: MeshCollider — https://docs.unity3d.com/2022.3/Documentation/ScriptReference/MeshCollider.html
- VRChat: Allowlisted World Components — https://creators.vrchat.com/worlds/whitelisted-world-components/
- 本地性能源（无本组件数据）：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`