---
title: "Quest 兼容性约束清单"
category: rules
subcategory: platform-constraints
knowledge_level: core
status: active
source: "VRChat 官方文档 creators.vrchat.com/platforms/android + VRCSDK 2026.2.1 Release Notes + 本地化经验"
source_type: official
version: 1.1
last_review: 2026-07-04
changelog: "2026-07-04 §2.1 追加 250K 面数 + §2.6 新增后处理禁用 + §2.5.1 追加重新烘焙光照"
confidence: High
tags:
  - rules
  - quest
  - android
  - performance
  - platform-constraints
  - cross-platform
aliases:
  - "Quest 限制"
  - "Quest Constraints"
  - "Android 限制"
  - "Quest 兼容性"
  - 跨平台约束
related:
  - ../../entities/world/performance-guide.md
  - ../../entities/avatar/optimization-guide.md
  - ../../entities/avatar/avatar-dynamic-bone-limits.md
  - ../../entities/avatar/avatar-particle-system-limits.md
  - performance-rules.md
  - udonsharp-language-limits.md
  - ../../entities/platform/index.md
  - ../../entities/world/scene-components/index.md
type: concept
created: 2026-07-04
sources: "VRChat 官方文档 creators.vrchat.com/platforms/android + VRCSDK 2026.2.1 Release Notes + 本地化经验"
updated: 2026-07-04
---

# Quest 兼容性约束清单

> Quest(Android 平台)有独立的性能与功能约束。
> 本文档汇总 **World** 和 **Avatar** 跨 Quest 兼容必须遵守的硬性约束与最佳实践。
>
> 🔴 **Quest 性能等级 = Poor**(默认),VRC 默认不推荐 Quest 用户进入 Very Poor 性能 World。

---

## 1. 平台基础

| 项目 | PC(Win/Mac/Linux) | Quest(Android) |
|------|--------------------|------------------|
| CPU 架构 | x86_64 | ARM64 |
| GPU | NVIDIA/AMD/Apple Silicon | Adreno / Quest 3 GPU |
| 内存上限 | ~16 GB+ | **4 GB 进程上限**(Quest 2/3) |
| SDK | Unity 2022.3.22f1 LTS | Unity 2022.3.22f1 LTS(同一 SDK) |
| 上传通道 | VRChat SDK | VRChat SDK(同一通道) |
| Performance Rank | Excellent/Good/Medium/Poor/Very Poor | **Poor / Very Poor** |

---

## 2. World 约束

### 2.1 性能等级与容量

| 等级 | Triangles | Materials | Lights | Shadow Casters | Texture Memory |
|------|-----------|-----------|--------|----------------|----------------|
| Excellent | < 75k | < 5 | < 3 | < 0.5k | < 50 MB |
| Good | 75-200k | 5-15 | 3-5 | 0.5-2k | 50-200 MB |
| Medium | 200-500k | 15-25 | 5-10 | 2-5k | 200-400 MB |
| Poor | 500k-1M | 25-50 | 10-20 | 5-15k | 400-800 MB |
| **Very Poor** | 1M+ | 50+ | 20+ | 15k+ | 800 MB+ |

> 🔴 **Quest 移动端硬上限**: 250k 三角形 / 25 材质 / 4 实时阴影 / 2048² 纹理。超过会被 SDK 直接拒绝上传。

> **2026-07 数字更新（创作者实操推荐值）**:
>
> **来源**: creators.vrchat.com/platforms/android/quest-content-optimization
>
> | 内容类型 | 官方推荐 |
> |---------|---------|
> | **World 三角形总数** | **250,000** 三角形（不是 Vowgan 2022 提到的 50,000） |
> | **Avatar 三角形数** | **≤ 10,000**（与 Fallback 硬性要求一致） |
> | **Quest Fallback Avatar** | **≤ 10,000** + Good 评级 |
>
> VRChat 官方原文: "We recommend that you budget approximately 250,000 triangles for your world in total." + "We recommend that you aim for under 10,000 triangles for your avatar."

### 2.1.1 平台切换后重新烘焙光照（2026-07 补充）

> **来源**: VRCD 20-vowgan.md (COMMUNITY) + 工程经验【推断】
> **状态**: ⚠️ COMMUNITY (合理推断,官方未直接说明)

**原因**: 光烘焙数据（Lightmap, Light Probe, Reflection Probe）的存储格式与目标平台相关,切换平台后需重新烘焙。

