---
title: "CapsuleCollider — 世界组件"
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
  - CapsuleCollider
  - 胶囊碰撞体
related:
  - ../whitelisted-world-components.md
  - boxcollider.md
  - spherecollider.md
  - rigidbody.md
  - ../../api/pickups.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-CapsuleCollider.html | https://docs.unity3d.com/2022.3/Documentation/ScriptReference/CapsuleCollider.html | https://docs.unity3d.com/2022.3/Documentation/ScriptReference/Rigidbody-collisionDetectionMode.html | https://creators.vrchat.com/worlds/whitelisted-world-components/ | https://creators.vrchat.com/common-components/contacts/"
updated: 2026-08-17
---
# CapsuleCollider — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无本组件独立帧时间数据，详见性能小节）

---

## 核心功能（官方，[FACT]）

Capsule Collider 是内置 3D 胶囊形状碰撞体（两个半球 + 连接圆柱）。适合圆柱形物品，也常用于玩家/非玩家角色。因为**没有棱角**，可柔化关卡几何尖角处的碰撞区域，让玩家移动更平滑。**资源需求相对较低**（官方明示）。

- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-CapsuleCollider.html

## 关键属性/行为

| 属性 | 说明（[FACT]） |
|---|---|
| **Is Trigger** | 作触发器，触发 `OnTriggerEnter` / `OnTriggerStay` / `OnTriggerExit`。 |
| **Provides Contacts** | 恒定生成接触信息，资源密集，默认关闭。 |
| **Material** | Physic Material（摩擦/弹性）。 |
| **Center** | 本地空间位置，默认 (0,0,0)。 |
| **Radius** | 半径（从中心起），可与 Height 独立调整，默认 0.5。 |
| **Height** | 总高度（Unity units），可与 Radius 独立调整，默认 2。 |
| **Direction** | 胶囊长度方向轴（本地空间 X/Y/Z）。 |
| **Layer Overrides** | 覆盖 Layer 碰撞矩阵。 |

- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-CapsuleCollider.html
- 源：https://docs.unity3d.com/2022.3/Documentation/ScriptReference/CapsuleCollider.html

## VRChat 特定限制/注意

**官方地位**：CapsuleCollider 是 VRChat World 白名单组件（Unity Components → Physics/Colliders 段，见 [[../whitelisted-world-components.md|Allowlisted World Components]]）。**VRChat 官方无 CapsuleCollider 专属页**；地位仅见于官方白名单清单。
- 源：https://creators.vrchat.com/worlds/whitelisted-world-components/

**[INFERENCE] 与 VRC Contact / PhysBone / Pickup 的关系**（结合官方文档推断）：
- **VRC Contact** 的 Capsule 形状是专用 Contact 组件，与 Unity CapsuleCollider 独立（官方："separate from standard Unity colliders"）。Contact capsule 高度含两端半球，尺寸上限：半径 ≤3m、宽/高/深 ≤6m（缩放后生效）。
- CapsuleCollider 常作为拾取物 / PhysBone Collider 的原始形状之一（PhysBone Collider 本身是独立 VRCPhysBoneCollider，但可参考其几何）。详见 [[../../api/pickups.md|Pickups]]。
- 源：https://creators.vrchat.com/common-components/contacts/

**[FACT] Quest 限制**：白名单页 Quest 脚注承认 Android 端存在例外清单，但 Quest 缺失列表**不含** Physics/Colliders 段组件（仅 PPSv2 / Final IK / Dynamic Bone 等受限）——CapsuleCollider 未列入已知 Quest 缺失清单；最终依据以官方 Quest 限制页为准。
- 源：https://creators.vrchat.com/platforms/android/quest-content-limitations#components

**[FACT] CCD 支持**：Continuous Collision Detection 仅支持 Sphere/Capsule/Box 碰撞体（Rigidbody.collisionDetectionMode）；快速移动的 Rigidbody（如抛射物）若需防穿透可配 Capsule/Sphere + CCD，与 [[rigidbody.md|Rigidbody]] 协同。
- 源：https://docs.unity3d.com/2022.3/Documentation/ScriptReference/Rigidbody-collisionDetectionMode.html

## 性能数据（unityvrchat 或 [UNKNOWN]）

`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）` **无 CapsuleCollider 独立帧时间数据** → **无该组件独立性能数据**。

相关邻近事实（[FACT]，**非本体数据**）：Cloth 的 Collider 使布料帧时间约翻倍，每 10 个 Collider 每个额外约 7%（`unityvrchat.md` 第 198 行）——间接提示碰撞体数量抬高物理/布料成本。

## 社区佐证

[COMMUNITY] 官方无专页。社区普遍以 Capsule 作角色/可拾取物碰撞体。无强权威 secondary 源，仅作普通佐证。

## 相关页面

[[../whitelisted-world-components.md|Allowlisted World Components]] · [[boxcollider.md|BoxCollider]] · [[spherecollider.md|SphereCollider]] · [[rigidbody.md|Rigidbody]] · [[../../api/pickups.md|Pickups]]

## 源清单

- Unity Manual: Capsule collider component reference (2022.3) — https://docs.unity3d.com/2022.3/Documentation/Manual/class-CapsuleCollider.html
- Unity Scripting API: CapsuleCollider — https://docs.unity3d.com/2022.3/Documentation/ScriptReference/CapsuleCollider.html
- Unity Scripting API: Rigidbody.collisionDetectionMode — https://docs.unity3d.com/2022.3/Documentation/ScriptReference/Rigidbody-collisionDetectionMode.html
- VRChat: Allowlisted World Components — https://creators.vrchat.com/worlds/whitelisted-world-components/
- VRChat: Contacts — https://creators.vrchat.com/common-components/contacts/
- 本地性能源（无本组件数据）：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`