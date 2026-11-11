---
title: "VRC Placement Zone"
category: world
subcategory: scene-components
knowledge_level: applied
status: active
source: https://creators.vrchat.com/worlds/components/vrc_placementzone/
source_type: official
version: 1.0
sdk_version: "3.10.6"
last_review: 2026-11-10
confidence: High
tags:
  - world
  - component
  - portal
  - networking
  - content-control
aliases:
  - "VRC Placement Zone"
  - "VRCPlacementZone"
  - vrc-placementzone.md
related:
  - vrc-portalmarker.md
  - ../creator-economy.md
  - ../items.md
type: entity
created: 2026-11-10
sources: https://creators.vrchat.com/worlds/components/vrc_placementzone/
updated: 2026-11-10
---
# VRC Placement Zone

> SDK **3.10.6** 新增的世界组件：用盒体区域限制**用户放置类内容**出现的位置。
> 来源: [creators.vrchat.com · VRC Placement Zone](https://creators.vrchat.com/worlds/components/vrc_placementzone/)（Last updated 2026-10-08）

---

## 概述

`VRCPlacementZone` 是一个**有宽/高/深的盒体**，用来控制以下内容的放置：

| 内容类型 | 典型来源 |
|---|---|
| **Portals**（传送门） | 用户放置的 portal（**不是**作者摆的 `VRC_PortalMarker`） |
| **Stickers**（贴纸） | 用户贴在任何表面的 sticker |
| **Sharing Pedestals**（分享基座） | 用户放置的分享/展示基座 |
| **Attached Items**（附着物） | 用户贴到表面的物件 |

> **FACT**（官方原文）: `Allows you to control the placement of user portals, stickers, sharing pedestals, and/or world-attached items in specified areas of your world.`
> ⚠️ 官方原文是「**user** portals」——该区块约束的是**用户放置**的内容；世界作者自己摆在场景里的 `VRC_PortalMarker` 属于静态场景物件，不在这个「放置控制」语境里。
> ⚠️ 官方原文：不附着在表面的物件（例如 pickup）**不受** Attached Items 规则影响。

---

## 组件字段

| 字段 | 说明 |
|---|---|
| **Center** | 盒体中心（GameObject 局部空间） |
| **Rotation** | 盒体旋转（局部空间） |
| **Size** | 盒体宽/高/深（局部空间） |
| **Portals** | 本区对 portal 放置的影响方式 |
| **Stickers** | 本区对 sticker 放置的影响方式 |
| **Sharing Pedestals** | 本区对分享基座的影响方式 |
| **Attached Items** | 本区对表面对附着物的影响方式 |

启用 gizmo 时，SDK 会用**黄色盒线框**画出区域范围——摆放时一定开着看。

---

## 三种权限语义（每类内容各选一个）

| 选项 | 语义 | 说明 |
|---|---|---|
| **Unaffected** | 不影响 | 该类型内容在本区内外的规则不变 |
| **Block Here** | 区内禁止 | 本区内不能放；区外照常（除非被别的 Only Allow Here 约束） |
| **Only Allow Here** | 只允许区内 | 本区内容许；区外被禁——**除非**另一个 `Only Allow Here` 区允许 |

### 多区组合规则（官方明文）

| 组合 | 结果 |
|---|---|
| 两个区都 **Block Here** | 区外仍可放（两个区内的区域被禁） |
| 两个区都 **Only Allow Here** | 只能放在任一区内（重叠处可放），区外不行 |
| 一个 **Only Allow Here** + 一个 **Block Here**，且两区重叠 | 重叠区域**仍可放**；完全落在允许区内部的禁止区**不生效** |

---

## 世界端行为要点

### 1. 只管「放置」，不删「已有」

> **FACT**（官方原文）: `placement zones don't remove content; they only prevent placement.`
- 开启一个禁止贴纸的区**不会**清掉已在区内的贴纸。
- 想让旧内容消失，得自己写清理逻辑（或让用户手动移除）。

### 2. 动态改区必须做网络同步

> **FACT**（官方原文）: `When moving zones around or changing placement rules dynamically, make sure the changes are synced over the network, since mismatched zones between users can result in unexpected behavior.`
- 静态摆放的区（场景内固定）不需要额外同步。
- **移动 / 改规则**的区：每个客户端必须看到同一份状态，否则 A 玩家能在 B 玩家眼里「禁放」的区域放东西，表现会分裂。
- 实现方式：把 Center/Size/Rotation 或规则字段放进 `[UdonSynced]`，由 owner 写入后 `RequestSerialization()`，并处理 late joiner 的初始状态。

### 3. 摆放建议：把区**略微埋进**不想要内容的表面

> **FACT**（官方原文）: `For the best results, you should slightly embed the zone inside the surfaces where you don't want placed objects to appear.`
- 尤其对 **stickers**：贴纸能贴在任何表面，不只是地面；区若只贴着地平面浮在上面，墙面上仍能贴。
- 正确做法：让盒体边缘稍微**穿进**墙面/地面，把「表面自身」也包在禁止体积里。

### 4. 区域是局部空间盒体

- Center/Rotation/Size 都在**组件所在 GameObject 的局部空间**；父节点缩放/旋转会一起改变实际覆盖体积。
- 做「房间级禁止」时，把组件挂在房间根节点并核对世界空间尺寸，别在缩放父级下凭感觉调 Size。

---

## 典型用法

| 目标 | 配置 |
|---|---|
| 演播厅中央不许乱放 portal | 大盒体覆盖舞台区 → Portals = **Block Here** |
| 只允许在指定展示墙贴 sticker | 薄盒体贴住展示墙（略埋入表面） → Stickers = **Only Allow Here** |
| 全图禁贴纸，但留一块留言板 | 覆盖全图的大盒体 Stickers = **Block Here**，留言板处再放一个小盒体 Stickers = **Only Allow Here**（重叠处按官方组合规则仍可放） |
| 分享基座只在入口处 | 入口盒体 Sharing Pedestals = **Only Allow Here** |

---

## 兼容性与限制

| 项 | 说明 |
|---|---|
| SDK 版本 | **3.10.6+**（更早 SDK 无该组件） |
| 内容范围 | 只覆盖 portal / sticker / sharing pedestal / attached item 四类；**pickup 等不附着物件不受影响** |
| 不删既有内容 | 见上（官方明文） |
| 客户端版本联动 | 放置类内容本身依赖较新客户端；团队审计记录：官方 Dev Update（2026-10-22）补充说明仅**用户放置**的 portal 适用本区 |
| 实测状态 | **官方 placement 行为未在本库实测**（无客户端/上传环境）；以上均引自官方组件页 |

---

## 常见坑

1. **以为能清场** → 它只阻止新放置，旧内容还在。
2. **动态改区不同步** → 各客户端看到不同区域，放置结果随机化。
3. **区浮在表面外侧** → 贴纸仍能贴到墙上/地上；要把表面本身也包含进体积。
4. **`Only Allow Here` 用了两个不重叠的区** → 两个区之间（区外）全都不能放，容易误伤。
5. **在缩放父级下调 Size** → 实际体积与 Inspector 数值不一致。

---

## 相关文档

- [[entities/world/scene-components/vrc-portalmarker]] — 作者摆放的 portal 组件（与本区的「用户放置」区分）
- [[entities/world/creator-economy]] — 分享基座 / 创作者经济相关内容
- [[entities/world/items]] — pickup 与交互物件（不受 Attached Items 规则影响）
- [官方文档 · VRC Placement Zone](https://creators.vrchat.com/worlds/components/vrc_placementzone/)
- [Release 3.10.6](https://creators.vrchat.com/releases/release-3-10-6/)