**最佳实践**:
1. PC 项目稳定后,首次切换到 Android 时**预留 30-60 分钟**烘焙时间
2. 烘焙前: 降低 Lightmap Resolution（Quest 推荐 1024-2048）
3. 烘焙后: 用 ClientSim 在 Android 模拟器上验证

### 2.2 实时光照

| 约束 | 说明 |
|------|------|
| 实时光源数 | Quest 端**最多 4 个**(Baked Light 无限制) |
| 实时阴影 | 禁用或烘焙(Realtime Shadow 在 Quest 上成本极高) |
| Light Probe | 可用,但数量限制: < 32 |
| Reflection Probe | **Quest 端禁用实时反射**;全部烘焙 |
| Lightmap | 强制烘焙,**Atlas 共享**(减少 Texture Sampler) |

### 2.3 Udon VM 性能

> Quest 端 Udon 解释执行成本约 PC 的 **2-3 倍**。约束:

| 维度 | PC 上限 | Quest 等效上限 |
|------|---------|----------------|
| 活跃 UdonBehaviour 数 | ~150 | **~60** |
| 每帧 Udon 指令预算 | ~1M ops | **~300k ops** |
| 序列化变量数(Manual) | 280,496 bytes | 同 PC,但实际帧时间紧 |
| Update() 频率 | 每帧 | **必须用 SendCustomEvent 降频** |

> ⚠️ **Quest 上绝对避免 Update() 中分配内存**(GC 暂停在 Quest 上是 100-500 ms 一次,直接卡死)。

### 2.4 物理与动态

| 维度 | 限制 |
|------|------|
| PhysBone 数 | **1024 房间上限**(PC 同样,但 Quest 上每 PhysBone 成本高 3-5x) |
| Contact 数 | **1024 房间上限** |
| Rigidbody 数 | < 50 active(Quest 物理 CPU 预算紧) |
| Cloth 模拟 | 禁用或仅限 PC |
| Particle System | < 50 active,**强制 GPU 粒子** |

### 2.5 Shader 与材质

| 约束 | 说明 |
|------|------|
| Shader 数量 | < 25(避免变体爆炸) |
| 纹理 Atlas | 强制(同材质多对象共享一张 Atlas) |
| 纹理格式 | **ASTC**(Android 专用压缩) |
| 多 Pass | 禁用 |
| Tessellation | 禁用 |
| Compute Shader | 禁用或仅 PC |

> **2026-07 ASTC 确认**: creators.vrchat.com/platforms/android/quest-content-optimization 官方推荐 ASTC。
> **ASTC 规格速查** ([[entities/avatar/lac-avatar-compressor]]):
> - ASTC 4×4: 8 bpp (最高质量)
> - ASTC 6×6: 3.55 bpp (均衡)
> - ASTC 8×8: 2 bpp (最小)
> - ASTC 12×12: 1 bpp (极小)

### 2.6 后处理(Post-Processing) (2026-07 补充)

> **来源**: creators.vrchat.com/platforms/android/quest-content-limitations
> **状态**: ✅ FACT-OFFICIAL

> **VRChat 官方原文**:
> "Post-Processing | Post processing systems are disabled completely on Android and Quest. The GPU is not designed to handle these effects very well."

#### 实际影响

| 场景 | 表现 |
|------|------|
| **Unity Editor 测试** | 后处理**可用**(全效果) |
| **Quest 端运行时** | **完全不生效**(即使配置了也不会渲染) |
| **PC 玩家** | 正常生效 |

#### 最佳实践

- **不能依赖后处理**实现核心视觉效果
- PC/Quest 端提供**不同视觉效果**(PC 用后处理, Quest 用烘焙/材质技巧)
- 例: PC 端用 Bloom 实现"魔法光晕", Quest 端用自发光贴图 + 体积光烘焙模拟

#### Avatar 端完全禁用的组件（2026-07 补充）

来自 `creators.vrchat.com/platforms/android/quest-content-limitations` 官方禁用列表:

| 组件 | Avatar 端 | World 端 |
|------|-----------|---------|
| **Dynamic Bones** | 完全禁用 | 可用 |
| **Cloth** | 完全禁用 | 可用 |
| **Cameras** | 完全禁用 | 可用（注意性能） |
| **Lights** | 完全禁用 | 可用 |
| **Video Players** | 部分限制 | 可用 |
| **Post-Processing** | 完全禁用 | 完全禁用 |

