---
title: VRChat Avatar 大小限制与定义
category: avatar

knowledge_level: applied
status: active

tags:
  - avatar
  - size-limit
  - vram
  - compliance
  - regulation
  - compression
  - optimization

aliases:
  - "Avatar Size Limits"
  - "VRAM 限制"
  - "Avatar 大小限制"
  - "模型大小限制"
  - "Avatar File Size"

related:
  - optimization-guide.md
  - performance-rank.md
  - avatar-fallback-system.md
  - impostor-fallback.md
  - avatar-audio-optimization.md

source: 18-vrchat-CI4 (VRCD) + VRChat 官方 Dev Update (2024-03-14)
source_type: community
version: 1.1
last_review: 2026-07-04
changelog: "2026-07-04 §2.3 新增 Quest 实操目标值（5-8MB）"
confidence: High
type: entity
created: 2026-07-04
sources: 18-vrchat-CI4 (VRCD) + VRChat 官方 Dev Update (2024-03-14)
updated: 2026-07-04
---

# VRChat Avatar 大小限制与定义

> **来源**:
> - VRChat 官方 Dev Update (2024-03-14): https://ask.vrchat.com/t/developer-update-14-march-2024/23401
> - VRCD 18-vrchat-CI4: https://docs.vrcd.org.cn/books/vrchat-CI4
> - 知识库沉淀日期: 2026-07-04
> **状态**: ✅ FACT (VRChat 官方 + VRCD 中文社区)
> **关键 Insight**: 创作者必须理解 3 个不同的大小指标（压缩/未压缩/VRAM），混淆会导致优化方向错误。

---

## 1. Avatar 大小三件套定义 ⭐K01

### 1.1 三个评价指标

> [FACT] VRChat 中评估一个 Avatar 模型的大小，必须区分 **3 个不同**的指标：

| 指标 | 名称 | 含义 | 影响 |
|------|------|------|------|
| 1 | **压缩大小 / 下载文件大小** | 模型文件下载时的实际大小 | 下载时间、上传拦截 |
| 2 | **未压缩大小 / 解压后大小** | 模型文件解压到设备 RAM 后的总大小 | SDK 上传拦截、玩家运行时控制 |
| 3 | **显存使用量 / 显存占用 (VRAM)** | 模型使用的显卡内存（VRAM）大小 | 显存带宽、GPU 性能 |

### 1.2 关键区分：VRAM vs Uncompressed Size

> [FACT] **VRChat 官方明确区分**：
>
> 当前 VRChat 的"显存占用"**只统计贴图大小，不包含网格占用**。
> — VRCD 18-vrchat-CI4

> [FACT] **tupper (VRChat Head of Community) 官方说明 (2024-03-14 Dev Update)**：
>
> "It's the size of the avatar after it has been decompressed out of its asset bundle packaging as it sits in your RAM. This includes **EVERYTHING** – not just textures and mesh, but also animations, audio, shaders, currently-unrendered assets, etc. **It isn't quite VRAM, but it is linearly related.**"
> — https://ask.vrchat.com/t/developer-update-14-march-2024/23401

**关键差异总结**：

| 指标 | 包含内容 | 评价对象 |
|------|----------|----------|
| **VRAM（显存占用）** | **仅贴图** | 显卡性能 |
| **Uncompressed Size（未压缩大小）** | **贴图 + 网格 + 动画 + 音频 + Shader + 未渲染资产** | RAM 容量 |

### 1.3 在 Unity 内的查看方法

