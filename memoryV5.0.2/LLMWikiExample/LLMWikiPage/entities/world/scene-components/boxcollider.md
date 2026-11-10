---
title: "BoxCollider — 世界组件"
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
  - BoxCollider
  - 盒碰撞体
related:
  - ../whitelisted-world-components.md
  - capsulecollider.md
  - spherecollider.md
  - rigidbody.md
  - ../../api/pickups.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-BoxCollider.html | https://docs.unity3d.com/2022.3/Documentation/ScriptReference/BoxCollider.html | https://creators.vrchat.com/worlds/whitelisted-world-components/ | https://creators.vrchat.com/worlds/components/vrc_pickup | https://creators.vrchat.com/common-components/contacts/"
updated: 2026-08-17
---
# BoxCollider — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无本组件独立帧时间数据，详见性能小节）

---

## 核心功能（官方，[FACT]）

Box Collider 是内置的立方体形状碰撞体：适合箱子/板条箱等物品，或拉伸压扁成台面、台阶等任意长方体对象。**低多边形、资源需求低**（官方明示）。

- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-BoxCollider.html

## 关键属性/行为

| 属性 | 说明（[FACT]） |
|---|---|
| **Is Trigger** | 开启后作为触发器：其他碰撞体穿过它，并触发 `OnTriggerEnter` / `OnTriggerStay` / `OnTriggerExit`。 |
| **Provides Contacts** | 开启后恒定生成接触信息（即使无人消费）。接触生成**资源密集**，默认关闭。 |
| **Material** | 指定 Physic Material（摩擦/弹性）。 |
| **Center** | 碰撞体在 GameObject 本地空间的位置，默认 (0,0,0)。 |
| **Size** | 各轴尺寸（Unity units），默认 (1,1,1)。 |
| **Layer Overrides** | 覆盖项目级 Layer 碰撞矩阵（Include/Exclude Layers + Priority）。 |

- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-BoxCollider.html
- 源：https://docs.unity3d.com/2022.3/Documentation/ScriptReference/BoxCollider.html

## VRChat 特定限制/注意

**官方地位**：BoxCollider 是 VRChat World 白名单组件（Unity Components → Physics/Colliders 段，见 [[../whitelisted-world-components.md|Allowlisted World Components]]）。**VRChat 官方无 BoxCollider 专属页**；地位仅见于官方白名单清单。
- 源：https://creators.vrchat.com/worlds/whitelisted-world-components/

**[INFERENCE] Collider 与 VRC Contact / Pickup / Interact 的关系**（结合官方文档推断）：
- **VRC Pickup** 要求一个 Collider——BoxCollider 是拾取物常用的最简单碰撞体。
- **Interact**（UdonBehaviour 的 Interact 事件）要求对象有 Collider（Community 教程普遍要求 Box/Sphere/etc.）。
- **VRC Contact（Sender/Receiver）** 使用 Sphere/Capsule/Box 三类形状，但它们是**独立的 Contact 组件**，与 Unity 原生 Collider 无关（官方明确 "Contacts are separate from standard Unity colliders"）——不要把 BoxCollider 与 VRCContact 混淆。详见 [[../../api/pickups.md|Pickups]]。
- 源：https://creators.vrchat.com/worlds/components/vrc_pickup
- 源：https://creators.vrchat.com/common-components/contacts/

**[FACT] Quest 限制**：白名单页 Quest 脚注承认 Android 端存在例外清单，但 Quest 缺失列表**不含** Physics/Colliders 段组件（仅 PPSv2 / Final IK / Dynamic Bone 等受限）——BoxCollider 未列入已知 Quest 缺失清单；最终依据以官方 Quest 限制页为准。
- 源：https://creators.vrchat.com/platforms/android/quest-content-limitations#components

**[FACT] 玩家默认不允许被经 Unity Collider 的物理碰撞推挤**：VRChat 玩家碰撞由 VRChat 玩家控制器管理（玩家胶囊体特殊处理），一般对 Rigidbody 物理碰撞有特殊规则。若要让玩家与物块物理交互，多用 VRCObjectSync + 可拾取对象策略。*（此为社区普遍理解，官方无专页明文。）*

## 性能数据（unityvrchat 或 [UNKNOWN]）

`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）` **无 BoxCollider 独立帧时间数据** → **无该组件独立性能数据**。

相关邻近事实（[FACT]，**非本体数据**）：Cloth 的 Collider 使布料帧时间约翻倍，每 10 个 Collider 每个额外约 7%（`unityvrchat.md` 第 198 行）——间接提示碰撞体数量抬高物理/布料成本。

## 社区佐证

社区普遍将 BoxCollider 用作拾取物 / Interact 类的最轻量碰撞体（见 Ask Forum / World 教程）。[COMMUNITY] 无官方专页，仅作 secondary 佐证，不作为唯一来源。

## 相关页面

[[../whitelisted-world-components.md|Allowlisted World Components]] · [[capsulecollider.md|CapsuleCollider]] · [[spherecollider.md|SphereCollider]] · [[meshcollider.md|MeshCollider]] · [[rigidbody.md|Rigidbody]] · [[../../api/pickups.md|Pickups]]

## 源清单

- Unity Manual: Box collider component reference (2022.3) — https://docs.unity3d.com/2022.3/Documentation/Manual/class-BoxCollider.html
- Unity Scripting API: BoxCollider — https://docs.unity3d.com/2022.3/Documentation/ScriptReference/BoxCollider.html
- VRChat: Allowlisted World Components — https://creators.vrchat.com/worlds/whitelisted-world-components/
- VRChat: VRC Pickup — https://creators.vrchat.com/worlds/components/vrc_pickup
- VRChat: Contacts — https://creators.vrchat.com/common-components/contacts/
- 本地性能源（无本组件数据）：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`