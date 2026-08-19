---
title: "SkinnedMeshRenderer — 世界蒙皮网格渲染组件"
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
  - quest
aliases:
  - SkinnedMeshRenderer
  - 蒙皮网格渲染器
related:
  - ../whitelisted-world-components.md
  - ../performance-guide.md
  - ../../avatar/avatar-optimizer.md
  - ../shader/index.md
  - linerenderer.md
  - trailrenderer.md
  - meshrenderer.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-SkinnedMeshRenderer.html | https://docs.unity3d.com/ScriptReference/SkinnedMeshRenderer.html | https://creators.vrchat.com/worlds/whitelisted-world-components/ | https://vrcreators.net/docs/vrchat/quest-compatibility-checklist | https://feedback.vrchat.com/sdk-bug-reports/p/alphablended-on-particles-not-android-compatible-despite-in-vrchat-mobile-catego"
updated: 2026-08-17
---
# SkinnedMeshRenderer — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（Skinned renderers 列为未公布测试项，Cloth 章节有定性数据，详见性能小节）

---

## 核心功能（官方，[FACT]）

[FACT] [Unity 2022.3 Manual — Skinned Mesh Renderer component](https://docs.unity3d.com/2022.3/Documentation/Manual/class-SkinnedMeshRenderer.html)：**Skinned Mesh Renderer 渲染可变形的网格**，包括：带骨骼与绑定姿势的蒙皮网格（skinned meshes）、带有 Blend Shapes 的网格、以及运行布料模拟（cloth）的网格。渲染规则网格用 Mesh Renderer + Mesh Filter。

- [FACT] 继承 `Renderer` 基类（与 Line/Trail/Mesh Renderer 共通：Cast/Receive Shadows、Probes、Anchor Override、Dynamic Occlusion 等）。

## 关键属性/行为

| 属性 | 说明（[FACT]） |
|---|---|
| **Bounds** | 用于离屏判定。蒙皮网格变形时 bounds 会变；Unity 在**导入时取所有动画下最大包围体**做可见性判定。叠加动画、脚本改骨骼位置、顶点 shader 改顶点、ragdoll 可能把顶点/骨骼推出已知最大 bounds。解决方案：**手动改 Bounds 匹配最大可能包围体（性能更好）**，或开启 **Update When Offscreen**（每帧重算 bounds，性能差，仅 ragdoll 等不可预测场景）。 |
| **BlendShapes** | 存储网格中 blend shapes 的权重值。 |
| **Quality / Skin Weights** | 顶点可被影响的最大骨骼数（1/2/4 bone 或 Auto=全局 Quality 限制），**影响性能**。建议导入时在 Model Import Settings → Rig → Skin Weights 设定，而非运行时截断。 |
| **Update When Offscreen** | 离屏时是否继续蒙皮（默认关闭以省性能）。 |
| **Mesh** | 须含合法 bind pose + skin weights，或用 blend shapes / cloth。 |
| **rootBone** | 骨架根 transform；bounds 随其移动。 |
| **Skinned Motion Vectors** | 双缓冲蒙皮数据以插值蒙皮运动（更多 GPU 内存，更正确的运动向量）。 |

## VRChat 特定限制/注意

**[FACT] 白名单地位**：SkinnedMeshRenderer 在 VRChat World「Allowlisted World Components」官方白名单 Unity Components 列表中（[官方白名单页](https://creators.vrchat.com/worlds/whitelisted-world-components/)，Nov 25, 2025）；也是 **Avatar 核心渲染组件**（眼部/觉醒由 BlendShape 驱动）。详见 [[../whitelisted-world-components.md|Allowlisted World Components]]。

**[INFERENCE] BlendShape 性能**：Blend Shape 权重按帧驱动（如 facial tracking、AudioLink）会增加 CPU/GPU 变形负担；过多样本与高顶点权重的 BlendShape 是 Avatar 性能排名的重要扣分项。Avatar 侧可用 [[../../avatar/avatar-optimizer.md|Avatar Optimizer]] 的 Freeze BlendShape / Remove Mesh By BlendShape 组件收敛成本。

**[FACT] 骨骼权重预算**：官方推荐导入时限制每顶点骨骼数（Skin Weights），过高的 bone influence 增计算资源（上引 Unity 官方页 Quality 项）。Quest/Avatar 上尤其重要。

**[FACT] Cloth**：SkinnedMeshRenderer 可驱动 cloth，但 VRChat Quest/Avatar 明确**禁用 Avatar cloth**（[Quest Compatibility Checklist](https://vrcreators.net/docs/vrchat/quest-compatibility-checklist)）；cloth 本身是 heavy（`unityvrchat.md` 第 186–198 行 Cloth 章节：每 1000 顶点约 0.2ms，>20 万顶点后锐增；Mirror/Shadow clone 会 ×3 模拟；collider 约 ×2 成本）。

**[INFERENCE] 与 MeshRenderer 的区别**：不需要骨骼蒙皮的静态网格请用 [[meshrenderer.md|MeshRenderer]]（省蒙皮计算）；Avatars 与带动画/变形的部分用 SkinnedMeshRenderer。

**[INFERENCE] 常见优化（AAO 合并）**：Avatar/世界侧常用合并类工具降低 SkinnedMeshRenderer 成本——[[../../avatar/avatar-optimizer.md|Avatar Optimizer]] 的 **Merge Skinned Mesh**（合并多个 SkinnedMeshRenderer/MeshRenderer 为一个，降 DrawCall/CPU）与 **Merge Material**（同 Shader 材质合并，降材质槽）是既有成熟做法；合并时注意其已知局限（不自动设置 Root Bone、BlendShape/材质动画冲突）。

**[FACT] 透明/shader**：移动端 SkinnedMeshRenderer（类网格）透明受支持性受限，只能 VRChat Mobile shaders（同 MeshRenderer 条目，[VRChat feedback](https://feedback.vrchat.com/sdk-bug-reports/p/alphablended-on-particles-not-android-compatible-despite-in-vrchat-mobile-catego)）；Avatar shader 常用 lilToon/Poiyomi（支持 Light Volumes，见 [[../shader/index.md|Shader]] 与 `vrc-light-volumes.md`）。

## 性能数据（unityvrchat 或 [UNKNOWN]）

[UNKNOWN] — `[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）` **明确将 Skinned mesh renderers（材质数量 vs 网格数量、性能）列为「后续计划」未公布的测试项**（第 204–207 行原文：「未来我打算进行基准测试的组件包括：…Skinned mesh renderers（如材质数量与网格数量的关系）…Lights…」）。故无该源帧时间数据。

可用定性数据（`unityvrchat.md` Cloth 章节，SkinnedMeshRenderer 驱动 cloth 场景，[FACT] 非本体通用数据）：
- Cloth：~20 万顶点内每增 1000 顶点 +0.2ms；>20 万后锐增；Mirror/Shadow clone ×3 模拟；每 10 个 collider 帧时间约 ×2。
- [INFERENCE] 材质数量/网格数量的关系被该源列为待测项，提示 SkinnedMeshRenderer 的成本主要来自蒙皮计算与材质槽数量而非单一网格本身（[[../performance-guide.md|Performance Guide]] 第 3 节材质语境）。

## 社区佐证

- [SECONDARY] [VR Creators — Quest Compatibility Checklist](https://vrcreators.net/docs/vrchat/quest-compatibility-checklist)：Avatar cloth 禁用、三角形/顶点预算紧。
- 本地知识库 45 文件提及 SkinnedMeshRenderer（exposure/优化，`01-gap-matrix.md` 1.1 节）。

## 相关页面

[[../whitelisted-world-components.md|Allowlisted World Components]] · [[../performance-guide.md|Performance Guide]] · [[../../avatar/avatar-optimizer.md|Avatar Optimizer]] · [[../shader/index.md|Shader]] · [[linerenderer.md|LineRenderer]] · [[trailrenderer.md|TrailRenderer]] · [[meshrenderer.md|MeshRenderer]]

## 源清单

- Unity 2022.3 Manual — Skinned Mesh Renderer: https://docs.unity3d.com/2022.3/Documentation/Manual/class-SkinnedMeshRenderer.html
- Unity Scripting API — SkinnedMeshRenderer: https://docs.unity3d.com/ScriptReference/SkinnedMeshRenderer.html
- VRChat 官方白名单: https://creators.vrchat.com/worlds/whitelisted-world-components/
- VRChat Quest Compatibility Checklist (VR Creators): https://vrcreators.net/docs/vrchat/quest-compatibility-checklist
- VRChat feedback（透明支持范围）: https://feedback.vrchat.com/sdk-bug-reports/p/alphablended-on-particles-not-android-compatible-despite-in-vrchat-mobile-catego
- 本地性能源（clothing 定性数据）：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）` 第 186–198 行（Cloth 章节）