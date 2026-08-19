---
title: "Avatar 粒子系统创建工作流"
category: avatar
knowledge_level: applied
status: active
source: "VRCD 视频蒸馏产物 (video_8) + Unity Particle System 官方 + VRChat Limits 官方"
source_type: community
version: 1.0
last_review: 2026-07-04
confidence: Medium
tags:
  - avatar
  - particle-system
  - vrc-particle-limits
  - workflow
  - shader
aliases:
  - "粒子系统创建工作流"
  - avatar-particle-system-creation.md
  - KP04
related:
  - avatar-particle-system-limits.md
  - optimization-guide.md
  - modular-avatar.md
  - bone-binding-workflow.md
type: entity
created: 2026-07-04
sources: "VRCD 视频蒸馏产物 (video_8) + Unity Particle System 官方 + VRChat Limits 官方"
updated: 2026-07-04
---
# Avatar 粒子系统创建工作流

> 本文档提供 Avatar 粒子系统(Particle System)的"创建端"完整工作流
> 来源:VRCD 视频蒸馏(video_8, 2026-07-04) + Unity 官方 + VRChat 官方
> 配套:本文是"创建端",`avatar-particle-system-limits.md` 是"限制端"

## §1 概述

### §1.1 粒子系统在 Avatar 中的作用

- 眼睛流星特效(本文示例)
- 脚下光环
- 背后光环
- 法术 / 技能特效
- 环境氛围(光斑、萤火虫等)

### §1.2 文档范围(创建端 vs 限制端)

| 文档 | 视角 | 内容 |
|------|------|------|
| **本文(创建端)** | 玩家/创作者 | 怎么创建一个 Avatar 粒子系统 |
| `avatar-particle-system-limits.md`(限制端) | 用户/平台 | 平台对粒子系统的硬性限制 |

### §1.3 必备前提

- 已读 `avatar-particle-system-limits.md` 了解平台限制
- 熟悉 Unity Particle System 基础
- 已配置好 Shader 包(lilToon / Poiyomi 等)

## §2 粒子纹理准备

### §2.1 尺寸

- **Canvas 尺寸**:512×512
- **理由**:平衡质量与性能

### §2.2 配色

- 🔴 **黑底 + 白图案**(**不是**白底)
- 原因:配合 §3 Additive Rendering Mode
- 白底 + Additive = 过曝 / 全白

### §2.3 格式

- **PNG**(支持透明通道)
- 不要用 JPG(会丢失透明)

### §2.4 透明通道设置

Unity 导入设置:
- Alpha Source: Input Texture Alpha
- ✅ Alpha is Transparency

## §3 粒子材质设置(关键配方)

### §3.1 Shader

**`Particles > Standard Unlit`**

理由:Unity 官方粒子专用 Shader,性能最优

### §3.2 Rendering Mode

**`Additive`(自发光叠加)**

> 来源:Unity 官方文档
> "Additive: Adds the background and final particle color together. This is useful for glow effects, like those you might use for fire or magic spells."

### §3.3 Two Sided

**勾选 Two Sided(双面渲染)**

理由:粒子朝向不定时,避免"消失面"

### §3.4 颜色

- **Maps 调色板**:调整贴图颜色
- **Intensity**:调整自发光强度

## §4 Particle System Main Module 配置

### §4.1 参数总表(视频参考值)

| 参数 | 值(参考) | 说明 |
|------|---------|------|
| Duration | 1 | 持续时间(秒) |
| Start Lifetime | 3.5 | 粒子生命周期(秒) |
| Start Speed | 0.01 | 初始速度 |
| Start Size | 0.0075 | 初始大小 |
| Simulation Speed | 0.8 | 仿真速度 |
| Scaling Mode | Hierarchy | 缩放模式 |
| Max Particles | 15 | 数量限制 |

> ⚠️ **数值说明**:以上数值为 video_8 视频参考值,实际应根据场景调整
> 建议先复制本文档的"基线配置",再根据视觉调试

### §4.2 关键参数解读

**Duration = 1**
- 粒子系统的"激活时长"(秒)
- 配合 Looping 决定持续发射

**Start Lifetime = 3.5**
- 单个粒子从发射到消失的时间
- 视频为"眼睛流星"的悬停感设计

**Start Speed = 0.01**
- 极低速度
- 配合 Shape(§5.2)实现"聚集感"

