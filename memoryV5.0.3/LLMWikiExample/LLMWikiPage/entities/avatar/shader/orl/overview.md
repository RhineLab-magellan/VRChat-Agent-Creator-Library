---
title: "ORL Shaders 概览"
category: avatar
subcategory: shader
knowledge_level: applied
status: active
source: 本地知识库整理
source_type: official
version: 1.1
upstream_version: "v7.3.1 (2026-07-11)"
last_review: 2026-11-10
confidence: Medium
tags:
  - avatar
  - shader
  - orl
  - liltoon
  - json
  - light
aliases:
  - 着色器
  - "ORL Shaders 概览"
related:
  - shader-list.md
  - configurable.md
  - ../../../../comparisons/orl-comparison.md
type: entity
created: 2026-06-21
sources: 本地知识库整理
updated: 2026-11-10
---
# ORL Shaders 概览


## 项目信息

| 项目 | 内容 |
|------|------|
| **名称** | orels Unity Shaders (ORL Shaders) |
| **发布时间** | 2024年（持续更新） |
| **最新功能** | Per-Module Channel Selectors + Toon Decal 混合模式 + `%Priority()` 抽屉函数（v7.3.0）；LTCGI 屏幕修复（v7.3.1）|

## 定位

orels Unity Shaders - 模块化 VRChat World 着色器套件。

> **注意**: ORL 主要面向 World 开发，Avatar 开发推荐使用 lilToon。

ORL Shaders 是 **VRChat World 开发专用** 的模块化着色器套件，核心优势在于：

1. **Configurable Shader 系统** - 可自由组合功能模块
2. **丰富的 VFX 特效库** - 护盾、激光、溶解等
3. **专业的 PBR 实现** - Parallax OM、GSAA、Mobile Tonemapping
4. **深度 VRChat 集成** - Shader Fallback、VRCLightVolumes、Camera View 控制


## 安装方式

### VCC 一键安装（推荐）

```
https://orels1.github.io/orels-Unity-Shaders/index.json
```

1. 打开 VRChat Creator Companion (VCC)
2. 进入 Project Management
3. 添加 Community Repository
4. 搜索并安装 "ORL Shaders"

### 手动安装

1. VCC → Project Management → Add Repository
2. 粘贴上述 URL
3. 在 Project Management 中添加 "ORL Shaders" 包

## Shader 分类总览

| 类别 | 数量 | 说明 |
|------|------|------|
| **Standard Shaders (PBR)** | 22 | 基础 PBR + 功能扩展 |
| **Toon Shaders** | 3 | 卡通渲染（v1/v2） |
| **Special Shaders** | 1 | 积雪覆盖 |
| **VFX Shaders** | 9 | 视觉效果特效 |
| **UI Shaders** | 5 | UI 专用着色器 |
| **Configurable** | 模块化 | 可组合的 Uber Shader |

### Standard Shaders (PBR) 列表

| Shader | 功能 |
|--------|------|
| Base Shader | PBR 核心，包含 Parallax/Details/VRChat Features |
| AudioLink | 音频响应（5种模式） |
| Color Randomisation | 颜色随机化 |
| Decals | 贴花系统 |
| Dissolve | 溶解效果 |
| Dither Fade | 抖动淡入淡出 |
| Foliage | 植被专用（SSS近似） |
| Glass | 玻璃材质 |
| Hotspotting | 热区效果 |
| Layered Material | 分层材质 |
| Layered Parallax | 分层视差 |
| LTCGI | LTCGI 光照集成 |
| AreaLit | 区域光照 |
| Neon Light | 霓虹灯光 |
| Puddles | 水坑/积水 |
| Pulse | 脉冲效果 |
| Tessellated Displacement | 细分位移 |
| Triplanar Effects | 三平面映射 |
| Vertex Animation | 顶点动画 |
| Vertical Fog | 垂直雾效 |
| Video Screen | 视频屏幕 |
| VRSL GI | VRSL 全局光照 |

### VFX Shaders 列表

| Shader | 功能 |
|--------|------|
| Clouds | 云效果 |
| Ghost Lines | 幽灵线效果 |
| Glitch Screen | 故障屏幕效果 |
| Patterns | 图案效果 |
| Shield | 能量护盾（双层噪声+深度融合） |
| Laser | 激光效果 |
| Holographic Parallax | 全息视差 |
| Cubemap Screen | 立方体贴图屏幕 |
| Block Fader | 块状淡入淡出 |

## 核心架构

### Configurable Shader 系统

ORL 的核心创新是 **Configurable Shader**，允许用户：

- 自由组合功能模块
- 基于已有 Shader 扩展（如：Vertex Animation + LTCGI）
- 减少材质变体数量，提升 Draw Call Batching

### Shader Generator

面向 Shader 开发者的代码生成系统：

- 声明式 Shader 定义 → 自动生成 HLSL
- 支持自定义模块（.orlsource）
- 支持自定义 Lighting Models

## 版本历史

