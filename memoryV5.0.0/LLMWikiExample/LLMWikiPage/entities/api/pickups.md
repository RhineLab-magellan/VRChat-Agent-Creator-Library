---
title: "API: Pickups (VRCObjectPickup)"
category: api
knowledge_level: core
status: active
source: "VRChat 官方文档 + 项目实测 + VRChat 2026.1.2 / 2026.2.3 / 2026.3.1 Open Beta Release Notes"
source_type: official
version: 1.1
last_review: 2026-06-30
confidence: High
tags:
  - api
  - sync
  - pickup
  - ownership
aliases:
  - Pickups
  - "拾取 API"
  - VRCPickup
  - 拾取系统
related:
  - events-reference.md
  - networking.md
  - official-doc-clarifications.md
  - player-api.md
  - animator.md
type: entity
created: 2026-06-30
sources: "VRChat 官方文档 + 项目实测 + VRChat 2026.1.2 / 2026.2.3 / 2026.3.1 Open Beta Release Notes"
updated: 2026-06-30
---
# API: Pickups (VRCObjectPickup)


---

## VRCObjectPickup

VRCObjectPickup 是 VRChat 的内置拾取组件，挂载在 GameObject 上与 UdonBehaviour 配合。

### 硬性前件（官方必需）
- **Rigidbody** — 必须在同一 GameObject 上（或用父级）。位置/旋转同步依赖它。
- **Collider** — 用于玩家接近检测与抓取判定（禁用 Collider 的对象无法被拾取）。
- 来源: [官方 VRC Pickup](https://creators.vrchat.com/worlds/components/vrc_pickup)。

### Inspector 关键参数（官方）
| 参数 | 说明 |
|------|------|
| **Momentum Transfer Method** | 投掷时动量传递方式（影响扔出后的轨迹） |
| **Disallow Theft** | 勾选后，其他玩家不能从持有者手中抢走对象 |
| **Exact Gun / Exact Grip** | 精确到手指/手掌的抓握姿势 |
| **Allow Manipulation When Equipped** | 装备状态下手动操纵 |
| **Auto Hold** | 自动手持（1.0 旧版 / 1.1 新版；Yes/No/Sometimes/Auto） |
| **Use Text** | 显示在用户界面上的交互文本 |
| **Throw Velocity Boost Min Speed / Scale** | 投掷速度加成下限/比例 |
| **Pickupable** | 是否允许被拾取 |
| **Proximity** | 玩家接近即高亮/自动抓取（Raycast/Hover 规则，手部 0.4m 半径内） |

### OnPickup()
- **暴露**: ✅
- **说明**: 玩家拾取对象时调用。此时 ownership 自动转移给拾取玩家。

### OnDrop()
- **暴露**: ✅
- **说明**: 玩家放下对象时调用。ownership 保留在放下玩家。

### OnPickupUseDown()
- **暴露**: ✅
- **说明**: 手持时按下 Use 键（Trigger）。

### OnPickupUseUp()
- **暴露**: ✅
- **说明**: 手持时松开 Use 键。

## VRCObjectSync

### 自动同步
- 位置、旋转由 VRCObjectSync 自动处理
- 不需要手动同步 Transform

### Respawn
- **暴露**: ✅
- **说明**: 重置对象到初始位置（`VRCObjectSync.Respawn()` 官方公开）。

## 与 UdonBehaviour 配合

### 典型结构
```text
GameObject
  ├── VRCObjectPickup
  ├── VRCObjectSync
  └── UdonBehaviour (你的脚本)
```

### Sync Mode 建议
拾取对象使用 `BehaviourSyncMode.Continuous` 如果需要 synced variable 配合位置同步，或 `Manual` 仅用于状态同步。

### Ownership 转移
- Pickup 时 ownership 自动转移给拾取者
- Drop 时 ownership 保留
- 其他玩家 Pickup 时 ownership 自动转移

## 常见错误
- 拾取对象上使用 Manual Sync 但需要连续位置同步（位置由 VRCObjectSync 处理，独立于 UdonBehaviour SyncMode）
- OnDrop 中没有处理状态清理
- 多人争抢拾取时的竞态（通常由 VRChat 内置处理）

## 🔄 VRChat 修复历史 (2026.1.2+)

> **FACT** (2026.1.2):"Fixed an issue where Auto Hold could leave a pickup stuck in a 'grabbed' state for a remote user."

### 问题场景

| 场景 | 表现 | 修复状态 |
|------|------|----------|
| **AutoHold 启用** | 远端用户可能看到 Pickup **卡在"被抓住"状态** | ✅ 2026.1.2 修复 |
| **所有权转移异常** | Owner 已放下,但远端仍显示 grabbed | ✅ 2026.1.2 修复 |

### 创作者影响

| 影响项 | 说明 |
|--------|------|
| **无需代码改动** | 纯引擎层修复 |
| **AutoHold 行为恢复** | 2026.1.2+ 远端状态正确同步 |
| **旧版客户端** | 2026.1.1 及之前可能仍触发此问题 |

### 缓解措施(在旧版客户端仍有效)

```csharp
// 在 OnDrop 中显式重置 AutoHold 状态
public override void OnDrop()
{
    if (!Networking.IsOwner(gameObject)) return;
    // 显式重置内部状态以避免卡住
    // 注意:这只是缓解措施,2026.1.2+ 修复后不需要
}
```

### 关联文档

- [[entities/world/scene-components/vrc-objectsync]] - 物理同步与所有权
- [[entities/api/events-reference]] - OnPickup / OnDrop 事件签名

---

## 🔄 SkinnedMeshRenderer 高亮(2026.2.3 → 2026.3.1 Open Beta)

> **FACT** (2026.2.3): 曾加入对可变形 Pickups(SkinnedMeshRenderer)的精确高亮;**因影响部分内容在 2026.2.3p1 回滚**。
>
> **FACT** (2026.3.1 Open Beta Build 1878,2026-07-22):**重新引入** SkinnedMeshRenderer 高亮,并新增一个字段以指定需要 outline 的具体 mesh 列表。
> "Reintroduced precisely highlighting pickups that can change shape, in other words SkinnedMeshRenderer support for VRC_Pickup. We reverted this one in the last release due to it affecting some content. We've now added a field to specify which meshes exactly to outline."
>
> **🔴 关键约束**:支持自定义 mesh 列表的 **SDK 字段未随此补丁发布**——官方明确:"The SDK that will allow them to fix that is not shipping with this patch, however. Next release will have it!"(下一个版本才有 SDK 字段)
> **[Missing Information / 未确认]**:SDK 字段具体名称、API 路径待下一个版本公布。

### 影响与建议

| 影响项 | 说明 |
|--------|------|
| **受影响内容范围** | 官方评估"extraordinarily small",大多数 World/Avatar 不受影响 |
| **当前 Live(2026.2.3p3)** | 仍为回滚状态(无此高亮) |
| **Open Beta(2026.3.1)** | 高亮已重新启用,但**创作者尚无法通过 SDK 自定义 mesh 列表**(等下版本) |
| **建议** | 使用 SkinnedMeshRenderer Pickup 的创作者留意下个 SDK 版本的字段公告 |

---

## 相关页面

[[events-reference.md]] · [[networking.md]] · [[official-doc-clarifications.md]] · [[player-api.md]] · [[animator.md]]
