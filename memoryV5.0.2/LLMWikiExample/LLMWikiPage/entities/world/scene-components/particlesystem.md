---
title: "ParticleSystem — 世界粒子组件"
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
  - particle
  - performance
  - quest
  - shader
aliases:
  - ParticleSystem
  - 粒子系统
related:
  - ../whitelisted-world-components.md
  - particlesystemforcefield.md
  - light.md
  - ../../avatar/avatar-particle-system-limits.md
  - ../performance-guide.md
  - ../vrc-light-volumes.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-ParticleSystem.html | https://creators.vrchat.com/worlds/whitelisted-world-components/ | https://docs.vrchat.com/docs/avatar-particle-system-limits | https://docs.vrchat.com/docs/avatar-optimizing-tips"
updated: 2026-08-17
---
# ParticleSystem — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无本组件独立帧时间数据，详见性能小节）

---

## 核心功能（官方，[FACT]）

[FACT] [Unity 2022.3 Manual — Particle System](https://docs.unity3d.com/2022.3/Documentation/Manual/class-ParticleSystem.html)：**Particle System 组件通过生成并驱动大量小型 2D 图像（或网格）来模拟液体、云、火焰等流体形态实体**。

- [FACT] Inspector 将大量属性组织为可折叠的「模块」（modules），各模块在 [Particle System Modules](https://docs.unity3d.com/2022.3/Documentation/Manual/ParticleSystemModules.html) 单独说明；可启用/禁用单个模块（如 Size over Lifetime）。
- [FACT] Main 模块（[MainModule Scripting API](https://docs.unity3d.com/2021.1/Documentation/ScriptReference/ParticleSystem.MainModule.html)）含 `loop`、`duration`（播放时长）、`maxParticles`（最大粒子数）、`simulationSpace`（世界/局部空间模拟）、`scalingMode`、`playOnAwake`、`prewarm`、`simulationSpeed`、`cullingMode`（离屏时是否仍每帧模拟）、`stopAction` 等。
- [FACT] `cullingMode` 决定系统离屏时是否继续模拟 —— 离屏停止模拟对性能关键。

## 关键属性/行为

| 属性/模块 | 说明（[FACT]） |
|---|---|
| **Render Mode（ParticleSystemRenderer）** | Render Mode = Mesh 时粒子可用网格渲染；`ParticleSystemRenderer` 负责把粒子渲染成 billboard / mesh / stretched 等形式。 |
| **Trails 选项** | 可为粒子生成拖尾，VRChat 特别关注（对应 `ps_trails_penalty`，见下）。 |
| **Collision 模块** | 支持与场景碰撞体碰撞，质量分 High/Medium/Low。 |
| **External Forces 模块** | 启用后粒子受 `ParticleSystemForceField`（力场）或 Layer Mask 指定力场影响（[FACT] [ParticleSystemForceField Manual](https://docs.unity3d.com/2022.3/Documentation/Manual/class-ParticleSystemForceField.html)）。 |

## VRChat 特定限制/注意

**[FACT] 白名单地位**：ParticleSystem 在 VRChat World「Allowlisted World Components」官方白名单的 Unity Components 列表中（[官方白名单页](https://creators.vrchat.com/worlds/whitelisted-world-components/)，Last updated Nov 25, 2025）；`ParticleSystemRenderer` 亦在列。详见 [[../whitelisted-world-components.md|Allowlisted World Components]]。

**[FACT] Quest/Android 粒子严格受限**：[Android Content Limitations — Components](https://creators.vrchat.com/platforms/android/quest-content-limitations)：「Particles are limited heavily on avatars for Android and Quest, with settings mirroring the Avatar Particle System Limits on PC.」即 Quest 端粒子限制镜像 PC 的 Avatar Particle System Limits 且默认启用、**不可禁用**。

**[FACT] 用户端粒子限制（Particle System Limits，ps_\* 配置）**：[docs.vrchat.com — Avatar Particle System Limits](https://docs.vrchat.com/docs/avatar-particle-system-limits)（宇宙默认 config：`ps_max_particles=50000`、`ps_max_systems=200`、`ps_max_total_emission=40000`、`ps_collision_penalty_high/med/low=50/30/10`、`ps_trails_penalty=10`、`ps_mesh_particle_divider=60`、`ps_mesh_particle_poly_limit=50000`）。详见 [[../../avatar/avatar-particle-system-limits.md|Avatar Particle System Limits]]。

- [INFERENCE] 关键 Insight：实际最大粒子 ≈ `ps_max_particles / (meshPoly/divider) / collision_penalty / trails_penalty`（既有页记 [INFERENCE]，原文未给出公式）。
- 创作者建议：单系统粒子 <50000、网格多边形尽量低、慎用高质量碰撞（penalty 50）、**Trails 是惩罚最重的特性**。

**[FACT] 粒子带 Light = 实时光（极贵）**：[docs.vrchat.com — Avatar Optimizing Tips](https://docs.vrchat.com/docs/avatar-optimizing-tips)：「Each particle with a light counts as a real time light, which is extremely expensive.」→ 不要给粒子挂点光源。相关协同见 [[light.md|Light]]。

**[FACT] Shaders**：Quest/Android 上 Avatar 只能用 VRChat Mobile shaders（[Android Content Optimization](https://creators.vrchat.com/platforms/android/quest-content-optimization)）；粒子透明在 Quest 受支持，但 **MeshRenderer 透明不受支持**（[VRChat 官方 feedback#AlphaBlended](https://feedback.vrchat.com/sdk-bug-reports/p/alphablended-on-particles-not-android-compatible-despite-in-vrchat-mobile-catego)）。

[INFERENCE] **世界侧粒子**：官方建议限制粒子系统数量与单时刻最大发射量；动态合批 sprite 粒子、不用碰撞、简化粒子运动可提升性能（[Avatar Optimizing Tips](https://docs.vrchat.com/docs/avatar-optimizing-tips)）；世界优化检查表把「常驻粒子/动画特效」当作需要 justify 的成本项（[VR Creators — World Optimization Checklist](https://vrcreators.net/docs/vrchat/world-optimization-checklist)）。

[INFERENCE] **VRCShader 关系**：粒子材质使用 VRChat 白名单 shader（VRChat/Mobile、Unlit 等），世界粒子 mesh 渲染需低面数网格以避免 mesh particle penalty。

## 性能数据（unityvrchat 或 [UNKNOWN]）

[UNKNOWN] — `[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（VRCLibrary 性能基准移植）**未收录 ParticleSystem 的具体帧时间数据**。该文档覆盖 Animator/Constraint/Audio/Contact/Cloth/PhysBones，且在「后续计划」（第 200–213 行）明确将 Skinned mesh renderers、Lights 列为**未来未公布**测试项；ParticleSystem 亦无定量数据。

注意：前述 `ps_*` 定量（`ps_max_particles=50000` 等）属于**用户端限制配置（上限值）**，并非本组件帧时间数据 —— 已由 [[../../avatar/avatar-particle-system-limits.md|Avatar Particle System Limits]] 页承载，勿与帧时间混淆。

可用定性参考（非该源）：

- [FACT] 官方 [Avatar Optimizing Tips](https://docs.vrchat.com/docs/avatar-optimizing-tips)：粒子带 Light 每个算一个实时光（极贵）；大而透明的粒子比多个小而透明粒子差。
- [INFERENCE] 既有页 [[../performance-guide.md|Performance Guide]] 将「粒子效果」列入动态系统中高成本项（中-高，适度使用）。

## 社区佐证

- [COMMUNITY/SECONDARY] [VR Creators — Quest Compatibility Checklist](https://vrcreators.net/docs/vrchat/quest-compatibility-checklist)：Quest 上 avatar 粒子被 heavily limited、只能 VRChat mobile shaders、网格顶点/三角形预算很紧。
- [COMMUNITY/SECONDARY] [VR Creators — World Optimization Checklist](https://vrcreators.net/docs/vrchat/world-optimization-checklist)：常驻粒子特效需 justify 成本；移动端避免过度透明与频繁 active 特效。

## 相关页面

[[particlesystemforcefield.md|ParticleSystemForceField]] · [[light.md|Light]] · [[../../avatar/avatar-particle-system-limits.md|Avatar Particle System Limits]] · [[../whitelisted-world-components.md|Allowlisted World Components]] · [[../performance-guide.md|Performance Guide]] · [[../vrc-light-volumes.md|VRC Light Volumes]]

## 源清单

- Unity 2022.3 Manual — Particle System: https://docs.unity3d.com/2022.3/Documentation/Manual/class-ParticleSystem.html
- Unity 2022.3 Manual — Particle System Modules: https://docs.unity3d.com/2022.3/Documentation/Manual/ParticleSystemModules.html
- Unity Scripting API — ParticleSystem.MainModule: https://docs.unity3d.com/2021.1/Documentation/ScriptReference/ParticleSystem.MainModule.html
- Unity 2022.3 Manual — ParticleSystemForceField: https://docs.unity3d.com/2022.3/Documentation/Manual/class-ParticleSystemForceField.html
- VRChat 官方白名单: https://creators.vrchat.com/worlds/whitelisted-world-components/
- VRChat Android Content Limitations: https://creators.vrchat.com/platforms/android/quest-content-limitations
- VRChat Android Content Optimization: https://creators.vrchat.com/platforms/android/quest-content-optimization
- VRChat Avatar Particle System Limits (docs.vrchat.com): https://docs.vrchat.com/docs/avatar-particle-system-limits
- VRChat Avatar Optimizing Tips (docs.vrchat.com): https://docs.vrchat.com/docs/avatar-optimizing-tips
- 既有页（只读引用，不重复造节）：
  - `entities/avatar/avatar-particle-system-limits.md`（用户端粒子限制，含 penalty 公式推断）
  - `entities/world/performance-guide.md`（粒子=动态系统中高成本项）
  - `entities/avatar/avatar-dynamic-bone-limits.md`（config.json 同源机制）
- 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无粒子数据 → [UNKNOWN]）