> [FACT] 创作者可在 Unity 内通过以下工具查看自己的模型大小：
> - **thry** (Thry's Avatar Optimization Tools) — 性能评估
> - **lilAvatarUtils** (lilxyzw) — 性能检测 + 批量贴图设置
> - VRChat SDK 内置的 Avatar Performance Rank

**Avatar 详情页面三栏展示**：
- `UNCOMPRESSED SIZE`：解压文件大小
- `下载文件大小`：压缩后大小
- `显存占用`：VRAM 使用量

---

## 2. Avatar 大小限制 ⭐K02

### 2.1 PC 平台限制（强制实施）

> [FACT] **VRChat 官方 FlareRune (2024-03-14 Dev Update)**：
>
> "We are changing the 'Compressed Size' (download size) limit from 500MB to 200MB, and the 'Uncompressed Size' (Estimated Memory usage) from 1.2GB to 500MB. These numbers aren't completely locked in and may change."

| 指标 | 7 月 16 日前 | 7 月 16 日后 | 变化 |
|------|--------------|--------------|------|
| **未压缩大小（RAM 使用率）** | 1200 MB | **500 MB** | -41.6% |
| **压缩大小（下载大小）** | 500 MB | **200 MB** | -40% |

> ⚠️ **官方说明**: "These numbers aren't completely locked in" → 限制数字可能微调

### 2.2 Quest (Android) 平台限制

> [FACT] **Quest 平台指标未改变**（来源: VRCD 18-vrchat-CI4）：

| 指标 | 限制 |
|------|------|
| 最大压缩大小 | **10 MB** |
| 最大未压缩大小 | **40 MB** |

> Quest 标准极为严格，Very Poor 会被强制不显示（详见 `performance-rank.md`）

### 2.3 实操目标值（2026-07 补充）

> **来源**: creators.vrchat.com/platforms/android/quest-content-optimization
> **状态**: ✅ FACT-OFFICIAL

> **VRChat 官方原文**:
> "You should be aiming for a maximum of 5-8 MB. You cannot upload or wear/view avatars that exceed 10MB in size after build-time compression for VRChat on Android."

| 指标 | 硬性限制 | 创作者实操目标 | 差异 |
|------|---------|---------------|------|
| **Avatar 压缩大小** | 10 MB | **5-8 MB** | 留出 20-50% 缓冲 |
| **Avatar 未压缩大小** | 40 MB | ~25-35 MB | 应对多玩家并发 |
| **World 压缩大小** | 100 MB | 越低越好 | 加载时间 + 稳定性 |

> 🔴 **关键 Insight**: 虽然硬性限制是 10MB, 但官方建议**瞄准 5-8 MB** —
> 因为要考虑**多玩家同时加载** + **不同设备性能差异** + **后期持续优化**。

#### 优化方向对大小的影响

| 优化方向 | 减少大小效果 | 优先级 |
|---------|------------|--------|
| 降低贴图分辨率 | 显著（10MB+ → 3-5MB） | ⭐⭐⭐ |
| 切换为 ASTC 压缩 | 中等（2-3MB 节省） | ⭐⭐⭐ |
| 减少材质数 | 中等 | ⭐⭐ |
| 减面 | 中等 | ⭐⭐ |
| 移除未使用资源 | 小 | ⭐ |

### 2.4 强制实施时间线

> [FACT] **来源: VRCD 18-vrchat-CI4**（⚠️ 时间点未在 VRChat 官方 Dev Update 中明确，标注为 VRCD 文档）：

| 日期 | 强制阶段 |
|------|----------|
| **2024-07-16** | VRC SDK 端上传时**强制拦截**超限 Avatar |
| **2024-11-01** | VRC 游戏端**强制安全检查**（Security Checks Failed） |

### 2.5 违规后果

**1. SDK 端上传拦截**

> 当您的模型未压缩大小违规时，SDK 将提醒您的模型已超过对应限制，且会报错无法上传

**2. 游戏端安全检查失败**

> 当 Avatar 超过限制时，会被标记为 "安全检查失败（Security Checks Failed）"，**无法加载**

### 2.6 玩家侧运行时控制

> [FACT] **玩家在 Settings 中可独立配置**（详见 `impostor-fallback.md §1.1-1.2`）：

| 玩家设置 | 默认值 | 步长 | 超限后果 |
|----------|--------|------|----------|
| **Maximum Avatar Download Size** | 200MB | 5MB | 不会自动下载，需手动点击 "Show Avatar" |
| **Maximum Uncompressed Size** | 300MB（推荐） | - | 用 Impostor / Fallback 显示 |
| **Block Poorly Optimized Avatars** | Excellent | - | 自动屏蔽低于最低 rank 的 Avatar |

> 💡 **创作者启示**:
> - 即使通过 SDK 上传，玩家**仍可能**因为自己的设置看不到你的 Avatar
> - 优化目标是让 Avatar 在 **300MB 以下**（玩家侧推荐值）

---

## 3. 优化方向总览

> 以下为详细的优化方法，分别沉淀在对应子文档中：

| 方向 | 关键工具 | 详细文档 |
|------|----------|----------|
| **贴图** | AAO + LAC + lilAvatarUtils | `optimization-guide.md §Texture Memory` |
| **音频** | Unity Audio Clip 设置 | `avatar-audio-optimization.md` |
| **动画** | AAO Trace And Optimize + Keyframe Reduction | `optimization-guide.md §Animation Optimization` |
| **网格** | Meshia / AAO Remove Mesh | `performance-rank.md` + `meshia-mesh-simplification.md` |
| **骨骼/物理** | AAO Merge Bone / Merge PhysBone | `optimization-guide.md §Bones / §PhysBone` |

### 优化优先级（2026 推荐）

```
1. 贴图（影响最大）→ LAC 自动压缩 + 手动 Max Size
2. 音频（长音频显著）→ Vorbis + Compressed In Memory
3. 动画 → AAO Trace And Optimize
4. 网格 → Meshia 减面（仅当必要）
5. 骨骼/物理 → AAO Merge 系列
```

---

## 4. FAQ

### 4.1 为什么 VRAM 不算网格？

> [FACT] **VRChat 官方 Performance Rank 系统的 Texture Memory 指标仅统计贴图**。这是因为：
> - 网格占用相对贴图较小（顶点数 vs 像素数）
> - 贴图是 VRAM 带宽的主要消耗者
> - 性能分级系统需要可量化的指标

### 4.2 1.2GB → 500MB 过渡期内未优化的旧 Avatar 怎么办？

> [FACT] **VRChat 官方不追溯既往**（Grandfathered 规则）：
> - 旧 Avatar 在 2024-07-16 前上传的不受影响
> - 但**重新上传**时必须符合新限制
> - 详见 `avatar-fallback-system.md`

### 4.3 Quest 平台为什么不变？

> [FACT] Quest 平台（10MB / 40MB）**早已严格**，此次 PC 平台调整是**向 Quest 标准靠拢**。

### 4.4 三种大小指标有什么关系？

> [FACT] 关系图：
>
> **VRAM** ⊂ **Uncompressed Size** ⊃ **Compressed Size**（下载）
>
> - VRAM 是 Uncompressed Size 的一部分（仅贴图）
> - Uncompressed Size 经 AssetBundle 压缩后得到 Compressed Size
> - AssetBundle 压缩比因内容而异

---

## 5. 引用与参考

### 5.1 官方来源

- [VRChat Dev Update - 14 March 2024](https://ask.vrchat.com/t/developer-update-14-march-2024/23401)
  - FlareRune 关于大小限制数字的官方声明
  - tupper (VRChat Head of Community) 关于 Uncompressed Size 的解释
  - VRChat 官方对 Crunch 压缩的立场
  - VRChat 官方对 400% 量化关系的说明
- [VRChat Creator Docs](https://creators.vrchat.com/avatars/avatar-performance-ranking-system/) — Performance Rank 标准

### 5.2 社区来源

- [VRCD 18-vrchat-CI4](https://docs.vrcd.org.cn/books/vrchat-CI4) — 如何压缩您的 VRChat 模型大小
- VRCD 18-vrchat-CI4 知识整理笔记（来源引证：VRCD 18-vrchat-CI4 知识整理笔记——源引文件不随发布包分发）

### 5.3 知识库内部

- `optimization-guide.md` — 最佳化实操指南
- `performance-rank.md` — PC/Quest 性能等级标准
- `impostor-fallback.md` — 玩家侧 Maximum Size 控制
- `avatar-fallback-system.md` — Fallback 系统（含 Grandfathered 规则）
- `avatar-audio-optimization.md` — 音频优化（独立文档）

---

## 相关页面

[[optimization-guide.md]] · [[performance-rank.md]] · [[avatar-fallback-system.md]] · [[impostor-fallback.md]] · [[avatar-audio-optimization.md]]
