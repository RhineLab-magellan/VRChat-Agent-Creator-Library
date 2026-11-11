---
title: 光照烘焙深度指南
category: world
subcategory: bakery
knowledge_level: applied
status: active
source: "本地知识库整理 + VRChat 2026.2.2p1 Release Notes + z3y/glim 官方项目资料"
source_type: community
version: 1.2
last_review: 2026-08-13
confidence: Medium
tags:
  - world
  - bakery
  - udonsharp
aliases:
  - 光照烘焙深度指南
  - light-baking-guide.md
related:
  - ../../../concepts/rules/performance-rules.md
  - index.md
  - ../examples/README.md
  - ../examples/detect-controller-collide.md
  - ../examples/image-loading.md
  - ../vrc-light-volumes.md
  - ../reflection-probes.md
type: entity
created: 2026-06-30
sources:
  - "本地知识库整理 + VRChat 2026.2.2p1 Release Notes"
  - https://raw.githubusercontent.com/z3y/glim/master/README.md
  - https://raw.githubusercontent.com/z3y/glim/master/unity/package.json
  - https://github.com/z3y/glim/blob/master/unity/Editor/Glim.cs
  - https://github.com/z3y/glim/blob/master/unity/Editor/Bake.cs
updated: 2026-08-13
---
# 光照烘焙深度指南

> 来源: VRCLibrary (Silent)

---

## 为什么要烘焙光照

| 原因 | 说明 |
|------|------|
| **性能** | 实时光源需要反复绘制接触的一切，烘焙后只是纹理贴图 |
| **光反射** | 现实世界光反射无处不在（地表反射 + 太阳直射），实时光源无法重现 |

> 大地图优化后光照贴图约 50-60MB

---

## Bakery 工具

### 概述

| 项目 | 说明 |
|------|------|
| **价格** | $55（定期打折） |
| **实时预览** | 额外 $35 |
| **GPU 要求** | 仅英伟达 GPU |
| **特点** | RTX 加速 + AI 降噪 |

### 优点

- 高质量光线追踪 + AI 降噪
- 比 Unity 内置快很多
- 打包更小的光照贴图（比 xatlas）
- 支持物理精确光照 + 风格化 AO 选项

### VRChat 注意事项

> ⚠️ Bakery 需要被列入白名单才能在 VRChat 中使用

