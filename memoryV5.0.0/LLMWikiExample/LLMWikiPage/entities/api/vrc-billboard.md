---
title: "VRCBillboard — 原生公告板（Billboard）API 类"
category: api
knowledge_level: applied
status: active
source: "https://ask.vrchat.com/t/developer-update-13-august-2026/48800 (VRChat 官方开发者更新 2026-08-13)"
source_type: official
version: 1.0
last_review: 2026-08-19
confidence: medium
tags:
  - api
  - udonsharp
  - performance
  - world
  - optimization
aliases:
  - "VRCBillboard"
  - Billboard
  - 公告板
  - 面向玩家
related:
  - index.md
  - player-api.md
  - ../world/scene-components/index.md
  - ../world/udon/players/player-positions.md
  - ../avatar/contact.md
type: entity
created: 2026-08-19
sources: "https://ask.vrchat.com/t/developer-update-13-august-2026/48800"
updated: 2026-08-19
---
# VRCBillboard — 原生公告板（Billboard）API 类

> ⚠️ **[PLANNED] 未正式发布** —— 本页基于 VRChat 官方开发者更新（2026-08-13）的 SDK 新特性预告，官方明确标注"正式发布前一切均可能变动（subject to change until release）"。
> **签名、行为、可用 SDK 版本均为预告，入库时未在正式 SDK 中确认。** 发布后须按 SCHEMA §8 复核更新。

---

## 概述

**告示板（Billboarding）** 指让某个对象始终面向本地玩家的常见需求（如面板、名字牌、UI 板、图片）。

传统实现需要在 Udon 中用**每帧更新循环（update loop）**旋转 Transform 使其面向玩家，开销较大。VRChat 正在为 World/场景对象添加**原生支持**，让告示板更易实现且更高效。

> **FACT**（官方 2026-08-13 公告）：官方动机即消除 Udon 中昂贵的 update loop —— 与骨骼挂载 API（[[../world/udon/players/player-positions.md|AttachTransformToBone]]）同期宣布，同属"原生替代每帧 Transform 搬运"的性能优化方向。

---

## API 签名（预告）

将提供三个新方法（静态调用，需通过 `VRCBillboard` 类访问）：

| 方法 | 行为 | 说明 |
|------|------|------|
| `VRCBillboard.Register(Transform target)` | **直立（upright）**，仅跟踪**偏航角（yaw，左右）** | 简单告示板场景；对象保持垂直不倾斜 |
| `VRCBillboard.Register(Transform target, bool allowTilt)` | 在 yaw 基础上**允许俯仰（pitch，上下）** | `allowTilt = true` 时对象可随视角倾斜（如抬头/低头看板） |
| `VRCBillboard.Unregister(Transform target)` | **取消注册** | 注册对象销毁时**自动清理**，无需手动调用；也可手动触发以提前解绑 |

> ⚠️ [PLANNED]：以上为官方公告原文签名；正式发布前的"一切均可能变动"声明同样适用于此签名。

---

## U# 用法示例（预告）

```csharp
using UdonSharp;
using UnityEngine;
using VRC.SDKBase;
using VRC.Udon;

public class BillboardPanel : UdonSharpBehaviour
{
    void Start()
    {
        // 直立，仅跟偏航（左右）
        VRCBillboard.Register(transform);
    }

    // 若需手动提前解绑（正常情况下销毁时会自动清理）
    // void OnDisable()
    // {
    //     VRCBillboard.Unregister(transform);
    // }
}
```

带俯仰的版本：

```csharp
// 允许上下俯仰变化
VRCBillboard.Register(transform, true);
```

> 注册为一次性调用，无需 Update 循环 —— 这正是本能力的性能优势所在（对比旧式每帧 `transform.LookAt(localPlayer)`）。

---

## 与 Unity 原生 `BillboardRenderer` 的区别

> ⚠️ 避免混淆：`BillboardRenderer`（Unity 原生组件，已并入 scene-components 索引的 [unity-rendering-aux.md](../world/scene-components/unity-rendering-aux.md) 合并页）是**面向摄像机的十字片渲染**组件，属于渲染管线；本页 `VRCBillboard` 是 **VRChat SDK 层面向本地玩家的 Transform 朝向** API 类，二者机制与用途不同。

---

## 性能动机（对比旧式做法）

| 方案 | 开销 | 说明 |
|------|------|------|
| 旧式：Udon Update 循环 + `transform.LookAt` | ⚠️ 每帧 Udon 指令成本高 | Udon VM 中每帧矩阵/四元数运算昂贵（官方强调） |
| 原生：`VRCBillboard.Register()` | ✅ 一次性注册，运行时原生处理 | 无需每帧脚本执行，替换 update loop |

> **FACT**（官方 2026-08-13 公告）：官方动机即"用 transforms 实现需要 Udon 更新循环，开销很大"，原生支持使其"更易用且更高效"。

---

## 适用场景

- 始终面向玩家的信息面板 / 名字牌 / 提示 UI
- 图片、视频面片（配合 VideoPlayer）朝向本地玩家
- 与 [[player-api.md|Player API]] 本地玩家相关逻辑配合（依赖本地玩家位置）

> 📌 该能力针对 **World/场景对象**（官方措辞为 world creator 视角）；Avatar 端公告板需求与 [[../avatar/contact.md|Contact]] 等 Avatar 机制分属不同域。

---

## 跨页引用

- **Scene Components（World 组件索引）**：[[../world/scene-components/index.md]]
- **骨骼挂载 API（同期预告）**：[[../world/udon/players/player-positions.md]]
- **VRC Contact / 接近度参数（同期预告）**：[[../avatar/contact.md]]
- **Player API**：[[player-api.md]]
- **API 总入口**：[[index.md]]

---

## 引用

- 官方公告：https://ask.vrchat.com/t/developer-update-13-august-2026/48800
- 关联 API：[[player-api.md]]（VRCPlayerApi / Networking.LocalPlayer）
- 同期 SDK 预告：骨骼挂载（[[../world/udon/players/player-positions.md]]）与接触接近度参数（[[../avatar/contact.md]]）
