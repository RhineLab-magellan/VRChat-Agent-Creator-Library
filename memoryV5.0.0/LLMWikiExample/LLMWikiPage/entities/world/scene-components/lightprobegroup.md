---
title: "LightProbeGroup — 世界光照组件"
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
  - probe
  - performance
aliases:
  - LightProbeGroup
  - 光照探针组
  - 灯光探针组
related:
  - ../whitelisted-world-components.md
  - ../performance-guide.md
  - ../vrc-light-volumes.md
  - ../reflection-probes.md
  - light.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-LightProbeGroup.html | https://docs.unity3d.com/ScriptReference/LightProbeGroup.html | https://docs.unity3d.com/2022.3/Documentation/Manual/LightProbes.html | https://creators.vrchat.com/worlds/whitelisted-world-components/"
updated: 2026-08-17
---
# LightProbeGroup — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无本组件独立帧时间数据，详见性能小节）

---

## 核心功能（官方，[FACT]）

**要在场景中放置 Light Probes，必须在带 Light Probe Group 组件的 GameObject 上操作**。Light Probe Group 是一组采样点，在烘焙/运行时为**移动对象与静态 LOD 对象**提供插值光照（烘焙光照的补充）。

- 从 Component > Rendering > Light Probe Group 添加；建议加到新建空 GameObject。
- 关键属性/操作：**Edit Light Probes**（开启后只能编辑 probe）、**Show Wireframe**、**Remove Ringing**（2018.3+）、**Selected Probe Position**、Add Probe / Select All / Delete Selected / Duplicate Selected。
- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-LightProbeGroup.html

## 关键属性/行为

- **[FACT] Ringing（振铃）**：当 probe 周围光照差异巨大（一侧亮一侧暗）时会在背面产生过冲光斑。处理：开 Remove Ringing（降低精度/对比）、用障碍物遮挡、或避免把直射光烘进 Light Probes（用 Mixed lighting 只烘间接光）。
- **[FACT] 放置原则**：probe 是点不是 GameObject；默认 8 个构成立方体；**应形成 3D 体积**（至少 2 个竖直层，否则无法构成四面体插值）。
- **[FACT] 数据**：每个 probe 是 SPH（Spherical Harmonics L2）编码的全景 HDR 图像，存储为 **27 个浮点**。过密放置浪费内存，应在光照变化大处加密、平稳处稀疏（线性插值足够时稀疏）。
- **[FACT] 与 Enlighten Realtime GI 结合**：可为移动光源提供移动对象上的动态间接光；光照变化/遮挡多/移动灯区域需密探针。
- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-LightProbeGroup.html

## VRChat 特定限制/注意

**官方地位**：LightProbeGroup 在 VRChat World「Allowlisted World Components」官方白名单 Unity Components 列表中（[[../whitelisted-world-components.md|Allowlisted World Components]]，Nov 25, 2025 版本）。同列还有 LightProbeProxyVolume。
- 源：https://creators.vrchat.com/worlds/whitelisted-world-components/

**[FACT] Quest 推荐**：[[../performance-guide.md|Performance Guide]] 第 7 节——Quest 端 VRChat「推荐大量使用烘焙光照，使用 Light Probes 让 Avatar 和动态对象接收采样光照」。→ Light Probe Group 是让动态 Avatar 在烘焙世界里正确受光的核心手段。

**[INFERENCE] 与 VRC Light Volumes 替代关系**：社区 VRCLightVolumes 的 Regular Light Volumes 用以替换 Unity Light Probes（体素化逐像素、基于 SH L1），见 [[../vrc-light-volumes.md|VRC Light Volumes]]。创作者有两种路线：Unity Light Probe Group（官方原生）或 Light Volumes（第三方更高级替代）。

**[FACT] 创建/编辑注意**：Light Probes 不能烘进直射光（应只含间接光），振铃问题在 VRChat 世界（大对比环境）尤其明显；经验上在可步行区域与 Avatar 常处高度放置（[[../performance-guide.md|Performance Guide]] 烘焙检查清单：「Light Probes 在可步行和 Avatar 可见区域放置」）。

## 性能数据（unityvrchat 或 [UNKNOWN]）

`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）` **无 LightProbeGroup 独立帧时间数据** → **`[UNKNOWN]`，无该组件独立性能数据**。

相关邻近事实（[FACT]，**非本体数据**）：官方指出过密探针浪费内存（27 浮点/个），但无 VRChat 帧时间定量数据。世界光照整体建议见 [[../performance-guide.md|Performance Guide]] 与 [[../reflection-probes.md|Reflection Probes]]。

## 社区佐证

- [SECONDARY] VRCLightVolumes（REDSIM）以「体素化光照探针」替代 Unity Light Probes 为卖点，侧面印证 Unity Light Probes 的高密度/曲面材质局限——见 [[../vrc-light-volumes.md|VRC Light Volumes]]。

## 相关页面

[[../whitelisted-world-components.md|Allowlisted World Components]] · [[light.md|Light]] · [[../performance-guide.md|Performance Guide]] · [[../vrc-light-volumes.md|VRC Light Volumes]] · [[../reflection-probes.md|Reflection Probes]]

## 源清单

- Unity 2022.3 Manual — Light Probe Groups: https://docs.unity3d.com/2022.3/Documentation/Manual/class-LightProbeGroup.html
- Unity Scripting API — LightProbeGroup: https://docs.unity3d.com/ScriptReference/LightProbeGroup.html
- Unity 2022.3 Manual — Light Probes 总览: https://docs.unity3d.com/2022.3/Documentation/Manual/LightProbes.html
- VRChat 官方白名单: https://creators.vrchat.com/worlds/whitelisted-world-components/
- 既有页（只读引用）：`entities/world/performance-guide.md`（Quest Light Probes 建议/烘焙清单）、`entities/world/vrc-light-volumes.md`（Light Volumes 替代关系）