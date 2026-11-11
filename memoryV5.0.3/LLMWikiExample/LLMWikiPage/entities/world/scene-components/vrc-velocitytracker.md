---
title: "VRC Velocity Tracker"
category: world
subcategory: scene-components
knowledge_level: applied
status: active
source: https://creators.vrchat.com/worlds/components/vrc_velocitytracker/
source_type: official
version: 1.0
sdk_version: "3.10.6"
last_review: 2026-11-10
confidence: High
tags:
  - world
  - component
  - physics
  - udonsharp
aliases:
  - "VRC Velocity Tracker"
  - "VRCVelocityTracker"
  - vrc-velocitytracker.md
related:
  - vrc-objectsync.md
  - ../udon/networking/performance.md
  - ../udon/udonsharp/performance-tips.md
type: entity
created: 2026-11-10
sources: https://creators.vrchat.com/worlds/components/vrc_velocitytracker/
updated: 2026-11-10
---
# VRC Velocity Tracker

> SDK **3.10.6** 新增的世界组件：不开自己的更新循环，也能从 Udon 读到速度。
> 来源: [creators.vrchat.com · VRC Velocity Tracker](https://creators.vrchat.com/worlds/components/vrc_velocitytracker/)（Last updated 2026-09-29）
> **本页范围**: 组件行为与四个读取方法的语义。Udon 侧方法表归 API 域维护，本页不另立 API 表。

---

## 概述

`VRCVelocityTracker` 提供一条**低成本**的速度读取通道：不给物体挂自己的 Udon 更新循环，由 SDK 侧跟踪该物体的速度。

> **FACT**（官方文档）: `VRCVelocityTracker provides an efficient way to read velocity from Udon programs without running its own update loop.`

**为什么重要**：以往要在 Udon 里拿速度，常见做法是 `Update` 里缓存上一帧位置自己除 deltaTime——每物体一个更新循环 + 每帧几次 Udon 调用；物体一多（抓取物、投射物、载具部件）就会明显吃性能。该组件把这部分移出 Udon。

---

## 四个读取方法（语义与单位）

| 方法 | 单位 | 采样基准 | 说明 |
|---|---|---|---|
| `GetLinearVelocity()` | **米/秒** | **上一渲染帧** | 线性速度 |
| `GetAngularVelocity()` | **弧度/秒** | **上一渲染帧** | 角速度 |
| `GetFixedLinearVelocity()` | **米/秒** | **上一物理步** | 线性速度（FixedUpdate 域） |
| `GetFixedAngularVelocity()` | **弧度/秒** | **上一物理步** | 角速度（FixedUpdate 域） |

> **FACT**（官方 Notes）: 组件被**禁用**时，以上方法**全部返回 0**。不会报错、不会抛异常——「速度突然变 0」先查组件是否被禁用/GameObject 是否关闭。

### 该读哪一组？

| 场景 | 用哪组 |
|---|---|
| 表现层（拖尾、风效、UI 速度表） | `GetLinearVelocity()` / `GetAngularVelocity()`（渲染帧，和画面同步） |
| 物理判定（碰撞伤害、投掷判定、力反馈） | `GetFixedLinearVelocity()` / `GetFixedAngularVelocity()`（物理步，和 `FixedUpdate` 同步） |
| 混用 | 会得到「物理步 ≠ 渲染帧」的抖动，通常不是你要的 |

---

## 典型用法

```csharp
using UdonSharp;
using UnityEngine;
using VRC.SDK3.Components;

public class ThrowableSpeedLabel : UdonSharpBehaviour
{
    public VRCVelocityTracker Tracker;
    public TextMeshProUGUI Label;

    void Update()
    {
        Vector3 v = Tracker.GetLinearVelocity();   // m/s
        Label.text = v.magnitude.ToString("F1") + " m/s";
    }
}
```

**别做的事**：
- ❌ 在 `Update` 里自己算速度（`(pos - lastPos) / Time.deltaTime`）替代该组件——那是本组件存在的意义
- ❌ 用 `GetFixedLinearVelocity()` 驱动视觉效果——物理步频率与渲染帧不同步，会抖
- ❌ 在禁止物理的物体上指望它给出物理步数据（物体没有 Rigidbody 时物理步速度通常为 0，属预期）

---

## 与既有速度来源的对比

| 来源 | 单位/基准 | 成本 | 备注 |
|---|---|---|---|
| **VRCVelocityTracker** | m/s、rad/s（渲染帧 / 物理步各一组） | 低（SDK 侧跟踪，无 Udon 更新循环） | SDK 3.10.6+ |
| 自己缓存上一帧位置 | 任意 | 高（每物体一个 Update + 多次 Udon 调用） | 老世界常见写法，可替换 |
| `Rigidbody.velocity`（Udon 可读） | m/s | 中 | 仅对挂了 Rigidbody 的物体有意义，且是物理步语义 |
| `VRCObjectSync` 的位置差 | 任意 | 中 | 那是网络同步数据，不等于真实速度 |

---

## 兼容性与限制

| 项 | 说明 |
|---|---|
| SDK 版本 | **3.10.6+**；在更早 SDK 里该类型不存在，UdonSharp 会编译失败 |
| 平台 | PC / Quest 一致（无平台特化说明） |
| 禁用行为 | 返回 0（官方明文） |
| 网络同步 | 无；速度是本地采样，不做同步 |
| 写入 | 只读；组件本身不提供「设置速度」的能力 |

---

## 常见坑

1. **「速度恒为 0」** → 先查组件/GameObject 是否被禁用（禁用即返回 0）。
2. **物体被传送/瞬移后速度尖刺** → 那是真实位移导致的；需要过滤就自己加阈值。
3. **忘了 `using VRC.SDK3.Components;`** → 类型找不到，其实包已装。
4. **在非物理物体上找物理步速度** → 没有 Rigidbody 时物理步不推进，别拿它当唯一信号。

---

## 相关文档

- [[entities/world/scene-components/vrc-objectsync]] — 物理物体同步组件
- [[entities/world/udon/udonsharp/performance-tips]] — Udon 性能建议（本组件正是「少写 Update」的典型）
- [官方文档 · VRC Velocity Tracker](https://creators.vrchat.com/worlds/components/vrc_velocitytracker/)
- [Release 3.10.6](https://creators.vrchat.com/releases/release-3-10-6/)