**解决方案**：使用 [VRC-Bakery-Adapter](https://github.com/MerlinVR/VRC-Bakery-Adapter) 脚本转换 RNM/SH 方向光照贴图格式。

---

## 光源性能开销分析

### Draw Call Passes

| 光源类型 | 无阴影 | 有阴影 |
|----------|--------|--------|
| 天空盒 | 免费 | — |
| 方向光 | 免费 | 渲染 4 级阴影级联 + 深度缓冲 |
| 点光源 | +1 Pass | 渲染 6 面立方体贴图 |
| 聚光灯 | +1 Pass | +1 Pass（只需 1 个） |
| 区域光 | 无性能开销 | 仅烘焙后生效 |

> ⚠️ Unity 不使用遮挡剔除，阴影可能渲染对象最多 4-6 次

### VRChat 光源限制

VRChat Unity 只渲染 **8 个实时光源**作为完整额外 Pass，超出的按重要性优先级渲染。

---

## 光照贴图模式

### Baked Indirect

- 烘焙非直接光照到贴图
- 主光源单独添加（可选择是否包括）
- **性能开销高**，但视觉效果最好
- 适合：动态阴影的日光/月光

### Shadowmasks（阴影遮罩）

- 烘焙环境光 + 方向光阴影
- 保留镜面高光等实时优点
- 可与动态阴影结合
- 或使用遮蔽探针设为完全烘焙

### Subtractive（削减式）

- 烘焙所有光源
- 动态对象（如玩家）仍投射阴影
- **优化好**，但视觉效果稍差

---

## 光照贴图 UV 规范

> ⚠️ 光照贴图 UV 必须完全位于 0-1 范围内，不能重叠

### 7 条黄金规则

| 规则 | 说明 |
|------|------|
| **1. 不重叠 UV** | 重叠会导致光照贴图无法正确生成，出现黑色斑点 |
| **2. 保持 0-1 范围** | 超出范围无法正确生成 |
| **3. 岛屿间距足够** | 间距不足会导致阴影泄漏（Light Bleeding） |
| **4. 隐藏接缝** | 接缝可能非常明显 |
| **5. 与轴对齐** | 对角 UV 浪费性能且明显 |
| **6. 连接岛屿要谨慎** | 连接减少接缝，但分辨率不够会渗色 |
| **7. 边角增加分辨率** | 斜边区域留额外空间，避免光渗 |

### 额外建议

- 硬边缘模型使用斜面（Bevelled）改善光照贴图
- 斜面使光线平滑穿过，增加细节和自然外观
- Blender：获取现有 UV 并重新拆包作为光照贴图 UV

---

## 光照探针

### 原理

- 空间中的点，从各方向捕捉光照图像
- 足够多探针形成四面体 → 影响动态物体
- 性能开销极小（可使用数以万计探针）

### 自动放置工具

| 工具 | 价格 | 说明 |
|------|------|------|
| **Light Probes Volumes** | 免费 | 根据场景碰撞放置，使用方便 |
| **Magic Light Probes** | 付费 | 算法沿光照变化处放置，效果好但易出错 |
| **AutoProbe** | 付费（半价） | 均匀散射 + 优化去除不必要探针 |

---

## Bakery 使用要点

### 禁止事项

- ❌ 不要使用 Lighting 窗口的 **Generate Lighting** 按钮（会覆盖 Bakery）
- ❌ 不要打开 **Auto Generate** 选项
- ❌ 不要在反射探针上使用 **Bake** 按钮（除非自定义）

### 正确流程

```
1. 禁用 Unity 光源组件（除非是混合光源）
2. 添加 Bakery Sky Light（天空盒立方体贴图，用于环境光）
3. 添加 Bakery 方向光源（设置同主方向光）
4. 如需要，添加 Bakery 点光源
5. 使用 Show Checker 查看单个网格贴图分辨率
6. RTX 显卡 → 打开 RTX 模式
7. 使用法线贴图 → Directional Mode 设为 Dominant Direction
```

### 光照探针模式

| 模式 | 说明 |
|------|------|
| **L1（默认）** | 与贴图同时烘焙，快但无法捕捉特殊着色器 |
| **Legacy** | 单独烘焙，准确但慢 |

---

## Probe Ringing 问题

### 问题描述

高对比度照明区域，单方向强光可能导致极端阴影（Ring 现象）。

### 解决方案

| 方法 | 说明 |
|------|------|
| 使用 Bakery 标准着色器 | 启用 "non-linear SH" 选项 |
| 使用 Shadowmask 混合光源 | 将强方向光从探针移出 |
| 避免高对比度照明 | 根本上解决问题 |

---

## 🔄 VRChat 版本兼容修复 (2026.2.2p1+)

> **FACT** (2026.2.2p1 Build 1850):"Fixed Bakery lightmaps breaking in Build & Test mode, or some regular world load scenarios."

### 问题场景

| 场景 | 表现 | 修复状态 |
|------|------|----------|
| **Build & Test 模式** | Bakery 光照贴图**无法加载/显示异常** | ✅ 2026.2.2p1 修复 |
| **部分常规世界加载场景** | 烘焙贴图错位/不显示 | ✅ 2026.2.2p1 修复 |

### 创作者影响

| 影响项 | 说明 |
|--------|------|
| **无需代码改动** | 纯引擎层修复,场景/Prefab 不需修改 |
| **旧版客户端行为** | 2026.2.2 之前客户端仍可能触发问题 |
| **建议** | 发布前在最新客户端测试 Build & Test,确保贴图正确 |
| **无需重烘** | 现有 .exr / 贴图资产可继续使用 |

### 已知限制(2026.2.2p1 仍未完全解决)

> **【未确认】** 该修复可能未覆盖所有边缘场景:
> - 极特殊加载序列(动态加载 + 异步实例化)是否完全稳定**未确认**
> - Quest 平台 Bakery 兼容性仍受 GPU 限制(需 NVIDIA Kepler+ GPU 烘焙)

---

## Glim Lightmapper

### 项目定位与分发

`z3y/glim`（Glim Lightmapper）是社区开发的独立 Unity GPU 光照渲染器（lightmapper），基于 Vulkan，定位为 **Bakery 的开源替代品**，与 Unity 内置 Progressive Lightmapper、Bakery 三者并行。官方 Unity 包清单显示包名为 `io.github.z3y.glim`，观察到的包版本为 `0.5.2`，Unity 声明版本为 `2022.3`。支持 VPM 安装，也可通过 UPM 的 “Add package from disk” 选择包内 `package.json`。

> **[FACT]** 来源：官方 README、Unity package.json、用户确认定位（独立渲染器 / Bakery 开源替代 / 三者并行）

### Built-in / URP 支持

Glim 官方 README 明确声明支持 Unity Built-in Pipeline 与 URP；源码中还通过 `GraphicsSettings.currentRenderPipeline == null` 判断 Built-in 状态。

> **[FACT]** Glim 声明支持 Built-in/URP，源码存在 Built-in 检测分支。来源：Glim README、Glim.cs
> **[INFERENCE]** VRChat World 硬约束 Built-in Render Pipeline，因此 Glim 满足进入目标工程实测的管线前提。

### Editor-side 烘焙与 Unity 标准输出

Glim 的 Editor 程序集仅面向 Editor 平台，现有证据支持将其定位为编辑器侧烘焙工具。源码处理或写入 `LightingDataAsset`、`LightmapData`、`LightmapSettings.lightmaps` 和 `Lightmapping.lightingDataAsset`，并生成 Diffuse/Directional 光照贴图文件，接入 Unity 标准光照数据模型。

> **[FACT]** Editor asmdef 的 `includePlatforms` 仅为 Editor。来源：Editor asmdef、Bake.cs
> **[INFERENCE]** 当前证据未显示其作为 VRChat 客户端运行时烘焙插件；烘焙结果交给 Unity 标准光照数据模型。

### Vulkan/GPU 烘焙机要求

官方 README 列出 Windows/Linux 与 Nvidia/AMD GPU，并要求烘焙机器的 GPU 支持 `VK_KHR_ray_query`。这是**执行光照烘焙的编辑器机器要求**，不推导 VRChat 客户端平台兼容性。

> **[FACT]** 来源：Glim README

### 与 Unity 内置 Lightmapper、Bakery 的关系

- Glim 是独立光照渲染器，作为 Bakery 的开源替代，与 Unity 内置 Progressive Lightmapper、Bakery 并列可选。
- Bakery 的白名单要求、`VRC-Bakery-Adapter`、NVIDIA/OptiX 限制、Bakery 专用菜单流程等**不适用于** Glim，不得直接转移。
- Glim README 明确提到 `VRCLightVolumes baking`；VRCLightVolumes 是独立的 World 体积光照、烘焙与运行时 Shader 系统，Glim 与其存在烘焙关联，具体数据交互仍需按目标工程验证。参见 [[../vrc-light-volumes.md]]。
- Reflection Probe 烘焙属于 Unity World 光照数据验证范围。参见 [[../reflection-probes.md]]。

### VRChat 可用性

Glim 已确定在 VRChat 可用（用户确认；官方 README 亦含 VPM、VRChat、VRCLightVolumes baking 及 VRChat World 示例关联）。

> **[FACT]** Glim 可在 VRChat 使用（用户确认 + 官方 README 的 VRChat 生态证据）
> **[UNKNOWN]** 以下细节尚未逐项实测，不影响「VRChat 可用」的总体结论，但建议在目标工程中确认：

- Standard、lilToon、Poiyomi 等具体 Shader 的 Lightmap 采样表现；
- Non-Directional、Directional、Combined SH 等模式在目标 Shader 下的表现；
- PC Build & Test、正式客户端与 Android/Quest 的构建/加载差异；
- Glim 与 Bakery 同场景混用的数据覆盖行为。

---

## 相关文档

- `misc/maebbie.md` — Maebbie 烘焙光照贴图切换技术
- `rules/performance-rules.md` — World 性能约束