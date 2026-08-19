---
title: "Canvas — 世界 UI 组件"
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
  - performance
aliases:
  - Canvas
  - 画布
  - UI 画布
related:
  - ../../api/ui.md
  - ../whitelisted-world-components.md
  - audiosource.md
  - videoplayer.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-Canvas.html | https://docs.unity3d.com/2022.3/Documentation/Manual/script-CanvasScaler.html | https://docs.unity3d.com/2022.3/Documentation/ScriptReference/Canvas.html | https://creators.vrchat.com/worlds/components/vrc_uishape/ | https://creators.vrchat.com/worlds/whitelisted-world-components/ | https://docs.vrchat.com/docs/vrchat-202323"
updated: 2026-08-17
---
# Canvas — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（未包含 Canvas 数据；`entities/api/ui.md` 有节流 Inference，详见性能小节）

---

## 核心功能（官方，[FACT]）

Canvas 是 Unity UI 布局与渲染的抽象空间；**所有 UI 元素必须是带有 Canvas 组件的 GameObject 的子对象**。Render Mode 三选一：

- **Screen Space - Overlay**：直接渲染到屏幕（无需摄像机，始终在最上层）。
- **Screen Space - Camera**：渲染到指定摄像机前方平面。
- **World Space**：作为场景中的平面对象渲染（可用于 3D 世界内 UI）。

关联组件：`CanvasScaler`（控制像素密度/缩放，World Space 下控制 Dynamic Pixels Per Unit）、`GraphicRaycaster`（射线检测）。Scripting API 暴露 `renderMode`、`worldCamera`、`pixelPerfect`、`planeDistance`、`sortingOrder`、`referencePixelsPerUnit`（[FACT] https://docs.unity3d.com/2022.3/Documentation/Manual/class-Canvas.html / https://docs.unity3d.com/2022.3/Documentation/ScriptReference/Canvas.html）。

## 关键属性/行为

| 属性/行为 | 说明（[FACT]） |
|---|---|
| **嵌套 Canvas** | 子 Canvas 可用作优化（分批 mesh）。 |
| **Event Camera / Render Camera** | World Space 用 `Event Camera`，Screen Space-Camera 用 `Render Camera`；`Receives Events` 决定是否处理 UI 事件。 |
| **World Space 尺寸** | 由 RectTransform 决定，屏幕显示大小取决于观看角度/距离。 |

## VRChat 特定限制/注意

**[FACT] 白名单地位**：`Canvas`、`CanvasGroup`、`CanvasRenderer` 均在官方 Allowlisted World Components「Unity Components (77)」内，World 可用，见 [[../whitelisted-world-components.md|Allowlisted World Components]]。

**[FACT] VRC_UIShape 是交互关键**：世界内可交互 UI 必须满足以下 7 条必要条件（官方 [VRC Ui Shape](https://creators.vrchat.com/worlds/components/vrc_uishape/)）：
1. Canvas 组件上要有 `VRC_UIShape` 组件（同一 GameObject）。
2. Canvas 层**不能是 UI**（设为 Default 等），否则菜单未打开时无法交互。
3. Canvas「Render Mode」设为 **World Space**（Screen Space 跟随相机、不可交互）。
4. 必须有 Box Collider（无则自动加）。
5. 场景必须有 EventSystem（创建 Canvas 时自动加，别删）。
6. 交互元素需有 `GraphicRaycaster` + 带 `Raycast Target` 的 Image；被透明元素遮挡时关掉文字 raycast target。
7. 建议缩放到 ~0.01（1px=1m 太大）、Navigation 设 None、用 TextMeshPro。

官方 UI 创建路径：`右键 Hierarchy > UI > TextMeshPro (VRC)` 自动配置 Canvas + VRC_UIShape。

**[FACT] Focus View**：`VRC_UIShape.Allow Focus View` 允许手机/平板用户放大/缩放世界 UI（条件见官方页）。

**[FACT/更新日志] 版本坑**：[VRChat 2023.2.3](https://docs.vrchat.com/docs/vrchat-202323) — 从禁用状态启用 Canvas/VRC_UIShape 的射线现在正常工作；但「永久禁用 VRC_UIShape 来开启对 active canvas 的射线」是 legacy 行为，可能降性能，应保持 VRC_UIShape enabled。

**[FACT] UI 交互 vs Interact**：VRC_UIShape 支持远程指针/点击/滚动，比 Interact 事件更易用。

**[COMMUNITY] 手机端 UI 手感**：WASD 输入框打字会触发移动（社区 [Wikidot](http://vrchat.wikidot.com/world-component:vrc-uishape) 记录，无官方规避法）。

## 性能数据（unityvrchat 或 [UNKNOWN]）

本地 `[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`：**未包含** Canvas 性能数据 → **[UNKNOWN]**。

本地 `entities/api/ui.md` 提示（[INFERENCE]，非基准数据）：Canvas rebuild 在 Udon VM 有额外成本；频繁改 Text 触发网格重建，建议节流 UI 2–5Hz。

## 社区佐证

- [VRChat Unofficial Wiki — VRC UiShape](http://vrchat.wikidot.com/world-component:vrc-uishape)（[COMMUNITY]）：World UI 配置细节（EventSystem、collider trigger、输入框 WASD 问题）。
- 本地 [[../../api/ui.md|UI API]]：Udon 操作 Canvas（Button.onClick 编辑器绑定、Slider.value、避免 runtime AddListener）。

## 相关页面

[[../../api/ui.md|UI API]] · [[../whitelisted-world-components.md|Allowlisted World Components]] · [[audiosource.md|AudioSource]] · [[videoplayer.md|VideoPlayer]] · [[canvasgroup.md|CanvasGroup]]

## 源清单

- Unity Manual Canvas: https://docs.unity3d.com/2022.3/Documentation/Manual/class-Canvas.html
- Unity Manual Canvas Scaler: https://docs.unity3d.com/2022.3/Documentation/Manual/script-CanvasScaler.html
- Unity Scripting API Canvas: https://docs.unity3d.com/2022.3/Documentation/ScriptReference/Canvas.html
- VRChat VRC Ui Shape: https://creators.vrchat.com/worlds/components/vrc_uishape/
- VRChat Allowlist: https://creators.vrchat.com/worlds/whitelisted-world-components/
- VRChat 2023.2.3 更新日志: https://docs.vrchat.com/docs/vrchat-202323
- 本地: [[../../api/ui.md|UI API]]