---
title: "Interact / Pickup Outline Renderers"
category: world
subcategory: scene-components
knowledge_level: applied
status: active
source: https://creators.vrchat.com/releases/release-3-10-5/ | https://creators.vrchat.com/releases/release-3-10-6/
source_type: official
version: 1.0
sdk_version: "3.10.5 / 3.10.6"
last_review: 2026-11-10
confidence: Medium
tags:
  - world
  - interact
  - pickup
  - rendering
  - ux
aliases:
  - "Outline Renderers"
  - "Interact Outline Renderers"
  - "Pickup Outline Renderers"
  - interact-outline-renderers.md
related:
  - ../../api/pickups.md
  - vrc-scenedescriptor.md
  - ../udon/input-events.md
type: entity
created: 2026-11-10
sources: https://creators.vrchat.com/releases/release-3-10-5/ | https://creators.vrchat.com/releases/release-3-10-6/
updated: 2026-11-10
---
# Interact / Pickup Outline Renderers

> 世界侧「悬停描边」配置项：决定**玩家指向物体时，用哪些 Renderer 画描边**。
> 加进时间线: **3.10.5**（`VRCPickup` 先有）→ **3.10.6**（扩展到实现 `Interact()` 的 UdonBehaviour）
> 来源: [Release 3.10.5](https://creators.vrchat.com/releases/release-3-10-5/) · [Release 3.10.6](https://creators.vrchat.com/releases/release-3-10-6/)

---

## 是什么

玩家把手/准星指向可交互物体时，VRChat 会给它画一圈**悬停描边**。描边用哪些 Renderer 来算，过去是「自动取物体身上的渲染器」，现在可以由作者**显式指定**：

| 版本 | 变化 | 官方原文要点 |
|---|---|---|
| **3.10.5** | 给 **`VRCPickup`** 增加 `Outline Renderers` 属性 | `Added an "Outline Renderers" property to VRCPickup, which can be used to manually specify which parts of the pickup are highlighted when you hover over it in VRChat.` |
| **3.10.6** | 扩展到**实现 `Interact()` 的 UdonBehaviour** | `Added an Outline Renderers property to Udon behaviors that implement Interact(). This lets you select which child renderers should be used to create an outline while the object is being hovered over, similar to the matching property for VRCPickup.` |

> 该字段出现在**组件/UdonBehaviour 的 Inspector** 上（UdonBehaviour 的 Interact 相关设置区）。**没有 Udon 运行时 accessor**——官方 release note 只描述属性，未提供脚本读写入口；本轮按「仅 Inspector 配置」处理（未在 Udon API 文档中找到对应方法）。

---

## 为什么需要它

自动描边在两种结构上会出问题：

| 结构 | 默认行为的毛病 | 显式指定后 |
|---|---|---|
| **物体带很多子 Renderer**（角色模型、装饰件） | 整个物体所有 Renderer 都被描边，视觉噪音大、还可能描到不该描的部件 | 只列出「该被描边」的那几个（如主体 mesh） |
| **物体带 SkinnedMeshRenderer**（人物/宠物/NPC） | 描边成本高、观感差 | 3.10.6 起空列表**不再**默认描 SMR（见下） |
| **空列表 + 复杂层级** | 老表现可能描出一堆不该描的东西 | 明确列表 = 行为可预测 |

---

## 空列表语义（重要，且分客户端版本）

> **FACT**（3.10.6 原文）: `Pickups and interactables in worlds will no longer highlight skinned mesh renderers by default if the Outline Renderers list is empty, due to pre-existing content that relies on this. (Starting in client version 2026.4.1)`

| 客户端版本 | 列表为空时的默认行为 |
|---|---|
| **< 2026.4.1** | 仍会把 SkinnedMeshRenderer 也纳入默认描边 |
| **≥ 2026.4.1** | **不再**默认描边 SkinnedMeshRenderer（为了兼容既有内容） |

**这意味着**：同一个世界在不同客户端版本下，悬停描边的**观感会不一样**。
- 想稳定表现 → **显式填写 Outline Renderers**，别依赖空列表的默认行为。
- 如果你的世界依赖「空列表也会描 SMR」的旧观感，升级客户端后会变，要自己补列表。

---

## 配置步骤

1. 选中 **VRCPickup**（3.10.5+）或**实现了 `Interact()` 的 UdonBehaviour**（3.10.6+）。
2. 在 Inspector 找到 `Outline Renderers` 列表。
3. 把需要描边的 Renderer 拖进去（**只有列出的会被描边**）。
4. 进 ClientSim / 客户端实机看悬停效果（Editor 里不一定能完整复现描边时机）。

### 怎么选 Renderer

| 目标 | 列表填什么 |
|---|---|
| 单 mesh 物件 | 就那一个 MeshRenderer |
| 复合物件（灯 + 底座 + 贴图） | 只填「玩家该看到被指到」的那部分（常见是主体） |
| 人物/宠物类（带 SMR） | 要么显式列出想要的 SMR，要么留空（2026.4.1+ 不描 SMR） |
| 纯逻辑触发器 | 留空或填最简 Renderer，避免大范围描边 |

---

## 与 Udon 的关系

| 项 | 说明 |
|---|---|
| 触发条件 | 玩家悬停 + 该物体可交互（Pickup 或实现了 `Interact()`） |
| 读写 | **无 Udon accessor**（本轮核实：无脚本入口）；纯 Inspector 配置 |
| 与 `Interact()` 事件的关系 | `Interact()` 是**点击回调**；描边是**悬停表现**，两者独立 |
| 与 `VRCPickup` | 3.10.5 起 pickup 先支持；3.10.6 起 interactable 也支持 |

> 若真需要「运行时改变描边目标」，只能在设计上绕：换物体 / 换组件 / 用两套副本切换启用状态，而不是改这个列表。

---

## 常见坑

1. **以为留空 = 不描边** → 空列表在旧客户端会回退默认行为（含 SMR）。
2. **列表里塞了整个层级** → 描边把装饰件也描上，观感脏；只填主体。
3. **在旧 SDK 里找不到字段** → 3.10.5（pickup）/ 3.10.6（interactable）才有，需升级 SDK 包。
4. **跨客户端观感不一致** → 2026.4.1 起空列表的 SMR 行为变了，别把它当稳定表现。

---

## 相关文档

- [[entities/api/pickups]] — `VRCPickup` 完整 API（描边属性属表现层，API 页另有职责）
- [[entities/world/udon/input-events]] — `Interact()` 事件与 `InputUse` 的区别
- [[entities/world/scene-components/vrc-scenedescriptor]] — 场景级交互相关配置
- [Release 3.10.5](https://creators.vrchat.com/releases/release-3-10-5/) · [Release 3.10.6](https://creators.vrchat.com/releases/release-3-10-6/)