### 2.7 音频

| 维度 | 限制 |
|------|------|
| 同播 AudioSource | < 8 个 |
| 空间音 3D | 可用,但每增加 1 个 +5% DSP 成本 |
| Reverb Zone | < 4 个 active |
| 采样率 | 22-44 kHz 足够,48 kHz 浪费内存 |

### 2.8 加载与流送

| 维度 | 限制 |
|------|------|
| 启动时间 | < 30 秒(Quest 用户耐心阈值) |
| 异步加载 | **必须**(同步加载直接 OOM) |
| Addressable | 推荐(资源动态加载) |
| VRC Async Loading | 充分利用,避免 World 启动阻塞 |

### 2.9 视频

| 维度 | 限制 |
|------|------|
| Video Player | 同步在所有平台播放 |
| 分辨率 | 720p / 30fps 上限 |
| 格式 | H.264 / VP8(Quest 硬件解码) |
| 同步模式 | AVPro / Unity Video(无 VVC) |

---

## 3. Avatar 约束

### 3.1 性能等级

| 等级 | Triangles | Materials | Texture Memory | Skinned Meshes | PhysBones |
|------|-----------|-----------|----------------|----------------|-----------|
| Excellent | < 7.5k | < 1 | < 10 MB | < 1 | < 4 |
| Good | 7.5-20k | 1-3 | 10-25 MB | 1-2 | 4-8 |
| Medium | 20-50k | 3-5 | 25-50 MB | 2-3 | 8-16 |
| Poor | 50-100k | 5-10 | 50-100 MB | 3-4 | 16-32 |
| **Very Poor** | 100k+ | 10+ | 100 MB+ | 4+ | 32+ |

> 🔴 **Quest 移动端 Avatar 硬上限**: 50k 三角形 / 5 材质 / 100 MB 纹理 / 8 PhysBones。超过会被 SDK 拒绝上传。

> **2026-07 Quest Fallback 硬性规则(必读)**:
>
> **来源**: docs.vrchat.com/docs/avatar-fallback-system (FACT-OFFICIAL) + wiki.vrchat.com (FACT-WIKI)
>
> Quest Fallback Avatar 必须同时满足以下 4 个条件:
>
> | 条件 | 说明 |
> |------|------|
> | 1. **Performance Rank: Good 或 Excellent** | 基于 Android/Quest 评级标准 |
> | 2. **三角形数 ≤ 10,000** | Good 评级上限(Excellent 为 7,500) |
> | 3. **平台覆盖完整** | 必须同时上传 Windows + Android/Quest |
> | 4. **Fallback 标志** | Android 上传时必须设置 Fallback 标记 |
>
> 🔴 **关键 Insight**: Fallback 评级基于 **SDK 上传时**的判断, 不是用户 Perf Filtering 后的结果.
>
> 详细文档: [[entities/avatar/avatar-fallback-system]] §Quest Fallback 硬性规则

### 3.2 物理与动态

详见 [[entities/avatar/avatar-dynamic-bone-limits]] 与 [[entities/avatar/avatar-particle-system-limits]]。

核心约束:

| 维度 | Quest 等效上限 |
|------|----------------|
| PhysBone Transform 数 | < 64(Quest 端动态模拟预算紧) |
| PhysBone Collider 数 | < 16 |
| Contact Sender + Receiver | < 8 个总和 |
| Particle System | < 4 个 active |

### 3.3 Shader 与材质

| 约束 | 说明 |
|------|------|
| Shader 库 | lilToon / Poiyomi / SCSS(经 Quest 优化版本) |
| 变体数 | < 256(Quest 启动时 Shader 编译时间预算) |
| 烘焙光照 | 强制(Avatar 端无实时光) |
| 描边 / 高光 | 允许,但需每像素成本评估 |
| Tessellation | 禁用 |

### 3.4 动画与播放层

| 维度 | 限制 |
|------|------|
| Animator Controller | 可用,但 Layer 数量 < 5 |
| Animator Parameter | < 32(Quest 端每参数 +0.5% Update 成本) |
| Playable Layers | 5 层(Base/Additive/Gesture/Action/FX) |
| Blend Shape | < 100(Quest 端变形计算预算紧) |

### 3.5 音频

