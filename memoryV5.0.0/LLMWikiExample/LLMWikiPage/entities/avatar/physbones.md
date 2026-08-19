---
title: "VRCPhysBone 组件参考"
category: avatar
knowledge_level: applied
status: active
source: "creators.vrchat.com/common-components/physbones/ (2026-06-17)"
source_type: official
version: 1.0
last_review: 2026-08-19
confidence: High
tags:
  - avatar
  - world
  - physbone
  - dynamic-bone
  - common-components
aliases:
  - "VRCPhysBone"
  - "PhysBones"
  - 物理骨骼
  - "Phys Bone"
related:
  - vrc-constraints.md
  - performance-rank.md
  - quest-constraints.md
  - ../../concepts/rules/quest-constraints.md
  - ../api/dynamics.md
  - ../../entities/world/scene-components/index.md
  - ma2bt.md
type: entity
created: 2026-08-19
sources: "https://creators.vrchat.com/common-components/physbones/"
updated: 2026-08-19
---
# VRCPhysBone 组件参考

> **来源**: 官方 [PhysBones](https://creators.vrchat.com/common-components/physbones/)（2026-06-17 更新）。组件对 **Avatar 与 World** 均可用（World 端用 Udon 交互，见下）。

---

## 概述

PhysBones 是一组组件，为 Avatar/World 中的对象添加**次级运动**（头发、尾巴、耳朵、衣物、线缆、植物等）。

- **VRCPhysBone**: 定义一条受物理影响的骨骼链（核心组件）。
- **VRCPhysBoneCollider**: 定义与 PhysBone 碰撞的形状。
- **VRCPhysBoneRoot**: 定义一组 PhysBone 的运动根（**仅 World 可用**）。

三个组件配合，可实现软体/次级运动模拟。

---

## 1. VRCPhysBone

### 1.1 版本（Versions）

| 版本 | 特点 |
|------|------|
| **1.0** | 基础版本 |
| **1.1** | Squishy Bones 更新：骨骼可压缩/被运动拉长；Gravity 作为静止时旋转的比例（需正 Pull）；Stiffness 作为保持先前方向的比值 |

> **FACT**: 新建组件默认最新版本；已有 PhysBone 保持原版本直到手动升级并重新上传。

### 1.2 Transforms

| 参数 | 说明 |
|------|------|
| **Root Transform** | 组件起始的 Transform；留空则假设从该 GameObject 开始 |
| **Ignore Transforms** | 忽略的变换列表（含其子级） |
| **Ignore Other Phys Bones** | 忽略场景层级中此组件下方的其他 PhysBone（默认启用；关闭可能导致不支持行为） |
| **Endpoint Position** | 在链端点创建额外骨骼的向量；**仅非零时有意义**。通常沿 +Y 增加（指向骨骼"上方"） |
| **Multi-Child Type** | 多骨骼链时根骨骼的行为：`Ignore`（根不动，无视物理；适合头发）/ `First`（根与第一条链连成整体，其他链从各自首骨开始）/ `Average`（根的运动为所有链平均值） |

> ⚠️ **FACT（必须设置端点）**: 若使用单一 `RootBone`，或单一根 + 多个子骨骼（无孙级），**必须定义 Endpoint Position** 否则 PhysBone 不工作（与 Dynamic Bones 不同）。

### 1.3 Forces

**Integration Type** 决定模拟数学方式（`Simplified` 稳定简单 / `Advanced` 可配置更复杂、对外力更灵敏）。

> **INFO**: 大部分 Forces/Limits/… 选项支持 **Curves**（滑块旁 C 按钮），可沿骨骼链长度调整值，实现非常复杂的链内设置。

| 参数 | Simplified | Advanced | 说明 |
|------|:--:|:--:|------|
| **Pull** | ✅ | ✅ | 骨骼回到静止位置的力度 |
| **Spring** | ✅ | ❌ | 尝试回到静止位置的摆动量 |
| **Momentum** | ❌ | ✅ | 摆动量（效果与 Spring 略有不同） |
| **Stiffness** | ❌ | ✅ | 保持静止位置的程度 |
| **Gravity** | ✅ | ✅ | 重力大小；正值下拉、负值上拉 |
| **Gravity Falloff** | ✅ | ✅ | Gravity 非零时可用：静止位置时移除 Gravity 的程度；1.0=静止时完全不受重力影响 |
| **Immobile Type** | ✅ | ✅ | `All Motion`（默认：根父级计算的任何运动都被 Immobile 阻尼）/ `World (Experimental)`（仅抵消场景根参考系的位置移动，动画/IK 不受影响） |

**Gravity Falloff 用法示例**: 头发静止时已是想要的姿势（如站姿自然垂落），用 1.0 falloff——站立时重力不影响，保持建模姿势；若头发建模成 45° 伸出且想要自然弧度，用 0.5–0.8 只让部分重力生效。

### 1.4 Limits（限制，比 Collider 更高效）

用于限制骨骼链移动范围（如防头发插入头部）；Scene 视图会显示可视化限制。

| 模式 | 说明 |
|------|------|
| **None** | 无限制 |
| **Angle** | 限制到 `Max Angle`，以 `Rotation` 为中心轴（Scene 中显示为锥体） |
| **Hinge** | 限制到某平面内的 `Max Angle`（显示为圆切片/披萨片） |
| **Polar** | 将 Hinge 沿 Yaw 扫出球面段；可配 `Max Pitch` + `Max Yaw` + `Rotation`。**有性能开销**，超过 ~64 个可能出问题；若 Pitch≈Yaw 用 Angle 更省 |

### 1.5 Collision（碰撞）

| 参数 | 说明 |
|------|------|
| **Radius** | 每骨骼碰撞半径（米），同时用于碰撞与抓取 |
| **Allow Collision** | `True`（与全局碰撞体碰撞，含每玩家手部）/ `False`（仅与 Colliders 列表碰撞）/ `Other`（Avatar/World 各自过滤规则） |
| **Colliders** | 专门与此骨骼碰撞的碰撞体列表 |

### 1.6 Stretch & Squish

| 参数 | 说明 |
|------|------|
| **Stretch Motion** | 运动对骨骼拉伸/压缩的影响；0=仅抓取/碰撞时拉伸压缩 |
| **Max Stretch** | 最大拉伸量（原始骨骼长度的倍数，受最大边界限制） |
| **Max Squish** | 最大压缩量（原始骨骼长度的倍数） |

### 1.7 Grab & Pose（抓握与摆姿）

| 参数 | 说明 |
|------|------|
| **Allow Grabbing** | 允许玩家抓握骨骼（Avatar 可过滤：穿戴者/其他玩家） |
| **Allow Posing** | 允许抓后锁定姿势（头像可过滤） |
| **Grab Movement** | 抓握后骨骼如何移动：0=用 Pull/Spring 逐渐到达抓取位；1=立即到达 |
| **Snap To Hand** | 抓取时骨骼吸附到抓取者的手 |

> **FACT**: 抓握后**扣住 Trigger 可"Pose"骨骼**并保持其位置；其他玩家也可抓/摆你的 PhysBone（若交互设置允许）。

### 1.8 Options（Avatar）

| 参数 | 说明 |
|------|------|
| **Parameter** | 前缀，为动画控制器提供参数：`{parameter}_IsGrabbed` (Bool) / `{parameter}_IsPosed` (Bool) / `{parameter}_Angle` (Float 0-1，末端骨相对原始位置的角度) / `{parameter}_Stretch` (Float 0-1) / `{parameter}_Squish` (Float 0-1) |
| **Is Animated** | 允许骨骼 Transform 被动画（每帧按动画更新静止位置）；**必须启用**若要让链内骨骼（含 Root）响应动画 |
| **Reset When Disabled** | 组件被禁用时骨骼自动重置到默认位置 |

> **FACT（AV3 参数）**: PhysBone 参数**无需使用 Synced Parameters**（`VRCExpressionParameters`）——本地与远端都会运行 PhysBones，参数已同步。

### 1.9 Gizmos

`Show Gizmos` / `Bone Opacity` / `Limit Opacity` —— 仅编辑器可视化，不影响运行行为。

---

## 2. 重要限制与陷阱（FACT）

- **硬限制（Quest/Avatar）**: Avatar 端 PhysBones 在 **Meta Quest** 有硬限制（= Very Poor 性能等级的限制，见 [avatar-performance-ranking-system#mobile-limits](https://creators.vrchat.com/avatars/avatar-performance-ranking-system#mobile-limits)）。**World 端没有此硬限制**，但移动端 World 仍应限制数量避免掉帧。
- **单组件 ≤256 transforms**: 含 Root 及所有子级；**Dynamic Bone 转换也会受影响**。
- **勿设 Humanoid 骨骼为 Root**: 不能把 Hip/Spine/Chest/Upper Chest/Neck/Head 或肢体骨骼设为 PhysBone 根。应在 Blender 中复制骨骼作为新根再重挂子级（工具如 Cat's Blender Plugin）。
- **Root 可旋转不可平移**: 与 Dynamic Bones 不同，PhysBone 链的 Root 骨骼**允许旋转**（不可平移）。
- **属性不可动画（Avatar）**: PhysBone 属性（Spring/Pull/Stiffness 等）初始化时设定，Avatar 端**不可动画**（强制动画后开关节不保证持续支持）。
- **与 Constraint 冲突**: **同一 GameObject 上不要同时挂 Constraint 和 PhysBone**（执行顺序问题）；把 Constraint 放到父级 GameObject 上。
- **最大边界 10×10×10 米**: 每个组件的包围盒被强制限为 10³ 米；骨骼可移出但玩家可能无法触/抓。包围盒只算**有碰撞且 Radius>0** 的骨骼。
- **最佳实践**: 单组件超 ~128 个受影响的 transform 时考虑拆分（多线程利于并行）；但组件也不要过多。

---

## 3. World 端的 PhysBone Udon 访问

> **FACT（核心补充）**: World 中的 PhysBone **不用 Animator 交互**，改用 Udon 脚本：

- 类型: `VRCPhysBone`，命名空间 `VRC.SDK3.Dynamics.PhysBone.Components`。
- 事件（挂在同一 GameObject 的 UdonBehaviour）：
  - `OnPhysBoneGrabbed(PhysBoneGrabbedInfo)` — 被抓握
  - `OnPhysBoneReleased(PhysBoneReleasedInfo)` — 被释放
  - `OnPhysBonePosed(PhysBonePosedInfo)` — 被锁定姿势
  - `OnPhysBoneUnPosed(PhysBoneUnPosedInfo)` — 姿势释放
- 属性: `IsGrabbed` / `IsPosed`；`Angle` / `Squish` / `Stretch`（可随时读）。
- 字段: 所有属性字段（`pull`、`stretch`… 及 `pullCurve`、`stretchCurve`…）可读写。
- **`ApplyConfigurationChanges()`**: 修改任何属性后**必须调用**，否则不生效。⚠️ **昂贵**——不要频繁改，尽量批量后再调用一次。
- **`ReleaseGrabs()` / `ReleasePoses()`**: 强制本客户端结束抓握/姿势；**仅本地**，若要对全员生效需走 [Network Event](../world/udon/networking/events.md)。

---

## 4. VRCPhysBoneCollider

定义影响 PhysBone 的碰撞体：

| 参数 | 说明 |
|------|------|
| **Root Transform** | 碰撞体放置的 Transform；空=本 GameObject |
| **Global Collision** | 启用后所有允许的 PhysBone 将把它视作 Colliders 列表成员（受 Allow Collision 规则约束）。**Avatar 最多 4 个全局碰撞体**；World 无数量限制 |
| **Shape Type** | `Sphere` / `Capsule` / `Plane`（**Global Collision 仅支持 Sphere/Capsule**） |
| **Radius** / **Height** | 尺寸（Height 仅 Capsule，含两端半球） |
| **Position** / **Rotation** | 偏移 |
| **Inside Bounds** | 启用=把骨骼**包含**在边界内而非排斥在外 |
| **Bones As Sphere** | 将碰撞半径视为以骨骼位置为球心的球，而非骨骼长度胶囊 |

**Standard Colliders（Avatar）**: Avatar Descriptor 的 "Colliders" 节定义了标准碰撞体（Head/Torso/Hands/Feet/Fingers），自动设置但可微调，**不影响性能评级**；手指/手碰撞体还作为全局 PhysBone 碰撞体影响他人 PhysBone 与世界物体。

**World 端 Udon 访问**: 类型 `VRCPhysBoneCollider`（同命名空间），字段 `shapeType` / `radius` / `height` / `position` / `rotation` 可读写；改动后也需 `ApplyConfigurationChanges()`（同样昂贵，批量）。

---

## 5. VRCPhysBoneRoot（仅 World）

定义一组 PhysBone 的运动根，并设置其更新时机：

| 参数 | 说明 |
|------|------|
| **Timing** | `Automatic`（默认，自动在 Fixed/Real 间切换以最小化抖动）/ `Fixed Time`（按 FixedUpdate 步长）/ `Real Time`（按 Update） |

无运动根时 PhysBone 自动决定更新时机。

---

## 6. Dynamic Bone 转换

- **Avatar**: 客户端**总是自动**把 Dynamic Bones 转换为 PhysBones（默认 Advanced 模式 + Multi-Child=Ignore）。转换非完美复制，目标只是"大多数设置可用不破坏"。**Dynamic Bones 在 Avatar 上下文已被完全移除**。
- **World**: Dynamic Bones 组件当前**仍可用**但**不会自动转换**；官方推荐使用 PhysBones（性能更好、可抓握交互）。
- **手动转换（SDK）**: `VRChat SDK/Utilities/Convert DynamicBones to PhysBones`（先在 Build Control Panel 选择 Avatar）。⚠️ 会删除原 Dynamic Bone 组件**不可轻易撤销**——先备份。
- **不可迁移项**: Dynamic Bone 的 `Force` / Gravity 的 **X/Z 方向值被忽略**（PhysBones 无对应）。

---

## 相关

- [[vrc-constraints.md|VRChat Constraints]] — 约束系统（勿与 PhysBone 同 GameObject）
- [[performance-rank.md|Avatar 性能评级]] — Quest 硬限制上下文
- [[../../concepts/rules/quest-constraints.md|Quest 约束规则]]
- [[../api/dynamics.md|World 端 Dynamics API（Contact/PhysBone 事件）]]
- [[../../entities/world/scene-components/index.md|Scene Components 总览]]
- [[ma2bt.md|MA/Mesh 压缩工具]] — 骨骼合并参考
