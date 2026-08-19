---
title: "LineRenderer — 世界渲染组件"
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
  - rendering
  - performance
aliases:
  - LineRenderer
  - 线条渲染器
related:
  - ../whitelisted-world-components.md
  - ../performance-guide.md
  - trailrenderer.md
  - meshrenderer.md
  - skinnedmeshrenderer.md
  - particlesystem.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-LineRenderer.html | https://docs.unity3d.com/ScriptReference/LineRenderer.html | https://docs.unity3d.com/2022.3/Documentation/Manual/visual-effects-lines-trails-billboards.html | https://creators.vrchat.com/worlds/whitelisted-world-components/ | https://feedback.vrchat.com/sdk-bug-reports/p/alphablended-on-particles-not-android-compatible-despite-in-vrchat-mobile-catego"
updated: 2026-08-17
---
# LineRenderer — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无线渲染/拖尾测试项，详见性能小节）

---

## 核心功能（官方，[FACT]）

[FACT] [Unity 2022.3 Manual — Line Renderer component](https://docs.unity3d.com/2022.3/Documentation/Manual/class-LineRenderer.html)：**Line Renderer 取 3D 空间中的 2 个或更多点，在相邻点之间绘制直线**，可绘制从简单直线到复杂螺旋的任意连线。**线条始终连续**；要画多条独立线需多个带各自 Line Renderer 的 GameObject。

- [FACT] 线宽以**世界单位**（非像素）渲染多边形；渲染算法与 [Trail Renderer](https://docs.unity3d.com/2022.3/Documentation/Manual/class-TrailRenderer.html) 相同。
- [FACT] 默认材质为内置 **Default-Line**；自定义贴图/效果可用 **Standard Particle Shaders**（内置粒子 shader 与 LineRenderer 适配良好）。

## 关键属性/行为

| 属性 | 说明（[FACT]） |
|---|---|
| **Positions** | `Vector3` 点数组；**Loop** 连接首尾成闭环。 |
| **Width / Color** | 沿长度用曲线/渐变控制宽度与颜色。 |
| **Corner Vertices / End Cap Vertices** | 控制拐角与端盖顶点数（越大约圆）。 |
| **Alignment** | View（朝向相机，billboard 式）或 TransformZ。 |
| **Texture Mode** | Stretch / Tile / DistributePerSegment / RepeatPerSegment，控制贴图沿线的映射。 |
| **Use World Space** | 点为世界坐标（开启）或本地坐标（关闭）。 |
| **Generate Lighting Data** | 构建带法线/切线的几何，使材质可用场景光照。 |
| **Shadow Bias** | 修正 billboard 几何投影伪影。 |
| **Renderer 基类继承** | Cast Shadows / Receive Shadows / Light Probes / Reflection Probes / Anchor Override / Dynamic Occlusion / Sorting Layer 等（与其他 Renderer 一致）。 |
| **场景编辑模式** | Edit Points / Create Points / Simplified（Ramer-Douglas-Peucker 算法减少点数）。 |

## VRChat 特定限制/注意

**[FACT] 白名单地位**：LineRenderer 在 VRChat World「Allowlisted World Components」官方白名单 Unity Components 列表中（[官方白名单页](https://creators.vrchat.com/worlds/whitelisted-world-components/)，Nov 25, 2025）。详见 [[../whitelisted-world-components.md|Allowlisted World Components]]。

**[INFERENCE] 画笔/轨迹常用**：本地知识库中 LineRenderer 被 21 个文件提及，多与**画笔系统**（drawing / spray pen）相关。VRChat 世界常用 Udon 在每帧更新 Positions 实现动态画笔/激光指示。

**[INFERENCE] 使用注意**：`Use World Space` 下逐帧更新大量 Positions 会带来 CPU 开销；`Generate Lighting Data` 增加几何构建成本，纯 Unlit 画笔可关掉。Shadow/透明 billboard 线在移动端可能昂贵（结合移动端避免过度透明，见 [[particlesystem.md|ParticleSystem]] 的 Quest 说明）。

**[FACT] 透明度**：移动端 MeshRenderer 透明不受支持（粒子透明才支持，[VRChat feedback](https://feedback.vrchat.com/sdk-bug-reports/p/alphablended-on-particles-not-android-compatible-despite-in-vrchat-mobile-catego)）；LineRenderer 实质是 billboard 四边面，透明实现请选用 VRChat/Mobile 兼容 shader 并在 Quest 实机验证。

## 性能数据（unityvrchat 或 [UNKNOWN]）

[UNKNOWN] — `[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）` **未收录 LineRenderer 帧时间数据**（该源无线渲染/拖尾测试）。

邻近推断（[INFERENCE]，非本体数据）：线渲染单条 draw 量小（低顶点 billboard 几何），但**逐帧修改 Positions 会触发线几何重建**——CPU 成本随点数量与更新频率上升；大量同时活跃的画笔/激光线会放大该成本。参见 [[../performance-guide.md|Performance Guide]] 的动态系统审查建议。

## 社区佐证

- [SECONDARY] 本地既有知识库大量以 LineRenderer 描述 VRChat 画笔/激光系统（exposure/画笔文档，`01-gap-matrix.md` 记 21 文件提及）。

## 相关页面

[[../whitelisted-world-components.md|Allowlisted World Components]] · [[../performance-guide.md|Performance Guide]] · [[trailrenderer.md|TrailRenderer]] · [[meshrenderer.md|MeshRenderer]] · [[skinnedmeshrenderer.md|SkinnedMeshRenderer]] · [[particlesystem.md|ParticleSystem]]

## 源清单

- Unity 2022.3 Manual — Line Renderer: https://docs.unity3d.com/2022.3/Documentation/Manual/class-LineRenderer.html
- Unity Scripting API — LineRenderer: https://docs.unity3d.com/ScriptReference/LineRenderer.html
- Unity 2022.3 Manual — Lines, trails, and billboards: https://docs.unity3d.com/2022.3/Documentation/Manual/visual-effects-lines-trails-billboards.html
- VRChat 官方白名单: https://creators.vrchat.com/worlds/whitelisted-world-components/
- VRChat feedback（透明支持范围）: https://feedback.vrchat.com/sdk-bug-reports/p/alphablended-on-particles-not-android-compatible-despite-in-vrchat-mobile-catego