| 版本 | 主要更新 |
|------|----------|
| 7.3.1（2026-07-11） | 修复 LTCGI 屏幕（LTCGI screens）工作不正常的问题 |
| 7.3.0（2026-07-06） | 代码清理 + 大量 bugfix；见下方「v7.3.0 变更要点」 |
| 7.2.0（2026-01-24） | 新增 Snow Coverage Shader |
| 7.x | 重构 Standard Shader，添加 VRCLightVolumes 支持 |
| 6.x | Configurable Shader 系统引入 |
| 5.x | Toon Shader v2 发布 |

> 版本节奏说明：v7.4.0 目前仍是 dev 预发布（Dev 1~4，2026-07-13 ~ 2026-10-15），**VPM 稳定渠道的分发版本是 v7.3.1**，不要把 dev 版本文档当稳定版写入项目。
> 证据：https://github.com/orels1/orels-Unity-Shaders/releases

### v7.3.0 变更要点（按对使用者的影响排序）

| 类别 | 变更 | 说明 |
|------|------|------|
| 新功能 | Specular / Occlusion Toon 模块增加 Channel Selector | 可直接从输入贴图的任意通道取值 |
| 新功能 | Toon Decal 多层混合模式 | 现在支持 Replace / Add / Multiply / Overlay（原只有 Replace） |
| 新功能 | Toon Emission 支持 UV Tile 遮罩、Rim Masking、Emission 贴图 Power 调节 | 便于做单通道贴图省 VRAM |
| 新功能 | Toon Rim Light 增加 Normal Map Influence 滑条 | 强法线贴图上做柔和边缘光 |
| 新功能 | `%Priority()` 抽屉函数 | 同名属性多模块定义时，由优先级决定 Inspector 显示哪一个 |
| 新功能 | `%Linear()` 抽屉函数 | 标记贴图必须为线性数据，接错时给出「Auto Fix」提示框 |
| 新功能 | `%SkipAlwaysIncludedBlocks()` 指令 | 在自定义 Lighting Model 里跳过基础模块 |
| 新功能 | `GLOBAL_uvArray` 全 UV 数组 | 统一传所有被请求的 UV 通道，可用下标取任意通道 |
| 新功能 | VRCFeatures 模块可设 Fallback Smoothness/Metallic/Occlusion | 改善 Avatar 走回退 Shader 时的观感 |
| 行为变化 | Toon Shader 的 Alpha Scale 范围改为 -4 ~ 4 | **小于 0 会反相 albedo 贴图**，旧材质若用了负值需重新检查 |
| 行为变化 | Shader 默认改用 `float2` UV 通道 | 需要 float4 时要显式用 `NEED_FLOAT4_UV` |
| 行为变化 | Toon Outlines 模块改用 `%OutlinePassModifiers` 定义 stencil 属性 | 自定义 stencil 时改这里的写法 |
| 优化 | Toon Shader 的 uv 处理改进，插值器数量减少 | 移动端/Quest 变体收益 |
| 优化 | AudioLink Effects 模块的 cbuffer 与 sampler 数量降低 | 不用该效果时释放 sampler |
| 修复 | SSR 模块在 Queue ≤ 2500 且开 Depth Write 时的问题 | 影响依赖深度的屏幕空间效果（如屏幕空间描边） |
| 修复 | Toon Outline pass 在 VRChat 镜子里 alpha 混合不正确 | — |
| 修复 | Toon 在 Meta Pass 开 Emission 时编译失败 | — |
| 修复 | Layered Material 第 3 层的 Metallic / Smoothness remap 失效 | — |
| 修复 | Texture3D 在 sampler 宏剥离时被改写为 Texture2D | — |
| 移除 | 多余的 sampler 定义 | 无功能影响 |

> 证据：https://github.com/orels1/orels-Unity-Shaders/releases/tag/v7.3.0 、 https://github.com/orels1/orels-Unity-Shaders/releases/tag/v7.3.1

## 迁移指南

| 迁移路径 | 说明 |
|----------|------|
| **v7.2.x → v7.3.x** | release note 未列 breaking change，可直接覆盖升级；但 Toon Shader 的 **Alpha Scale 负值语义变了**（负数现在会反相 albedo），旧材质里用过负值的要逐个检查 |
| v6 → v7 | 有专门迁移文档 |
| 旧版 → v6 | 有专门迁移文档 |
| 常规更新 | 直接覆盖即可 |

> 证据：https://github.com/orels1/orels-Unity-Shaders/releases/tag/v7.3.0 （Changes 节 "Alpha Scale now goes between -4 and 4 … Values below 0 invert the albedo map"；release note 中无 Breaking Changes 段）

## 获取帮助

- **文档站**: shaders.orels.sh
- **Discord**: discord.gg/orels1
- **GitHub Issues**: github.com/orels1/orels-Unity-Shaders/issues

## 相关文档

- [Shader List](shader-list.md) - 详细 Shader 列表
- [Configurable Shader](configurable.md) - 模块化系统详解
- [Comparison](../../../../comparisons/orl-comparison.md) - 与其他 Shader 对比
---

## 相关页面

[[shader-list.md]] · [[configurable.md]] · [[../../../../comparisons/orl-comparison.md]]
