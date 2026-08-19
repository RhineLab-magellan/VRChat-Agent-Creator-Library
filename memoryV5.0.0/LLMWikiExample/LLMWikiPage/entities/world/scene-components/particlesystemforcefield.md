---
title: "ParticleSystemForceField — 世界粒子力场组件"
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
  - force-field
  - performance
aliases:
  - ParticleSystemForceField
  - 粒子系统力场
related:
  - ../whitelisted-world-components.md
  - particlesystem.md
  - light.md
  - ../../avatar/avatar-particle-system-limits.md
  - ../performance-guide.md
  - ../vrc-light-volumes.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-ParticleSystemForceField.html | https://creators.vrchat.com/worlds/whitelisted-world-components/ | https://feedback.vrchat.com/open-beta/p/whitelist-particlesystemforcefield-component-for-avatarworld-interaction"
updated: 2026-08-17
---
# ParticleSystemForceField — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无本组件独立帧时间数据，详见性能小节）

---

## 核心功能（官方，[FACT]）

[FACT] [Unity 2022.3 Manual — Particle System Force Field](https://docs.unity3d.com/2022.3/Documentation/Manual/class-ParticleSystemForceField.html)：**Particle System Force Field 组件向 Particle System 中的粒子施加力**。要让它生效，必须在 ParticleSystem 上启用 **External Forces Module**，再指定 Layer Mask 或具体的 Force Field 组件。**所有力都在力场的局部空间施加**（旋转 Transform 会影响方向/旋转属性）。

- [FACT] 引入于 Unity **2018.3**（该页 NewIn20183）。

## 关键属性/行为

| 属性 | 说明（[FACT]） |
|---|---|
| **Shape** | 影响区域形状；Start Range / End Range 定义作用范围内外边界。 |
| **Direction X/Y/Z** | 沿轴的线性力。 |
| **Gravity（Strength / Gravity Focus）** | 把粒子吸向形状内焦点（0=中心，1=外边缘）。 |
| **Rotation（Speed / Attraction / Rotation Randomness）** | 围绕力场中心的涡流推进。 |
| **Drag（Strength / Multiply Drag by Size / Multiply Drag by Velocity）** | 减速效果。 |
| **Vector Field（Volume Texture / Speed / Attraction）** | 用 3D 矢量场纹理（如 Houdini 生成）定向驱动力。 |

## VRChat 特定限制/注意

**[FACT] 白名单地位**：ParticleSystemForceField 在 VRChat World「Allowlisted World Components」官方白名单的 Unity Components 列表中（[官方白名单页](https://creators.vrchat.com/worlds/whitelisted-world-components/)，Nov 25, 2025）。详见 [[../whitelisted-world-components.md|Allowlisted World Components]]。

**[FACT] World 已入白名单，Avatar 未入**：[VRChat 官方反馈（Head of Community Tupper 确认）](https://feedback.vrchat.com/open-beta/p/whitelist-particlesystemforcefield-component-for-avatarworld-interaction)：**自 build 904 起已加入 SDK3 World 白名单**（SDK2 世界无权访问）；**VRChat 无计划加入 Avatar 白名单**。→ 用法：在世界中放置 ForceField 影响世界粒子；不要放在 Avatar 上依赖其影响他人粒子。

**[FACT] 影响屏蔽**：粒子系统的 External Forces 模块有 Influence Mask（Layer Mask），默认 Everything；世界创作者可移除 Player / MirrorReflection / PlayerLocal 层，避免玩家/镜像影响粒子（同源 VRChat 官方反馈 Tupper 确认）。

[INFERENCE] **性能叠加**：力场为粒子模拟增加每帧计算量；配合大量粒子时叠加粒子本身的 CPU 模拟成本，Quest 端应谨慎 —— Quest 粒子本已严格受限，见 [[particlesystem.md|ParticleSystem]] 与 [[../../avatar/avatar-particle-system-limits.md|Avatar Particle System Limits]]。

## 性能数据（unityvrchat 或 [UNKNOWN]）

[UNKNOWN] — `[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）` 未收录 ParticleSystemForceField 帧时间数据（该源无 ForceField 测试）。

相关上下文（非本体数据）：粒子数量与碰撞/trails 惩罚由[[../../avatar/avatar-particle-system-limits.md|Avatar Particle System Limits]] 页的 `ps_*` 用户端限制配置量化（`ps_max_particles=50000`、`ps_trails_penalty=10` 等），属于上限配置而非帧时间；力场本身的量化帧成本无官方/基准数据。

## 社区佐证

- [COMMUNITY/SECONDARY] [VRChat 官方反馈讨论](https://feedback.vrchat.com/open-beta/p/whitelist-particlesystemforcefield-component-for-avatarworld-interaction)（含 VRChat 官方确认 world 白名单/build 904、不进 avatar 白名单、Influence Mask 用法）。
- [COMMUNITY/SECONDARY] 与粒子交互相关：想让艾娃/肢体（结合 IKFollower）发射力场驱动的 30dof 交互暂不可行（因不入 Avatar 白名单）。

## 相关页面

[[particlesystem.md|ParticleSystem]] · [[light.md|Light]] · [[../../avatar/avatar-particle-system-limits.md|Avatar Particle System Limits]] · [[../whitelisted-world-components.md|Allowlisted World Components]] · [[../performance-guide.md|Performance Guide]] · [[../vrc-light-volumes.md|VRC Light Volumes]]

## 源清单

- Unity 2022.3 Manual — Particle System Force Field: https://docs.unity3d.com/2022.3/Documentation/Manual/class-ParticleSystemForceField.html
- Unity Scripting API — ParticleSystemForceField: https://docs.unity3d.com/ScriptReference/ParticleSystemForceField.html
- VRChat 官方白名单: https://creators.vrchat.com/worlds/whitelisted-world-components/
- VRChat 官方反馈（Head of Community Tupper 确认 build 904 / Avatar 不入白名单 / Influence Mask）: https://feedback.vrchat.com/open-beta/p/whitelist-particlesystemforcefield-component-for-avatarworld-interaction
- 关联既有页（只读）：`entities/avatar/avatar-particle-system-limits.md`（粒子限制背景）