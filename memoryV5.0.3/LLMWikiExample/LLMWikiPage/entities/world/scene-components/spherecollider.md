---
title: "SphereCollider — 世界组件"
category: world
subcategory: scene-components
knowledge_level: applied
status: active
source: "Unity Manual 2022.3 + VRChat 官方文档"
source_type: official
version: 1.0
last_review: 2026-08-17
confidence: high
tags:
  - world
  - unity
  - physics
  - collider
  - pickup
aliases:
  - SphereCollider
  - 球体碰撞体
related:
  - ../whitelisted-world-components.md
  - boxcollider.md
  - capsulecollider.md
  - rigidbody.md
  - ../../api/pickups.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-SphereCollider.html | https://docs.unity3d.com/2022.3/Documentation/ScriptReference/SphereCollider.html | https://docs.unity3d.com/2022.3/Documentation/ScriptReference/Rigidbody-collisionDetectionMode.html | https://creators.vrchat.com/worlds/whitelisted-world-components/ | https://creators.vrchat.com/worlds/components/vrc_pickup | https://creators.vrchat.com/common-components/contacts/"
updated: 2026-08-17
---
# SphereCollider — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无本组件独立帧时间数据，详见性能小节）

---

## 核心功能（官方，[FACT]）

Sphere Collider 是内置球体碰撞体：适合球/弹珠等物品，可拉伸缩放成抛射物、行星等球状对象，也能用于需要滚动/翻滚的物体（如下落的巨石）。**资源需求相对较低**（官方明示）。

- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-SphereCollider.html

## 关键属性/行为

| 属性 | 说明（[FACT]） |
|---|---|
| **Is Trigger** | 开启后作触发器，被物理引擎忽略于实体碰撞，触发 `OnTrigger*`。 |
| **Provides Contacts** | 恒定生成接触信息，资源密集，默认关闭。 |
| **Material** | Physic Material（摩擦/弹性）。 |
| **Center** | 本地空间位置。 |
| **Radius** | 唯一可调大小的属性——**不能按单轴压扁**成非球体形状（官方明示）。 |
| **Layer Overrides** | 覆盖 Layer 碰撞矩阵。 |

- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-SphereCollider.html
- 源：https://docs.unity3d.com/2022.3/Documentation/ScriptReference/SphereCollider.html

## VRChat 特定限制/注意

**官方地位**：SphereCollider 是 VRChat World 白名单组件（Unity Components → Physics/Colliders 段，见 [[../whitelisted-world-components.md|Allowlisted World Components]]）。**VRChat 官方无 SphereCollider 专属页**；地位仅见于官方白名单清单。
- 源：https://creators.vrchat.com/worlds/whitelisted-world-components/

**[INFERENCE] 与 VRC Contact / Pickup 关系**（结合官方文档推断）：
- **VRC Contact** 的 Sphere 形状是独立 Contact 组件，与 Unity SphereCollider 无关（官方："separate from standard Unity colliders"）。Contact sphere 半径上限 ≤3m（缩放后生效）。
- Sphere（或 Capsule/Box）Collider 是 VRC Pickup "Requires: Collider" 的常见满足方式——拾取物常用 Sphere 碰撞体以便随手抓取。详见 [[../../api/pickups.md|Pickups]]。
- 源：https://creators.vrchat.com/common-components/contacts/
- 源：https://creators.vrchat.com/worlds/components/vrc_pickup

**[FACT] Quest 限制**：白名单页 Quest 脚注承认 Android 端存在例外清单，但 Quest 缺失列表**不含** Physics/Colliders 段组件（仅 PPSv2 / Final IK / Dynamic Bone 等受限）——SphereCollider 未列入已知 Quest 缺失清单；最终依据以官方 Quest 限制页为准。
- 源：https://creators.vrchat.com/platforms/android/quest-content-limitations#components

**[FACT] CCD 支持**：Continuous Collision Detection 仅支持 Sphere/Capsule/Box 碰撞体；快速移动的 Rigidbody（如抛射物）可用 Sphere + CCD 防穿透，与 [[rigidbody.md|Rigidbody]] 协同。
- 源：https://docs.unity3d.com/2022.3/Documentation/ScriptReference/Rigidbody-collisionDetectionMode.html

## 性能数据（unityvrchat 或 [UNKNOWN]）

`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）` **无 SphereCollider 独立帧时间数据** → **无该组件独立性能数据**。

相关邻近事实（[FACT]，**非本体数据**）：Cloth 的 Collider 使布料帧时间约翻倍，每 10 个 Collider 每个额外约 7%（`unityvrchat.md` 第 198 行）——间接提示碰撞体数量抬高物理/布料成本。

## 社区佐证

[COMMUNITY] 官方无专页。社区普遍以 Sphere 作拾取物/可滚动物体/触发逻辑的轻量碰撞体。无强权威 secondary 源，仅作普通佐证。

## 相关页面

[[../whitelisted-world-components.md|Allowlisted World Components]] · [[boxcollider.md|BoxCollider]] · [[capsulecollider.md|CapsuleCollider]] · [[rigidbody.md|Rigidbody]] · [[../../api/pickups.md|Pickups]]

## 源清单

- Unity Manual: Sphere collider component reference (2022.3) — https://docs.unity3d.com/2022.3/Documentation/Manual/class-SphereCollider.html
- Unity Scripting API: SphereCollider — https://docs.unity3d.com/2022.3/Documentation/ScriptReference/SphereCollider.html
- Unity Scripting API: Rigidbody.collisionDetectionMode — https://docs.unity3d.com/2022.3/Documentation/ScriptReference/Rigidbody-collisionDetectionMode.html
- VRChat: Allowlisted World Components — https://creators.vrchat.com/worlds/whitelisted-world-components/
- VRChat: VRC Pickup — https://creators.vrchat.com/worlds/components/vrc_pickup
- VRChat: Contacts — https://creators.vrchat.com/common-components/contacts/
- 本地性能源（无本组件数据）：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`