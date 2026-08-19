---
title: "Light — 世界光照组件"
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
  - light
  - baking
  - performance
  - quest
  - probe
aliases:
  - Light
  - 光源
related:
  - ../whitelisted-world-components.md
  - particlesystem.md
  - particlesystemforcefield.md
  - ../../avatar/avatar-particle-system-limits.md
  - ../performance-guide.md
  - ../vrc-light-volumes.md
  - ../reflection-probes.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-Light.html | https://creators.vrchat.com/worlds/whitelisted-world-components/ | https://creators.vrchat.com/platforms/android/quest-content-limitations | https://docs.vrchat.com/docs/avatar-optimizing-tips"
updated: 2026-08-17
---
# Light — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（明确将 Lights 列为未公布测试项，详见性能小节）

---

## 核心功能（官方，[FACT]）

[FACT] [Unity 2022.3 Manual — Lights](https://docs.unity3d.com/2022.3/Documentation/Manual/class-Light.html)：**Light 组件定义场景光源**。类型：**Directional（方向光）、Point（点光）、Spot（聚光）、Area（矩形/圆盘面光）**。Inspector 属性因渲染管线而异（Built-in / URP / HDRP）；VRChat 用 Built-in Render Pipeline 语义为主。

- [FACT] 关键属性（Built-in）：**Type、Range**（点/聚光有效距离）、**Spot Angle**、**Color**、**Mode**（Realtime / Mixed / Baked）、**Intensity**、**Indirect Multiplier**（间接光强度，调 0 可让实时 GI 下某灯只发直射光）、**Shadow Type**（Hard/Soft/None）、**Shadow 参数**（Strength/Resolution/Bias/Normal Bias/Near Plane）、**Cookie**、**Draw Halo**、**Flare**、**Render Mode**（Auto/Important/Not Important）、**Culling Mask**。
- [FACT] 性能提示（官方 Hints）：Spot+cookie 适合窗光；低强度点光补纵深；**要最大性能用 VertexLit shader**（仅逐顶点光照，低端卡高吞吐）。

## VRChat 特定限制/注意

**[FACT] 白名单地位**：Light 在 VRChat World「Allowlisted World Components」官方白名单的 Unity Components 列表中（[官方白名单页](https://creators.vrchat.com/worlds/whitelisted-world-components/)，Nov 25, 2025）。详见 [[../whitelisted-world-components.md|Allowlisted World Components]]。

**[FACT] Quest 限制**：[Android Content Limitations](https://creators.vrchat.com/platforms/android/quest-content-limitations) 与 [Quest Compatibility Checklist](https://vrcreators.net/docs/vrchat/quest-compatibility-checklist)：Avatar lights 在 Quest 禁用；世界建议大量烘焙、少实时光、无实时阴影。

**[FACT] 烘焙是推荐路线**：既有页 [[../performance-guide.md|Performance Guide]] 第 7/11 节：Quest 推荐大量烘焙光照；用 Light Probes 让 Avatar 与动态对象接收烘焙采样光照；实时阴影很贵。

**[FACT] VRC Light Volumes 替代**：既有页 [[../vrc-light-volumes.md|VRC Light Volumes]]：社区工具 VRCLightVolumes 的 Regular Light Volumes 替代 Unity Light Probes、Point Light Volumes 替代实时 Light（最多 128 个同时可见、Area Light ×8 成本、不支持实时阴影、烘焙 Shadow Mask 支持每个 volume ≤4 阴影光源）。→ 世界创作者常用它替代大量实时 Light + Light Probes。

**[FACT] 粒子挂 Light 极贵**：Avatar Optimizing Tips（[docs.vrchat.com](https://docs.vrchat.com/docs/avatar-optimizing-tips)）——每个带 Light 的粒子=一个实时光（极贵）。详见 [[particlesystem.md|ParticleSystem]] 与 [[../../avatar/avatar-particle-system-limits.md|Avatar Particle System Limits]]。

[INFERENCE] **Bakery / Glim 烘焙关系**：世界光照烘焙常用 **Bakery**（GPU 光贴图烘焙器）或 **Glim**（社区光烘焙）；Light 组件在烘焙流程中作为光源输入，世界场景静态部分把 Light Mode 设为 Baked/Mixed 并用光贴图+反射探针。既有页 [[../vrc-light-volumes.md|VRC Light Volumes]] 明言「Bakery 优质烘焙（推荐配合 Light Volumes）」；反射探针协同见 [[../reflection-probes.md|Reflection Probes]]。

[INFERENCE] 动态/闪烁/移动灯是高成本项（[[../performance-guide.md|Performance Guide]] 第 7 节动态光常见问题）。

## 性能数据（unityvrchat 或 [UNKNOWN]）

[UNKNOWN] — `[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）` **明确把 Lights 列为「后续计划」未公布测试项**（第 204–211 行原文：「未来我打算进行基准测试的组件包括：…Lights…」）。故该源无 Light 帧时间数据。

定性参考（非该源）：

- [FACT] 官方 [Avatar Optimizing Tips](https://docs.vrchat.com/docs/avatar-optimizing-tips)：每粒子挂光=一个实时光，极贵；建议避免大量实时阴影/点光堆叠（[[../performance-guide.md|Performance Guide]]）。

## 社区佐证

- [COMMUNITY/SECONDARY] [VR Creators — Quest Compatibility Checklist](https://vrcreators.net/docs/vrchat/quest-compatibility-checklist)：Quest 上 avatar lights 禁用；烘焙/静态/移动 shader 优先。
- [COMMUNITY/SECONDARY] [uploadvr — VRChat Quest 细节](https://www.uploadvr.com/vrchat-quest-limitations-details/)：Quest 世界强烈建议烘焙、静态、动态批处理、移动 shader；不建议实时阴影与大量粒子。
- 本地知识库大量提及 Light（exposure/光照文档旁引）。

## 相关页面

[[particlesystem.md|ParticleSystem]] · [[particlesystemforcefield.md|ParticleSystemForceField]] · [[lightprobegroup.md|LightProbeGroup]] · [[../../avatar/avatar-particle-system-limits.md|Avatar Particle System Limits]] · [[../whitelisted-world-components.md|Allowlisted World Components]] · [[../performance-guide.md|Performance Guide]] · [[../vrc-light-volumes.md|VRC Light Volumes]] · [[../reflection-probes.md|Reflection Probes]]

## 源清单

- Unity 2022.3 Manual — Lights: https://docs.unity3d.com/2022.3/Documentation/Manual/class-Light.html
- Unity Scripting API — Light: https://docs.unity3d.com/ScriptReference/Light.html
- VRChat 官方白名单: https://creators.vrchat.com/worlds/whitelisted-world-components/
- VRChat Android Content Limitations: https://creators.vrchat.com/platforms/android/quest-content-limitations
- VRChat Avatar Optimizing Tips (docs.vrchat.com): https://docs.vrchat.com/docs/avatar-optimizing-tips
- 既有页（只读引用）：
  - `entities/world/performance-guide.md`（烘焙 vs 动态、Light Probes、Quest 建议）
  - `entities/world/vrc-light-volumes.md`（VRCLightVolumes 替代、Bakery 配合、128 动态光源限制）
  - `entities/world/reflection-probes.md`
  - `entities/avatar/avatar-particle-system-limits.md`（粒子挂光语境）