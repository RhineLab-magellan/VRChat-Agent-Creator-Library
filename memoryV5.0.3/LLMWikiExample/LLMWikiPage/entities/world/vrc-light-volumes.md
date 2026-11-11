---
title: "VRC Light Volumes"
category: world
knowledge_level: applied
status: active
source: 本地知识库整理
source_type: community
version: 2.0
upstream_version: "v3.0.1 (2026-10-14)"
last_review: 2026-11-10
confidence: High
tags:
  - world
  - shader
  - audio
  - light
  - avatar
aliases:
  - "VRC Light Volumes"
  - 光照系统
  - "Light Volumes"
  - VRCLightVolumes
related:
  - performance-guide.md
  - vvmw.md
  - creator-economy.md
  - reflection-probes.md
  - sdk-prefabs.md
type: entity
created: 2026-06-21
sources: 本地知识库整理
updated: 2026-11-10
---
# VRC Light Volumes

> 来源: https://github.com/REDSIM/VRCLightVolumes
> 当前版本: **v3.0.1** (2026-10-14)
> 版本核对: 本页工具版本已对照官方 Releases 核至 v3.0.1；v2.x 的字段级参数表尚未按 v3 逐项复核（见下方迁移小节）。
> 语言构成: C# 为主、ShaderLab/HLSL 次之（占比为 v2.x 期 GitHub 语言统计，v3 未复核）

---

## 概述

**Nextgen 基于体素的光照探针替代方案**，用于 VRChat World。

### 核心定位

| 组件 | 替代 | 说明 |
|---|---|---|
| **Regular Light Volumes** | Unity Light Probes | 体素化逐像素光照，基于 SH L1 |
| **Point Light Volumes** | Unity Realtime Lights | 动态点/聚光灯/面光源，参数化计算 |

> ⚠️ **Avatar 只能接收光照，不能投射 Light Volumes 光照**

### v3.0.1 当前版本 (2026-10-14) — 大版本断裂更新