| 维度 | 限制 |
|------|------|
| Audio Source | < 2 个持续播放 |
| 采样率 | 22 kHz 足够 |
| 3D Audio | 可用,但避免 DSP 插件 |

### 3.6 OSC 通信(Avatar 端 OSC)

> Quest 端 OSC **仅支持接收**(不能发送),且需要 OSCQuery 协议。
> 详见 [[concepts/hybrid/osc-protocol]] 与 [[entities/avatar/vrcraycast]]。

---

## 4. 跨平台兼容清单(World 上传 Quest 前必查)

```
□ 实时光源数 < 4
□ 实时阴影 = 0(全部烘焙)
□ Reflection Probe = 0 实时
□ Triangle 数 < 250k
□ Material 数 < 25
□ Texture 全部 ASTC
□ AudioSource 持续播放 < 8
□ Udon Update() 内存分配 = 0
□ UdonBehaviour 活跃数 < 60
□ PhysBone + Contact 房间总数 < 1024
□ Cloth Simulation = 0
□ Tessellation = 0
□ Compute Shader = 0
□ 启动时间 < 30s(实测,不是估算)
```

---

## 5. 性能调优建议

### 5.1 World 调优优先级

1. **静态合批 + GPU Instancing**(降低 Draw Call,Quest GPU 瓶颈)
2. **Lightmap Atlas 共享**(降 Texture Sampler)
3. **Occlusion Culling**(Quest 端 CPU 渲染预算紧)
4. **LOD Group**(降低远距 Triangle)
5. **Udon Update() 降频** → 改为 `SendCustomEventSeconds(0.1f, ...)` 之类

### 5.2 Avatar 调优优先级

1. **合并 SkinnedMesh**(`Avatar 3.0 Toolbox` / `modular-avatar`)
2. **Texture Atlas 合并**
3. **PhysBone 链数 < 8**
4. **Shader 选用 Poiyomi / lilToon 的 Quest 预设**
5. **动态骨骼降频**(部分骨骼用 30 Hz 模拟)

---

## 6. 测试与发布

### 6.1 本地测试

- **ClientSim**:模拟 Quest 性能等级(可选 Excellent/Good/Medium/Poor/Very Poor)
- **Android Build Target**: 实际打包 APK 测启动时间 / 内存占用
- **Profiler**:Unity Profiler → CPU Usage → Android Player

### 6.2 发布检查

- **VRChat SDK Control Panel → Build → Cross-Platform Build**
- 检查 "Quest Compatibility" 标签
- 如有警告,逐项修正后重新上传

### 6.3 性能等级(Performance Rank)

上传后 SDK 自动检测,生成等级标签。**VRChat Quest 用户可在设置中过滤等级**:

- "Show Excellent" / "Show Good" / 等
- 默认:**不过滤** = 显示所有等级
- 推荐设置:**不过滤 Poor 以上的**(给 Quest 用户访问机会)

---

## 7. 已知坑

1. **PC Build 通过 ≠ Quest Build 通过** — 必须在 Android Build Target 下完整测一次
2. **Texture Reimport 默认 PC 格式** — 上传前手动设 Android Override
3. **Lightmap 大小** — Quest 端 Atlas > 4096² 会被 SDK 警告
4. **Udon GC 在 Quest 上是 10-100 倍卡顿** — 任何 `new` / `List.Add` / string interpolation 在热路径都是禁忌
5. **VRC 消息载荷** — Quest 网络带宽 ~5 KB/s(PC 是 11 KB/s),Manual Sync 数据量需按比例缩减
6. **粒子 GPU 模拟** — Quest 上 `Particle System` 默认 CPU 模拟,需手动改 GPU 模式
7. **OSC 限制** — Quest OSC 只能收,无 VRChat Settings API
8. **Rigidbody 数量** — > 50 active 会在 Quest 上直接掉帧

---

## 8. 相关文档

- **World 性能优化**:[[entities/world/performance-guide]]
- **Avatar 优化**:[[entities/avatar/optimization-guide]]
- **PhysBone 限制**:[[entities/avatar/avatar-dynamic-bone-limits]]
- **Particle 限制**:[[entities/avatar/avatar-particle-system-limits]]
- **Udon 语言限制**:[[concepts/rules/udonsharp-language-limits]]
- **平台概览**:[[entities/platform/index]]
- **World Components 限制**:[[entities/world/scene-components/index]]
- **OSC 协议**:[[concepts/hybrid/osc-protocol]]
