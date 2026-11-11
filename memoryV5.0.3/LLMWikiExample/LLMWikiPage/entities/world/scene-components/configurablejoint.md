---
title: "ConfigurableJoint — 世界物理组件"
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
  - ConfigurableJoint
  - 可配置关节
related:
  - ../whitelisted-world-components.md
  - characterjoint.md
  - rigidbody.md
  - ../../api/pickups.md
  - vrc-objectsync.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-ConfigurableJoint.html | https://docs.unity.cn/2022.3/Documentation/Manual/class-ConfigurableJoint.html | https://docs.unity3d.com/2022.3/Documentation/ScriptReference/ConfigurableJoint.html | https://discussions.unity.com/t/how-can-configurable-joint-be-set-up/918483 | https://creators.vrchat.com/worlds/whitelisted-world-components/"
updated: 2026-08-17
---
# ConfigurableJoint — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无本组件独立帧时间数据，详见性能小节）
> 注：调研时 docs.unity3d.com 该页跨域重定向，正文取自同内容镜像 docs.unity.cn 2022.3，并以官方 docs.unity3d.com URL 为规范引用。

---

## 核心功能（官方，[FACT]）

Configurable Joint 把其他所有关节类型的功能整合进来，并提供对角色运动的更强控制（尤其适合自定义 ragdoll 姿态、强制姿态，或将关节改造成高度定制关节）。它不仅限制运动，还能用 **drive force** 把对象驱动到目标位置/速度。

- 源（正文取镜像）：https://docs.unity3d.com/2022.3/Documentation/Manual/class-ConfigurableJoint.html（同内容镜像 https://docs.unity.cn/2022.3/Documentation/Manual/class-ConfigurableJoint.html）
- 源：https://docs.unity3d.com/2022.3/Documentation/ScriptReference/ConfigurableJoint.html

## 关键属性/行为

| 属性 | 说明（[FACT]） |
|---|---|
| **X/Y/Z Motion、Angular X/Y/Z Motion** | 每轴平移/旋转设 **Free / Locked / Limited**。Locked=完全锁死，Limited=限位内自由，Free=任意。可勾选 **Configured In World Space** 改用世界轴。 |
| **Axis / Secondary Axis** | 定义关节本地坐标系（第三轴为正交自动计算）。 |
| **Linear Limit** | 平移限位距离（Limit/Bounciness/Contact Distance）。 |
| **Angular X/Y/Z Limit (+Spring)** | 旋转限位角度；X 可设 Low/High 上下限，Y/Z 上下同角。 |
| **Target Position / Target Velocity / Target Rotation / Target Angular Velocity** | 驱动目标。 |
| **X/Y/Z Drive、Angular X/YZ Drive、Slerp Drive** | 施加到目标的驱动力；设定 Mode（Position/velocity/both）、Position Spring、Position Damper、Maximum Force。公式：`force = PositionSpring*(target - current) + PositionDamper*(targetVelocity - velocity)`。 |
| **Rotation Drive Mode** | X and YZ（分轴）或 Slerp（最小旋转，不可分轴）。 |
| **Enable Collision** | 连接的两体可否碰撞。 |
| **Enable Preprocessing** | 关闭可稳定"不可能"配置。 |
| **Break Force / Break Torque** | 断裂力/扭矩（Break Torque 仅当轴向为 Limited/Locked 时生效；Break Force 不受轴状态限制）。 |
| **Swap Bodies / Configured in World Space / Mass Scale / Connected Mass Scale** | 高级稳定性与坐标系控制。 |
| **Projection Mode / Distance / Angle** | 越界投影回约束。 |

- 源：https://docs.unity.cn/2022.3/Documentation/Manual/class-ConfigurableJoint.html（2022.3 官方镜像）
- 源：https://docs.unity3d.com/2022.3/Documentation/ScriptReference/ConfigurableJoint.html

## VRChat 特定限制/注意

**官方地位**：ConfigurableJoint 是 VRChat World 白名单组件（Unity Components → Physics/Colliders 段，见 [[../whitelisted-world-components.md|Allowlisted World Components]]）。**VRChat 官方无 ConfigurableJoint 专属页**；地位仅见于官方白名单清单。
- 源：https://creators.vrchat.com/worlds/whitelisted-world-components/

**[INFERENCE] World 实用场景（机关/机械）**：
- 高度可定制关节非常适合 **World 机关/机械**：门、铰链、升降机（forklift 例子官方明示）、旋转炮塔、起重机吊臂等，用 Drive 驱动目标。与 [[../../api/pickups.md|Pickups]]、[[vrc-objectsync.md|VRC Object Sync]] 的协同见两页。
- 与 [[characterjoint.md|CharacterJoint]] 相同的 **Rigidbody 依赖限制**：关节连接 Rigidbody；在 VRChat 多人与同步下（VRC Object Sync），动态关节对象的同步/远程 kinematic 强制可能与关节解算冲突，需谨慎设计。此为推断（官方无专页说明关节在 World 的详细用法）。

**[FACT] 白名单 Quest 限制**：白名单页 Quest 脚注承认 Android 端存在例外清单，但 Quest 缺失列表**不含** Physics/Colliders 段组件——ConfigurableJoint 未列入已知 Quest 缺失清单；最终依据以官方 Quest 限制页为准。
- 源：https://creators.vrchat.com/platforms/android/quest-content-limitations#components

## 性能数据（unityvrchat 或 [UNKNOWN]）

`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）` **无 ConfigurableJoint 独立帧时间数据**（该书不覆盖关节类组件，未来计划里也未列出关节）→ **`[UNKNOWN]`，无该组件独立性能数据**。

相关邻近事实（[FACT]，**非本体数据**）：Cloth 的 Collider 使布料帧时间约翻倍（`unityvrchat.md` 第 198 行）。无关节性能基准。

## 社区佐证

官方无专页。Unity 官方讨论区对 ConfigurableJoint 轴/坐标系配置有社区解答（如 https://discussions.unity.com/t/how-can-configurable-joint-be-set-up/918483），说明 Axis/Secondary Axis 与 X/Y/Z Motion 的对应关系——但这是 Unity 社区，非 VRChat 专属，仅作一般佐证。[SECONDARY]

## 相关页面

[[../whitelisted-world-components.md|Allowlisted World Components]] · [[characterjoint.md|CharacterJoint]] · [[rigidbody.md|Rigidbody]] · [[../../api/pickups.md|Pickups]] · [[vrc-objectsync.md|VRC Object Sync]]

## 源清单

- Unity Manual: Configurable Joint component reference (2022.3, docs.unity3d.com) — https://docs.unity3d.com/2022.3/Documentation/Manual/class-ConfigurableJoint.html
- Unity Manual: Configurable Joint (2022.3, 官方镜像内容同) — https://docs.unity.cn/2022.3/Documentation/Manual/class-ConfigurableJoint.html
- Unity Scripting API: ConfigurableJoint — https://docs.unity3d.com/2022.3/Documentation/ScriptReference/ConfigurableJoint.html
- Unity Discussions (secondary): Configurable Joint setup — https://discussions.unity.com/t/how-can-configurable-joint-be-set-up/918483
- VRChat: Allowlisted World Components — https://creators.vrchat.com/worlds/whitelisted-world-components/
- 本地性能源（无本组件数据）：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`