**Start Size = 0.0075**
- 极小尺寸
- 适合"眼睛/面部"小范围特效

**Max Particles = 15**
- 极低数量
- VRChat 限制端:总上限 50000,单系统 50000
- 视频示例为低消耗场景

## §5 附加模块配置

### §5.1 Emission

```
Rate Over Time = 5
```

每秒发射 5 个粒子(配合 Max Particles = 15,平均 3 秒发射一轮)

### §5.2 Shape

```
Angle = 10
Radius = 0.005
```

- Angle 10:窄锥形发射
- Radius 0.005:极小半径(几乎点发射)

### §5.3 Velocity Over Lifetime

```
Linear Y = -0.0005
Speed Modifier = 0.8
```

- Y 轴缓慢向下
- 整体速度衰减

### §5.4 Color Over Lifetime

- 4 个关键帧渐变
- 首尾 Alpha = 0(淡入淡出)
- 中间渐变到目标色

### §5.5 Size Over Lifetime

- 模式:`Random Between Two Curves`
- 两条曲线,粒子大小随机分布在曲线之间

### §5.6 Size By Speed

- 模式:`Random Between Two Constants`
- 范围:0.25 ~ 0.5
- 速度影响大小的随机范围

### §5.7 Renderer

```
Render Mode = Mesh
Meshes = Quad
Material = 拖入 §3 配方的材质
```

### §5.8 Custom Vertex Streams

**保留项**:
- Position
- Color
- UV0
- Animation Frame

> 这是 Unity Particle 高级特性,新手可跳过

## §6 Parent Object 设计(防双重发射)

### §6.1 设计目的

- 避免双重发射(双眼分别发射)
- 单一 Bone Proxy 跟随 Head 骨

### §6.2 操作步骤

```
Step 1: 创建空 GameObject(作为双眼效果的父对象)
Step 2: Add Component → Particle System
Step 3: **所有模块 uncheck**(不实际发射,只作为父级)
Step 4: 父对象加 MA Bone Proxy
Step 5: Target = Head 骨
```

### §6.3 单一 Bone Proxy 原则

> 双眼效果共享一个父对象 + 一个 Bone Proxy
> 避免每只眼睛单独挂 Bone Proxy 导致性能浪费

## §7 镜像到双眼

### §7.1 操作流程

```
Step 1: Ctrl+D 复制父对象
Step 2: 改 X position 为负(镜像到另一侧)
Step 3: 改 Y rotation 为负(朝向镜像)
```

### §7.2 原理

镜面对称:
- X 位置取负 = 水平镜像
- Y 旋转取负 = 角度镜像

## §8 挂接到眼睛流程

### §8.1 VRC Avatar Descriptor 路径

```
Avatar → VRC Avatar Descriptor → Eye Look → Eyes → Transforms(眼锚)
```

### §8.2 完整操作

```
Step 1: 选中 VRC Avatar Descriptor
Step 2: 找到 Eye Look 部分
Step 3: 展开 Eyes,确认 Transforms(眼锚)已配置
Step 4: 拖入粒子系统 Prefab
Step 5: Reset Transform(归零位置)
Step 6: 调整位置到眼睛周围
Step 7: Bone Proxy 跟随 Head 骨
```

## §9 关键限制(交叉引用)

### §9.1 VRChat 用户端限制

详见 `avatar-particle-system-limits.md`:

| 限制项 | 默认值 | Quest |
|--------|--------|-------|
| Max Particles | 50000 | 50000 |
| Max Systems | 200 | 200 |
| Max Emission | 5000 | 5000 |
| **Quest 强制启用** | — | **不可禁用** |
| **PlayerLocal 碰撞** | 支持 | 限制 |

### §9.2 Quest 端行为

- 默认启用且**不可禁用**
- 超出会自动降低质量
- 详见 `avatar-particle-system-limits.md` §Quest 行为

### §9.3 性能优化检查清单

- [ ] 单个粒子系统粒子数 < 50000
- [ ] 总粒子系统数 < 200
- [ ] 总 Emission Rate < 5000
- [ ] 网格多边形 < 60000
- [ ] 配合 Optimization Guide

## §10 实战示例

### §10.1 示例 1:眼睛流星特效(参考 video_8)

详见视频源,本文 §2-§8 即此示例的完整还原

