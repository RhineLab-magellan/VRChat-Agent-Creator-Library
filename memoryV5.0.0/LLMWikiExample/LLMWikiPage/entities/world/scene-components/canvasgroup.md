---
title: "CanvasGroup — 世界 UI 组件"
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
  - ui
  - rendering
aliases:
  - CanvasGroup
  - 画布组
related:
  - ../whitelisted-world-components.md
  - canvas.md
  - ../../api/ui.md
  - ../udon/ui-events.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-CanvasGroup.html | https://docs.unity3d.com/2022.2/Documentation/ScriptReference/CanvasGroup.html | https://creators.vrchat.com/worlds/whitelisted-world-components/"
updated: 2026-08-17
---
# CanvasGroup — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无本组件独立帧时间数据，详见性能小节）

---

## 核心功能（官方，[FACT]）

Canvas Group 用于**从一个地方批量控制一组 UI 元素**（作用于其所在及所有子元素）。

- **[FACT] Alpha**：整组不透明度（0–1），与各元素自身 alpha **相乘**。
- **[FACT] Interactable**：false 禁用组内交互。
- **[FACT] Block Raycasts**：false 时本组不拦截 Graphic Raycaster 的射线（`IsRaycastLocationValid` 返回 false）。
- **[FACT] Ignore Parent Groups**：是否忽略更上层 CanvasGroup 设置（用于覆盖父组）。
- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-CanvasGroup.html
- 源：https://docs.unity3d.com/2022.2/Documentation/ScriptReference/CanvasGroup.html

## 关键属性/行为

- 典型用途：整窗淡入淡出（Alpha）、整组置灰不可用（Interactable=false）、让某些 UI 不挡鼠标（Block Raycasts=false）。
- 局部/全局射线：Block Raycasts 只影响 Graphic Raycaster（UI 事件），**不影响 Physics.Raycast**。

## VRChat 特定限制/注意

**官方地位**：CanvasGroup 在 [[../whitelisted-world-components.md|Allowlisted World Components]]「Unity Components (77)」内，World 可用（与 Canvas、CanvasRenderer 同列于 Rendering 组）。**无 VRChat 官方专页**：无专属文档；由 Allowlist + VRC_UIShape 交互体系覆盖。为纯 Unity UI 组件。
- 源：https://creators.vrchat.com/worlds/whitelisted-world-components/

**[FACT] World UI 交互配套**：在 VRChat World 中 CanvasGroup 最常用于**整体显示/隐藏一组 UI**（如对话框 Alpha 淡入）、禁用一组按钮（interactable）、以及让遮罩层不挡点击（blocksRaycasts）。需 VRC_UIShape 才能交互（见 [[canvas.md|Canvas]]）。UI 事件体系见 [[../../api/ui.md|UI API]] 与 [[../udon/ui-events.md|Udon UI Events]]。

**[INFERENCE] 无 VRChat 特有极限记录**：作为标准 Unity UI 组件随 Canvas 工作。可在 Alert/Dialog 弹出层用 Alpha+blocksRaycasts 控制。

## 性能数据（unityvrchat 或 [UNKNOWN]）

`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）` **无 CanvasGroup 独立帧时间数据** → **`[UNKNOWN]`，无该组件独立性能数据**。

相关邻近事实（[FACT]，**非本体数据**）：CanvasGroup Alpha 变化会触发 UI 重建，本地 `ui.md`（见 [[../../api/ui.md|UI API]]）提示需节流 UI 更新频率；无官方帧时间基准。

## 社区佐证

- [COMMUNITY] Unity Discussions: CanvasGroup not blocking raycasts（https://discussions.unity.com/t/canvas-group-not-blocking-raycasts-even-when-option-is-enabled/715290）——常见坑：需确认是 GraphicRaycaster 场景且 Block Raycasts 语义排他。非 VRChat 专属。

## 相关页面

[[../whitelisted-world-components.md|Allowlisted World Components]] · [[canvas.md|Canvas]] · [[../../api/ui.md|UI API]] · [[../udon/ui-events.md|Udon UI Events]]

## 源清单

- Unity Manual Canvas Group: https://docs.unity3d.com/2022.3/Documentation/Manual/class-CanvasGroup.html
- Unity Scripting API CanvasGroup: https://docs.unity3d.com/2022.2/Documentation/ScriptReference/CanvasGroup.html
- VRChat Allowlist: https://creators.vrchat.com/worlds/whitelisted-world-components/
- 本地: [[../../api/ui.md|UI API]]