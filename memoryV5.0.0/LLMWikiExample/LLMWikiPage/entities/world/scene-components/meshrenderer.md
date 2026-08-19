---
title: "MeshRenderer — 世界网格渲染组件"
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
  - mesh
  - material
  - optimization
  - shader
aliases:
  - MeshRenderer
  - 网格渲染器
related:
  - ../whitelisted-world-components.md
  - ../performance-guide.md
  - ../../avatar/avatar-optimizer.md
  - ../shader/index.md
  - linerenderer.md
  - trailrenderer.md
  - skinnedmeshrenderer.md
  - ../vrc-light-volumes.md
  - ../reflection-probes.md
  - ../occlusion-culling-guide.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-MeshRenderer.html | https://docs.unity3d.com/ScriptReference/MeshRenderer.html | https://creators.vrchat.com/worlds/whitelisted-world-components/ | https://creators.vrchat.com/platforms/android/quest-content-optimization | https://feedback.vrchat.com/sdk-bug-reports/p/alphablended-on-particles-not-android-compatible-despite-in-vrchat-mobile-catego"
updated: 2026-08-17
---
# MeshRenderer — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无渲染器本体帧时数据，详见性能小节）

---

## 核心功能（官方，[FACT]）

[FACT] [Unity 2022.3 Manual — Mesh Renderer component](https://docs.unity3d.com/2022.3/Documentation/Manual/class-MeshRenderer.html) + [MeshRenderer Scripting API](https://docs.unity3d.com/ScriptReference/MeshRenderer.html)：**Mesh Renderer 渲染网格；它与同一 GameObject 上的 Mesh Filter 配合，Mesh Renderer 渲染 Mesh Filter 引用的网格**（含子网格）。**渲染可变形的网格应用 SkinnedMeshRenderer 替代**。无需变形时，静态网格用 MeshRenderer+MeshFilter。

- [FACT] 继承 `Renderer` 基类，因此具备大量与其他 Renderer（Line/Trail/Skinned）共通的属性：Cast Shadows / Receive Shadows、Contribute/Receive Global Illumination（Lightmaps / Light Probes）、Reflection Probes（Simple / Blend Probes / Blend Probes and Skybox）、Anchor Override、Motion Vectors、Dynamic Occlusion、Sorting Layer、Order in Layer。
- [FACT] **材质**：Materials 列表驱动多材质子网格；同一材质可被多对象共享以省绘制调用。

## 关键属性/行为

| 属性 | 说明（[FACT]） |
|---|---|
| **material / materials** | 决定外观；多对象复用一个材质可利于批处理（材质数量直接关联绘制调用）。 |
| **shadowCastingMode** | Renderer 基类 Cast Shadows 属性：On/Off/Two Sided/Shadows Only。 |
| **receiveShadows** | Renderer 基类 Receive Shadows：是否接收其他光源投射的阴影。 |
| **motionVectorMode** | Renderer 基类 Motion Vectors：相机运动矢量处理方式（影响动态模糊等后处理）。 |
| **Receive GI** | Lightmaps 或 Light Probes 二选一供应 GI 数据（`MeshRenderer.receiveGI`）。 |
| **Reflection Probes / Anchor Override** | 接收反射探针 cubemap 的方式与锚点（默认锚点为渲染器包围盒中心）。 |
| **Dynamic Occlusion** | 被静态遮挡物遮蔽时是否被遮挡剔除（默认开）。 |

## VRChat 特定限制/注意

**[FACT] 白名单地位**：MeshRenderer 在 VRChat World「Allowlisted World Components」官方白名单 Unity Components 列表中（[官方白名单页](https://creators.vrchat.com/worlds/whitelisted-world-components/)，Nov 25, 2025）。详见 [[../whitelisted-world-components.md|Allowlisted World Components]]。

**[FACT] Quest 透明受限**：移动端 **MeshRenderer 透明不受支持**（与粒子不同，[VRChat feedback](https://feedback.vrchat.com/sdk-bug-reports/p/alphablended-on-particles-not-android-compatible-despite-in-vrchat-mobile-catego)）；移动端只能用 VRChat Mobile shaders（[Android Content Optimization](https://creators.vrchat.com/platforms/android/quest-content-optimization)）。

**[FACT] 材质/Shader 语境**：VRChat 世界常用 Poiyomi / lilToon 等社区 shader 赋予世界网格风格化材质；这些 shader 往往集成 VRC Light Volumes / 反射探针支持（见 [[../vrc-light-volumes.md|VRC Light Volumes]] 的兼容 shader 表，其中 Poiyomi Toon v9.2.67+、lilToon v2.0.0+ 支持 Light Volumes）。[[../shader/index.md|Shader]] 目录收录相关材质/shader 知识。

**[INFERENCE] 烘焙关系**：世界网格（MeshRenderer）是烘焙光照（Bakery / Glim / Unity Progressive）的接收主体；静态、非烘焙网格需标记 Static 并具备光照贴图 UV 才能接收烘焙（见 [[../performance-guide.md|Performance Guide]] 第 11 节「烘焙光照」与 [[../reflection-probes.md|Reflection Probes]]）。

**[INFERENCE] 性能排序：材质结合是核心维度**：材质泛滥=更多绘制调用（[[../performance-guide.md|Performance Guide]] 第 3 节「材质管理」：更多材质 = 更多绘制调用）；世界优化建议**复用材质、保持较小一致的材质集、按需合并材质（含材质槽合并/DrawCall 合并，如 [[../../avatar/avatar-optimizer.md|Avatar Optimizer]] 的 Merge Material / Merge Skinned Mesh 同类思路）**；世界侧约 250,000 三角形预算（Quest，[World Optimization Checklist](https://vrcreators.net/docs/vrchat/world-optimization-checklist)）。

## 性能数据（unityvrchat 或 [UNKNOWN]）

[UNKNOWN] — `[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）` **未收录 MeshRenderer 帧时间数据**。该源「后续计划」（第 204–207 行）将 **Skinned mesh renderers**（材质/网格数量关系）列为未来测试项，MeshRenderer 亦无定量数据。

可用定性参考（非该源）：
- [FACT] [Unity Shadow Performance](https://docs.unity3d.com/Manual/ShadowPerformance.html)（官方，未在本轮抓取正文，作为已知官方性能注意点——注：本轮未独立抓取，标记为推断性引用）。
- [INFERENCE] 材质数量/绘制调用是主要成本维度（[[../performance-guide.md|Performance Guide]] 第 3 节）。

## 社区佐证

- [SECONDARY] [VR Creators — Quest Compatibility Checklist](https://vrcreators.net/docs/vrchat/quest-compatibility-checklist)：整体顶点/三角形预算紧、只能用移动 shader。
- [SECONDARY] [VR Creators — World Optimization Checklist](https://vrcreators.net/docs/vrchat/world-optimization-checklist)：移动端避免过度透明与 alpha 混合。

## 相关页面

[[../whitelisted-world-components.md|Allowlisted World Components]] · [[../performance-guide.md|Performance Guide]] · [[../../avatar/avatar-optimizer.md|Avatar Optimizer]] · [[../shader/index.md|Shader]] · [[linerenderer.md|LineRenderer]] · [[trailrenderer.md|TrailRenderer]] · [[skinnedmeshrenderer.md|SkinnedMeshRenderer]] · [[../vrc-light-volumes.md|VRC Light Volumes]] · [[../reflection-probes.md|Reflection Probes]] · [[../occlusion-culling-guide.md|Occlusion Culling Guide]]

## 源清单

- Unity 2022.3 Manual — Mesh Renderer: https://docs.unity3d.com/2022.3/Documentation/Manual/class-MeshRenderer.html
- Unity Scripting API — MeshRenderer: https://docs.unity3d.com/ScriptReference/MeshRenderer.html
- Unity 2022.3 Manual — Line Renderer（Renderer 通用 Lighting/Probes 参考）: https://docs.unity3d.com/2022.3/Documentation/Manual/class-LineRenderer.html
- VRChat 官方白名单: https://creators.vrchat.com/worlds/whitelisted-world-components/
- VRChat Android Content Optimization: https://creators.vrchat.com/platforms/android/quest-content-optimization
- VRChat feedback（透明支持范围）: https://feedback.vrchat.com/sdk-bug-reports/p/alphablended-on-particles-not-android-compatible-despite-in-vrchat-mobile-catego
- 既有页（只读引用）：`entities/world/performance-guide.md`（材质/烘焙）、`entities/world/vrc-light-volumes.md`（兼容 shader 表）、`entities/world/reflection-probes.md`、`entities/world/occlusion-culling-guide.md`