### §10.2 示例 2:脚下光环

参数调整建议:
- Start Size: 0.05~0.1(更大)
- Max Particles: 30~50
- Shape: Cone, Angle 90(向下发射)
- Color: 暖色调

### §10.3 示例 3:背后光环

参数调整建议:
- Start Size: 0.1~0.3
- Max Particles: 50~100
- Shape: Hemisphere(半球)
- Velocity: 向上飘

### §10.4 性能优化检查清单

- [ ] 优先用 Mesh = Quad(比默认 Plane 轻)
- [ ] 减少 Max Particles 数量
- [ ] 关闭不必要的 Custom Vertex Streams
- [ ] 启用 Prewarmed(避免初始跳变)

## §11 关键帧核对记录

> 本节为视频数值核对结果(待用户 review)

### §11.1 视频参考值 vs 实测值

| 参数 | 视频值 | 实测建议 | 通用性 |
|------|--------|---------|--------|
| Start Lifetime | 3.5 | 视场景 | 中 |
| Start Speed | 0.01 | 视场景 | 中 |
| Start Size | 0.0075 | 视场景 | 高(小尺寸) |
| Max Particles | 15 | 视场景 | 高(低消耗) |
| Rate Over Time | 5 | 视场景 | 中 |
| Angle | 10 | 视场景 | 中 |
| Radius | 0.005 | 视场景 | 高(点发射) |

### §11.2 通用性评估

- **Start Size 0.0075**:适合眼睛/面部小范围特效,通用性高
- **Max Particles 15**:低消耗参考,通用性高
- **Start Lifetime 3.5 / Start Speed 0.01**:视频特定值,需根据场景调整

### §11.3 推荐做法

1. **先复制本文 §4.1 的"基线配置"**
2. **根据场景调整** Duration / Start Lifetime / Max Particles
3. **根据视觉效果** 调整 Color / Size
4. **在 VRChat 中实测** 验证性能

## §12 FAQ

### Q1:为什么我的粒子是白色的方块?

**A**:通常是以下原因:
- 纹理是白底而非黑底(违反 §2.2)
- 材质未配置 Additive(违反 §3.2)
- 解决:重新导入纹理(黑底) + 配置 Additive

### Q2:粒子发射了但看不到效果?

**A**:排查:
1. Render Mode 是否为 Mesh(§5.7)
2. Material 是否正确赋值
3. Start Size 是否过小(0.0075 接近 0 看不见)
4. 摄像机视角是否对

### Q3:双眼效果只有一只眼有?

**A**:违反 §7 镜像流程:
- 漏复制父对象(Ctrl+D)
- 镜像后未改 X position / Y rotation
- 解决:严格按 §7.1 三步操作

### Q4:Quest 上粒子消失?

**A**:Quest 默认启用粒子限制(不可禁用),可能超出限制:
- 降低 Max Particles
- 减少总粒子系统数
- 详见 `avatar-particle-system-limits.md` §Quest 行为

### Q5:为什么粒子被裁剪?

**A**:可能是相机裁剪问题:
- 调整 Particle Renderer 的 Sorting Fudge
- 检查 Layer 顺序
- 检查 Material 的 Render Queue

## §13 参考资料

- [L1 Unity 官方] Particle System 总览: https://docs.unity3d.com/2022.3/Documentation/Manual/PartSysReference.html
- [L1 Unity 官方] Particle System Modules: https://docs.unity3d.com/2022.3/Documentation/Manual/ParticleSystemModules.html
- [L1 Unity 官方] Standard Unlit Shader: https://docs.unity3d.com/2022.3/Documentation/Manual/shader-StandardUnlitParticles.html
- [L1 官方] VRChat Avatar Particle System Limits: https://docs.vrchat.com/docs/avatar-particle-system-limits
- 蒸馏产物: `源资料层笔记：《笔记/avatar-video-consolidated-2026-07-04/KP04-Avatar粒子系统创建工作流》`
- 配套文档: `avatar-particle-system-limits.md`(限制端)

## §14 视频来源

- video_8 (12:50, 310 帧,眼睛流星特效)
- 蒸馏产物: `KP04-Avatar粒子系统创建工作流.md`

---

## 相关页面

[[avatar-particle-system-limits.md]] · [[optimization-guide.md]] · [[modular-avatar.md]] · [[bone-binding-workflow.md]]
