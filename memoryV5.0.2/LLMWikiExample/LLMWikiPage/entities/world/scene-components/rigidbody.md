---
title: "Rigidbody — 世界物理组件"
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
  - sync
aliases:
  - Rigidbody
  - 刚体
related:
  - ../whitelisted-world-components.md
  - boxcollider.md
  - ../../api/pickups.md
  - vrc-objectsync.md
  - ../../api/dynamics.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-Rigidbody.html | https://creators.vrchat.com/worlds/whitelisted-world-components/ | https://creators.vrchat.com/worlds/components/vrc_objectsync | https://creators.vrchat.com/worlds/components/vrc_pickup"
updated: 2026-08-17
---
# Rigidbody — 世界物理组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无本组件独立帧时间数据，详见性能小节）

---

## 核心功能（官方，[FACT]）

Rigidbody 让 GameObject 的运动与位置受物理引擎（Unity 内置 3D 物理 = Nvidia PhysX）控制。相较直接修改 Transform，可以用模拟的力与扭矩移动对象，由物理引擎结算；仍可用脚本 API 施加力做物理拟真控制。

- 带 Rigidbody 的对象即使不加代码也会被重力下拉，并在具备正确 Collider 时响应与其他对象的碰撞。
- [COMMUNITY] **1 Unity unit ≈ 1 米**是物理默认尺度共识（官方类-Rigidbody 与 RigidbodiesOverview 页未显式列出该换算，属社区广泛认知）：比例失衡（如把本应 1~4 米的模型做成 100 单元）会导致"异常缓慢地下落"——新手常见"慢动作"现象的根源。
- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-Rigidbody.html（含 `RigidbodiesOverview`）

## 关键属性/行为

| 属性 | 说明（[FACT]） |
|---|---|
| **Mass** | 质量（千克），默认 1。质量不影响下落快慢；模拟空气阻力用 Drag。 |
| **Drag / Angular Drag** | 线性/旋转速度衰减率。Angular Drag 默认 0.05。 |
| **Use Gravity** | 是否受重力影响（默认沿 -Y 方向），默认开启。 |
| **Is Kinematic** | 开启后物理系统不再施力驱动，只能通过 Transform 移动。默认关闭。 |
| **Interpolate / Extrapolate** | 平滑物理更新之间的姿态表现，减少抖动。默认 None。 |
| **Collision Detection** | Discrete（默认）/ Continuous / Continuous Dynamic / Continuous Speculative；CCD 可防高速穿透但更耗性能，**仅支持 Sphere/Capsule/Box 碰撞体**。 |
| **Constraints** | 逐轴冻结位移/旋转。 |
| **Layer Overrides** | 覆盖项目级 Layer 碰撞矩阵。 |

## VRChat 特定限制/注意

**官方地位**：Rigidbody 是 World 白名单组件（Unity Components → Physics/Colliders 段）；VRChat 无 Rigidbody 专属页，地位仅见官方白名单清单 [[../whitelisted-world-components.md|whitelisted-world-components]]。Quest 端物理 **Avatar 禁用 / World 允许**（[FACT] 官方 Quest 限制页 https://creators.vrchat.com/platforms/android/quest-content-limitations#components；勿误报为 World 禁用）。

**[FACT] 与 VRC Object Sync 的协同**：
- VRC Object Sync 同步 position/rotation/kinematic 状态/gravity 状态，提供 `SetKinematic(bool)` / `SetGravity(bool)` / `TeleportTo` / `Respawn`。
- `Force Kinematic On Remote`：非拥有者的远程端刚体被强制 kinematic。
- **不要**用 Udon 直接写 `Rigidbody.isKinematic/useGravity` 去和同步打架——社区与官方反馈证实同步下直改不可靠。详见 [[vrc-objectsync.md|VRC Object Sync]]。

**[INFERENCE] Udon 直改 isKinematic 的坑（社区反馈）**：
- UdonBehaviour 开启 Synchronize Position（或对象带 VRC Object Sync）时直接设置 `Rigidbody.isKinematic = true` 会无效或下一帧被重置（官方反馈已部分修复，社区仍报告不可靠）。
- 结论：同步对象改 kinematic/gravity 应走 `SetKinematic`/`SetGravity`。

**[FACT] VRC Pickup 依赖 Rigidbody**：
- VRC Pickup "Requires: Rigidbody、Collider"；VRCPickup 会为无 Rigidbody 的物体自动添加，并移交所有权给抓取者。见 [[../../api/pickups.md|Pickups API]]。

**[INFERENCE] Pickup + isKinematic 交互注意**：带 VRC_Pickup 的对象被持有时通常需非 kinematic；上述同步位重置问题在 Pickup 场景表现为"拾起后 drop 仍静止不动"。

## 性能数据（unityvrchat 或 [UNKNOWN]）

`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）` **无 Rigidbody 独立帧时间数据**（该书覆盖 Animator/Constraints/Audio/Contacts/Cloth/PhysBones）→ **无该组件独立性能数据**。

相关邻近事实（[FACT]，上下文非本体）：
- Contact 每房间上限 4096；关闭状态每 1000 个 Senders/Receivers 约 0.5ms，开启的 Receivers 每 1000 个约 0.75ms。
- Cloth 的 Collider 使帧时间约翻倍，每 10 个 Collider 每个额外约 7%——间接提示碰撞体数量抬高物理成本。

## 社区佐证

- VRChat 官方反馈：`Rigidbody.isKinematic not toggleable if UdonBehaviour.synchronizePosition = true`（https://feedback.vrchat.com/udon/p/rigidbodyiskinematic-not-toggleable-if-udonbehaviorsynchronizeposition-true）——厂方承认该交互问题并部分修复。
- VRChat Ask Forum：`Is keeping Rigidbody's awake possible?`（https://ask.vrchat.com/t/is-keeping-rigidbodys-awake-possible/48042）——`sleepThreshold`/`NeverSleep` 未暴露给 Udon，精细控制刚体唤醒受限。

## 相关页面

[[../whitelisted-world-components.md|Allowlisted World Components]] · [[boxcollider.md|BoxCollider]] · [[../../api/pickups.md|Pickups]] · [[vrc-objectsync.md|VRC Object Sync]]

## 源清单

- Unity Manual: Rigidbody (2022.3) — https://docs.unity3d.com/2022.3/Documentation/Manual/class-Rigidbody.html
- Unity Scripting API: Rigidbody — https://docs.unity3d.com/2022.3/Documentation/ScriptReference/Rigidbody.html
- Unity Manual: RigidbodiesOverview — https://docs.unity3d.com/2022.3/Documentation/Manual/RigidbodiesOverview.html
- VRChat: Allowlisted World Components — https://creators.vrchat.com/worlds/whitelisted-world-components/
- VRChat: VRC Object Sync — https://creators.vrchat.com/worlds/components/vrc_objectsync
- VRChat: VRC Pickup — https://creators.vrchat.com/worlds/components/vrc_pickup
- 本地性能源（无本组件数据）：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`