> 来源: [VRCLightVolumes Releases](https://github.com/REDSIM/VRCLightVolumes/releases)（v3.0.0 发布于 2026-10-13，v3.0.1 修复版 2026-10-14）

**v2 → v3 是断裂性主版本更新**：Editor 侧辅助组件被移除，只剩 Udon 组件，且**不可轻易降级**。升级前先备份场景文件（只需备份 `.unity` 场景，材质/贴图不必）。

#### v3.0.0 主要变化

| 变化 | 说明 | 世界侧影响 |
|---|---|---|
| **EVSM 点光源阴影** | 模糊半影 + contact hardening | 阴影观感与参数语义改变，需重调 |
| **阴影三种烘焙方式** | Editor 烘焙 / 进游戏时烘焙（Bake In Game）/ 实时 | 可不在 bundle 里带 shadowmap，省包体与显存 |
| **旧 volumetric shadowmask 不再支持** | 编辑器与 shader 双双移除 | v2 世界升级后阴影遮罩全部失效，必须重烘 |
| **Area Light 大幅优化 + cookie 投影** | 面光源可像电视屏一样投射 cookie | 面光源成本显著下降，可替代部分 LTCGI 用法 |
| **点光源高光与光源尺寸分离** | specular 改用点光源 size | 旧参数化高光观感会变化，需目视比对 |
| **Froxel 光聚类** | 低端机大量点光源场景 FPS 提升 | Quest 场景收益明显，前提是 shader 用新版 inc |
| **Shader stripping** | 打包时剔除未使用的 VRCLV 特性 | 依赖新版 `LightVolumes.cginc` |
| **单组件模型** | 每类只剩一个组件（Manager / Light / Volume），Editor 与 Udon 组件合并 | 旧场景里的 editor helper 组件被删 |
| **新 Editor UI / gizmo / 调试视图** | 多选、灯光缩放、体素预览 | 上手流程变化，见新版文档 |

#### v3.0.1 修复（相对 v3.0.0）

- VRCLV Avatar debugger 改进
- LTCGI 支持修复
- `SetCustomMaterial()` 之后灯光闪烁修复
- Shader 支持列表更新

#### 迁移步骤（v2.x → v3.0.1）

1. **先备份场景**：至少备份 `Scenes/*.unity`；v3 移除 editor helper 组件后，降级无法自动还原。
2. **包与 shader 一起升**：升级 VRC Light Volumes 包后，把工程内所有自定义 shader 引用的 `LightVolumes.cginc` 换成 v3 版本（v3 起 LUT 采样方式与 v2 不同）。
3. **清理失效引用**：v3 只保留 Udon 组件；若场景/脚本/Inspector 引用了旧的 Editor 侧组件，升级后会丢失，需要重连。
4. **重做阴影**：v2 的 volumetric shadowmask 在 v3 编辑器与 shader 都不再支持，必须重新烘焙（Editor / Bake In Game / 实时三选一）。
5. **复查面光源与高光**：Area Light cookie 与 specular 尺寸语义都变了，升级后要重新比对观感与性能。
6. **小场景先验证再上正式图**：v3 有 shader stripping，若世界使用了小众 shader，先确认打包后功能仍在。

> ⚠️ 本页 v2.x 时期的字段级参数表（`LightVolumeManager` / `LightVolumeInstance` / `PointLightVolumeInstance` 字段与方法）**尚未按 v3 逐个复核**；v3 后组件结构已合并，引用前请在 v3 Inspector 里核对字段是否存在。

### v2.0.0 重大更新 (2025-07-20)

- Point Light Volumes 系统（128 动态光源）
- LUT/Cookie/Cubemap 投影
- 烘焙阴影遮罩
- 3D Custom RenderTexture 后处理
- Unity Debug 视图模式

---

## 系统架构

```
VRCLightVolumes System
├── Regular Light Volumes（烘焙系统）
│   ├── 替代 Unity Light Probes
│   ├── 基于 Spherical Harmonics L1
│   ├── 体素化存储（3D Texture Atlas）
│   ├── 支持 Additive 叠加模式
│   └── 支持烘焙阴影遮罩
│
├── Point Light Volumes（实时系统）
│   ├── Parametric / LUT / Custom 三种模式
│   ├── Point / Spot / Area Light 三种类型
│   ├── 最多 128 个同时可见
│   └── 支持烘焙阴影遮罩
│
└── Integrations
    ├── LightVolumeAudioLink
    ├── LightVolumeTVGI
    └── CustomRenderTexture 后处理
```

---

## Regular Light Volumes

### 核心原理 — Spherical Harmonics L1

| 分量 | 说明 | 数据类型 |
|---|---|---|
| **L0** | 环境色，无方向信息 | float3 |
| **L1 Red** | 红光平均方向，强度=向量长度 | float3 |
| **L1 Green** | 绿光平均方向 | float3 |
| **L1 Blue** | 蓝光平均方向 | float3 |

### 数据存储

```
单个 Light Volume → 3D Texture (Voxel Grid)
├── Texture0: L0, L1r.x, L1g.x, L1b.x, L1g.z
├── Texture1: L1r.y, L1g.y, L1b.y, L1b.z
└── Texture2 (Shadows): 烘焙阴影数据

多个 Volume → 3D Texture Atlas（打包合并）
```

### 计算公式

```glsl
FinalColor = L0 + dot(L1r, Normal) + dot(L1g, Normal) + dot(L1b, Normal)
```

### 限制

| 参数 | 值 | 说明 |
|---|---|---|
| 同时活跃 | **32** | 可动态开关切换更多 |
| 分辨率增长 | ×8 | 翻倍分辨率 → 体素数×8 |
| 权重优先级 | Weight 控制 | 重叠区域优先级 |

---

### 基础上手 / 世界侧最小配置流程（压缩版）

1. 在场景中放置 `LightVolumeManager` 与至少一个体积实例。
2. 先确定体积边界，再用 `Edit Bounds` / 相关可视化手段校准覆盖区域。
3. 从较低分辨率开始测试；只有在细节不足时再提高分辨率（因为分辨率翻倍会导致体素数约 ×8）。
4. 重叠体积先通过 `Weight` 规划优先级，再决定是否使用 Additive 模式。
5. 若是平台切换或大改光照结构后，重新检查烘焙与实例边界，不要直接沿用旧结果。

### 中文创作建议 / 实操提示（补充层）

- **先做边界，再做质量**：边界错了时，提高分辨率通常只会放大浪费。
- **Weight 先于“谁亮谁暗”的主观微调**：重叠区域先解决优先级，再调颜色 / 强度。
- **Additive 模式谨慎使用**：适合补局部效果，但也更容易叠出不自然结果。
- **World 侧最小验证要先跑通**：先确认基础接光正常，再加 AudioLink、TVGI、CRT 后处理等扩展集成。
- **示例世界 / 测试世界只作参考，不当成参数模板**：不同世界尺度、材质、光照目标差异很大。

## Point Light Volumes

### 三种 Light Shape

| 模式 | 说明 | 性能 |
|---|---|---|
| **Parametric** | 参数化计算衰减 | 最优 |
| **LUT** | 查找表定义衰减和形状 | 中等 |
| **Custom** | Cookie/Cubemap 投影 | 较高 |

### 三种 Light Type

| 类型 | 说明 | 性能 |
|---|---|---|
| **Point Light** | 球形衰减 | 基准 |
| **Spot Light** | 锥形衰减 | 基准 |
| **Area Light** | 矩形面光源 | **×8 成本** |

### 衰减公式（物理正确）

```
Attenuation = 1 / (LightSize² + Distance²)
FinalColor = Attenuation × Color × Intensity × LightSize²
```

> **Light Source Size** 影响整体能量：Intensity × Size² = 发射能量

### 限制

| 参数 | 值 | 说明 |
|---|---|---|
| 同时可见 | **128** | 可动态开关切换更多 |
| 实时阴影 | ❌ 不支持 | |
| 烘焙阴影 | ✅ 支持 | Shadow Mask |
| 每个 Volume 最多阴影光源 | **4** | |

---

## 核心组件

### LightVolumeManager

存储 3D atlas 和所有实例引用，控制更新。

| 字段 | 类型 | 说明 |
|---|---|---|
| `LightVolumeAtlas` | Texture | 合并的 SH 数据 atlas |
| `LightVolumeAtlasBase` | Texture3D | 后处理链基础（运行时未用） |
| `AtlasPostProcessors` | CustomRenderTexture[] | 后处理 CRT 数组 |
| `LightProbesBlending` | bool | 体积外回退到 light probes |
| `SharpBounds` | bool | 禁用平滑混合（提升性能） |
| `AutoUpdateVolumes` | bool | 运行时自动更新位置/旋转/缩放 |
| `AdditiveMaxOverdraw` | int | 限制叠加体积最大过绘数 |
| `ForceSceneLighting` | bool | 禁用亮度限制（现代 shader） |
| `LightsBrightnessCutoff` | float | 最小亮度阈值，高值=高性能 |
| `LightVolumeInstances` | LightVolumeInstance[] | 所有实例（按权重降序） |
| `CustomTextures` | Texture | LUT/Cookie/Cubemap 数组 |
| `CubemapsCount` | int | Cubemap 数量（6 faces/个） |
| `IsRangeDirty` | bool | 点光源范围需重新计算 |

| 方法 | 说明 |
|---|---|
| `void UpdateVolumes()` | 手动更新所有体积 shader 参数 |

### LightVolumeInstance

单个体积配置。

| 字段 | 类型 | 说明 |
|---|---|---|
| `Color` | Color | 运行时颜色（可分 R/G/B） |
| `Intensity` | float | 亮度 |
| `IsDynamic` | bool | 是否可移动 |
| `IsAdditive` | bool | 叠加模式（投射到光贴图） |
| `InvBakedRotation` | Quaternion | 烘焙姿态的逆旋转 |
| `BoundsUvwMin0/Max0` | Vector4 | Texture0 在 atlas 中的边界 |
| `BoundsUvwMin1/Max1` | Vector4 | Texture1 在 atlas 中的边界 |
| `BoundsUvwMin2/Max2` | Vector4 | Texture2 在 atlas 中的边界 |
| `InvLocalEdgeSmoothing` | Vector4 | 边缘平滑参数 |
| `InvWorldMatrix` | Matrix4x4 | 世界矩阵逆（UVW 计算） |
| `RelativeRotationRow0/1` | Vector3 | 相对旋转矩阵行 |
| `IsRotated` | bool | 是否有相对旋转 |
| `BakeOcclusion` | bool | 是否有烘焙阴影 |
| `IsInitialized` | bool | 是否已加入 Manager |
| `LightVolumeManager` | LightVolumeManager | 引用（运行时初始化用） |

| 方法 | 说明 |
|---|---|
| `void SetSmoothBlending(float radius)` | 计算边缘平滑 |
| `void UpdateTransform()` | 更新矩阵和旋转行 |

### PointLightVolumeInstance

点光源配置。

| 字段 | 类型 | 说明 |
|---|---|---|
| `Color` | Color | 光源颜色 |
| `Intensity` | float | 亮度 |
| `IsDynamic` | bool | 是否可移动 |
| `PositionData` | Vector4 | XYZ=位置, W=光源尺寸² |
| `DirectionData` | Vector4 | XYZ=方向, W=锥形衰减 |
| `CustomID` | float | 0=Parametric, 正=LUT, 负=Texture |
| `Angle` | float | 聚光灯半角（弧度） |
| `AngleData` | float | Cos(Angle) 或 tan(Angle) |
| `ShadowmaskIndex` | sbyte | 阴影遮罩通道 (-1=无) |
| `IsInitialized` | bool | 是否已加入 Manager |
| `SquaredRange` | float | 剔除距离² |
| `SquaredScale` | float | 平均缩放² |
| `IsRangeDirty` | bool | 范围需重新计算 |

| 方法 | 说明 |
|---|---|
| `bool IsSpotLight()` | 是否聚光灯 |
| `bool IsPointLight()` | 是否点光源 |
| `bool IsAreaLight()` | 是否面光源 |
| `bool IsCustomTexture()` | 是否自定义纹理 |
| `bool IsLut()` | 是否 LUT 模式 |
| `bool IsParametric()` | 是否参数化模式 |
| `void SetLightSourceSize(float size)` | 设置光源尺寸 |
| `void SetLut(int id)` | 设置 LUT ID |
| `void SetCustomTexture(int id)` | 设置 Cubemap/Cookie ID |
| `void SetParametric()` | 设置为参数化模式 |
| `void SetPointLight()` | 设置为点光源 |
| `void SetSpotLight(float angleDeg)` | 设置为聚光灯 |
| `void SetAreaLight()` | 设置为面光源 |
| `void SetColor(Color color)` | 设置颜色（自动标记 IsRangeDirty） |
| `void SetIntensity(float intensity)` | 设置强度（自动标记 IsRangeDirty） |
| `void UpdateTransform()` | 更新数据 |
| `void UpdateRange()` | 重新计算剔除距离 |

---

## 内置 Udon 脚本

### LightVolumeAudioLink

| 参数 | 类型 | 说明 |
|---|---|---|
| `AudioLink` | AudioLink | AudioLink 管理器引用 |
| `AudioBand` | enum | Bass/LowMid/HighMid/Treble |
| `Delay` | int | 采样延迟 (0-127) |
| `SmoothingEnabled` | bool | 启用平滑算法 |
| `Smoothing` | float | 平滑度 (0-1) |
| `ColorMode` | enum | Auto/Override Color |
| `Color` | Color | Override 模式颜色 |
| `TargetLightVolumes` | LightVolumeInstance[] | 目标体积 |
| `TargetPointLightVolumes` | PointLightVolumeInstance[] | 目标点光源 |
| `TargetMeshRenderers` | MeshRenderer[] | 目标渲染器 |
| `MaterialsIntensity` | float | 材质亮度乘数 |

> ⚠️ 目标 Mesh Renderer 的材质必须启用 emission，且 shader 包含 `_EmissionColor`

### LightVolumeTVGI

| 参数 | 类型 | 说明 |
|---|---|---|
| `TargetRenderTexture` | RenderTexture | 视频播放器 RenderTexture |
| `AntiFlickering` | bool | 启用平滑算法 |
| `TargetLightVolumes` | LightVolumeInstance[] | 目标体积 |
| `TargetPointLightVolumes` | PointLightVolumeInstance[] | 目标点光源 |

> ⚠️ Target RenderTexture 必须启用 Mip Maps

---

## Shader 集成

### 必需文件

```hlsl
#include "UnityCG.cginc"      // 必须在 LightVolumes.cginc 之前
#include "LightVolumes.cginc" // Light Volumes 函数
```

### 核心函数

| 函数 | 说明 |
|---|---|
| `LightVolumeSH(worldPos, out L0, out L1r, L1g, L1b)` | 采样 SH（自动 fallback 到 Unity Probes） |
| `LightVolumeSH_L0(worldPos)` | 仅采样 L0（轻量，用于粒子/雾） |
| `LightVolumeAdditiveSH(worldPos, out L0, L1r, L1g, L1b)` | 仅采样 Additive 体积（用于光贴图） |
| `LightVolumeAdditiveSH_L0(worldPos)` | 仅采样 Additive L0 |
| `LightVolumeEvaluate(worldNormal, L0, L1r, L1g, L1b)` | 计算最终光照颜色 |
| `LightVolumeSpecular(albedo, smooth, metal, normal, viewDir, ...)` | 多色高光（Avatar 推荐） |
| `LightVolumeSpecularDominant(albedo, smooth, metal, normal, viewDir, ...)` | 单色高光（静态 PBR） |
| `LightVolumesEnabled()` | 返回 0/1 标识 |
| `LightVolumesVersion()` | 返回版本号（0=无） |

### 基本集成流程

```hlsl
// 1. 采样 SH
float3 L0, L1r, L1g, L1b;
LightVolumeSH(i.worldPos, L0, L1r, L1g, L1b);

// 2. 计算光照
float3 lighting = LightVolumeEvaluate(i.worldNormal, L0, L1r, L1g, L1b);

// 3. 应用到最终颜色
float3 finalColor = albedo * lighting + specular;
```

### Additive 体积集成（光贴图）

```hlsl
// 在光贴图采样处添加
float3 L0, L1r, L1g, L1b;
LightVolumeAdditiveSH(i.worldPos, L0, L1r, L1g, L1b);
float3 additiveLighting = LightVolumeEvaluate(i.worldNormal, L0, L1r, L1g, L1b);
finalColor += albedo * additiveLighting;
```

---

## 兼容 Shaders

> 下表为 v2.x 期整理；v3.0.0 起 LUT 采样方式与 `LightVolumes.cginc` 有破坏性变化、v3.0.1 也更新过 shader 支持列表。迁移时请以 v3 官方支持列表 + 实际编译结果为准。

| Shader | 支持版本 |
|---|---|
| **lilToon** | 2.4.0+（v3 门槛；2.0.0+ 为 v2 时代下限） |
| **Poiyomi Toon** | v.9.2.67+ |
| **UnlitWF** | UnlitWF_Shader_20261101+（v3 门槛；2025/08/03 为 v2 时代下限） |
| **Graphlit** | v.2.0.1+ |
| **Filamented** | Jul 2025+ |
| **Silent Cel Shading** | Jul 2025+ |
| **Silent Clear Water** | Jul 2025+ |
| **Silent Crispy Foliage** | Jul 2025+ |
| **Orels Unity Shaders** | v7.0.0+ |
| **Mochies Unity Shaders** | v1.62.3+ |
| **RealToon** | v.5.0.13+ |
| **Xiexe's Unity Shaders** | v3.7.0+ |
| **Warren's Fast Fur** | v5.1.0+ |
| **ACLS Shader** | v.2.31+ |
| **Quantum Shader** | Jul 2025+ |
| **GeneLit** | v.1.0.8+ |

> 上述两行（lilToon / UnlitWF）的版本门槛为 2026-11-10 依各自 CHANGELOG 修正：VRCLV 3.0.0 是断裂大版本，v2 时代的兼容下限在 v3 下不再成立。
> 依据：lilToon CHANGELOG `2.4.0 — Changed: Minimal VRCLV 3.0 support`（https://github.com/lilxyzw/lilToon/blob/master/Assets/lilToon/CHANGELOG.md ）；UnlitWF CHANGELOG `UnlitWF_Shader_20261101 — Changed: VRC Light Volumes 3.0.1 に対応しました`（https://github.com/whiteflare/Unlit_WF_ShaderSuite/releases/tag/UnlitWF_Shader_20261101 ）。其余 Shader 行为未复核，保持原值。

---

## 最佳实践

### Regular Light Volumes

| 场景 | 建议 |
|---|---|
| 静态小道具 | 体素化无接缝，优于高分辨率光贴图 |
| 动态批处理 | 禁用 Light Probes 的同材质物体可批处理 |
| 粒子雾效果 | 结合体素 + 粒子 |
| 房间切换 | 运行时切换两个 Light Volume |
| 重叠区域 | 体积重叠 0.25m，Weight 控制优先级 |
| 边缘平滑 | 用 `Smooth Blending` 参数控制 |

### Point Light Volumes

| 场景 | 建议 |
|---|---|
| 动态点光源 | 用 Point Light，×4 性能优于 Additive Volume |
| 手电筒 | 用 Spot Light + Parametric |
| 可移动软盒 | 用 Area Light（注意 ×8 成本） |
| 迪斯科球 | 用 Point Light + Cubemap |
| Cookie 投影 | 用 Spot Light + Cookie |
| 灯光闪烁 | 用 AudioLink 控制 Intensity |

### 性能优化

- 动态关闭不需要的体积
- `IsDynamic=false` 提升性能
- `SharpBounds=true` 提升性能
- 限制 `AdditiveMaxOverdraw` 防止过绘
- `LightsBrightnessCutoff` 调高提升性能（物理正确性降低）

---

## 安装方式

> 当前包版本 **v3.0.1**；v3 起包内只含 Udon 组件，升级方式仍是 VCC / UPM。

### VCC (推荐)

1. 访问 https://redsim.github.io/vpmlisting/
2. 点击 "Add to VCC"
3. Package Manager → Samples → Import Examples

### Unity Package Manager

```
Window > Package Manager > + > Add from git URL
https://github.com/REDSIM/VRCLightVolumes/tree/main/Packages/red.sim.lightvolumes
```

---

## 测试世界

| 世界 | World ID |
|---|---|
| Japanese Alley | `wrld_af756ca8-30ee-41a4-b304-2207ebf79db9` |
| Light Volumes x AudioLink x FakeLTCGI | `wrld_ba751467-ca25-4734-91b3-7e503fc171f3` |
| 2000s Classroom | `wrld_f6445b27-037d-4926-b51f-d79ada716b31` |
| Concrete Oasis | `wrld_3641b8d9-04da-4ee4-8b06-966ca097b1a3` |

---

## 版本历史

| 版本 | 日期 | 重大更新 |
|---|---|---|
| **v.3.0.1** | 2026-10-14 | **当前版本**：Avatar debugger、LTCGI、SetCustomMaterial 闪烁修复 |
| **v.3.0.0** | 2026-10-13 | **断裂大版本**：EVSM 点光源阴影、Bake In Game、Area Light cookie、Froxel 聚类、shader stripping、单组件模型；旧 shadowmask 移除 |
| v.2.1.3 | 2025-11-17 | v2 最后版本 |
| v.2.1.2 | 2025-09-20 | Shadow Mask 修复 |
| v.2.1.0 | 2025-09 | 体素降采样、进度条、LUT 改进 |
| **v.2.0.0** | 2025-07-20 | **Point Light Volumes 系统、阴影烘焙、后处理** |

---

## 相关工具

| 工具 | 说明 |
|---|---|
| Bakery | 高质量烘焙（推荐配合 Light Volumes） |
| AudioLink | 音频可视化（可与 Light Volumes 联动） |
| LTCGI | 类似 GI 系统（不支持屏幕反射） |
| lilToon | 最流行 Avatar Shader（支持 Light Volumes） |
| Poiyomi | 功能丰富 Shader（支持 Light Volumes） |