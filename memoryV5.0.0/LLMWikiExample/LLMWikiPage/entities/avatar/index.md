---
title: "Avatar Domain — Knowledge Base"
category: avatar
knowledge_level: applied
status: active
source: 本地知识库整理
source_type: community
version: 1.10
last_review: 2026-07-04
confidence: Medium
tags:
  - misc
  - index
  - navigation
aliases:
  - "Avatar Domain — Knowledge Base"
related:
  - "[[animator-system.md]]"
  - "[[performance-rank.md]]"
  - "[[thry-avatar-evaluator-metrics.md]]"
  - "[[ndmf-tools.md]]"
  - "[[meshia-mesh-simplification.md]]"
  - "[[vrcraycast.md]]"
  - "[[lac-avatar-compressor.md]]"
  - "[[avatar-optimizer.md]]"
  - "[[optimization-guide.md]]"
  - "[[teaching-methodology.md]]"
  - "[[modular-avatar.md]]"
  - "[[modular-avatar-tutorials-detailed.md]]"
  - "[[tex-trans-tool.md]]"
  - "[[vrcfury-reference.md]]"
  - "[[ma2bt.md]]"
  - "[[avatar-parameter-staged-sync.md]]"
  - "[[ma-component-cards.md]]"
  - "[[contact.md]]"
  - ../../[[shader/other-shaders.md]]
  - ../../[[shader/unlitwf/index.md]]
  - ../../[[shader/orl/index.md]]
  - ../../[[shader/filamented/index.md]]
  - ../../[[shader/poiyomi/index.md]]
  - ../../[[shader/poiyomi/shader-locking-mechanism.md]]
  - "[[avatar-fallback-system.md]]"
  - "[[avatar-dynamic-bone-limits.md]]"
  - "[[avatar-particle-system-limits.md]]"
  - "[[avatar-particle-system-creation.md]]"
  - "[[full-body-tracking.md]]"
  - "[[ik-2.0.md]]"
  - "[[avatar-interaction-permissions.md]]"
  - "[[public-avatar-cloning.md]]"
  - "[[vrcraycast.md]]"
  - "[[integral-camera-params.md]]"
  - "[[blender-shape-keys.md]]"
  - "[[avatar-size-limits.md]]"
  - "[[avatar-audio-optimization.md]]"
  - "[[world-fixed-objects.md]]"
  - "[[self-mmd.md]]"
  - "[[zh-translation-glossary.md]]"
  - "[[bone-binding-workflow.md]]"
  - "[[blend-shape-troubleshooting.md]]"
type: entity
created: 2026-07-04
sources: 本地知识库整理
updated: 2026-07-04
---
# Avatar Domain — Knowledge Base

## Obsidian MOC — Avatar 可点击入口

| 分类 | 入口 | 说明 |
|---|---|---|
| 核心系统 | [[animator-system.md]] · [[playable-layers.md]] · [[vrc-constraints.md]] · [[contact.md]] · [[expression-menu.md]] | Avatar 3.0、Animator、Constraint、Contact、Expression |
| 优化与性能 | [[performance-rank.md]] · [[optimization-guide.md]] · [[avatar-optimizer.md]] · [[ndmf-tools.md]] · [[lac-avatar-compressor.md]] | PC/Quest 性能、NDMF 工具、贴图/网格/组件优化 |
| Shader | [[shader/index.md]] · [[shader/liltoon/index.md]] · [[shader/poiyomi/index.md]] · [[shader/orl/index.md]] · [[shader/filamented/index.md]] · [[shader/unlitwf/index.md]] | Avatar Shader 选择与专题入口 |
| 工具与改模 | [[modular-avatar.md]] · [[vrcfury-reference.md]] · [[teaching-methodology.md]] · [[avatar-modding-guide.md]] | MA / VRCFury / 教学与改模流程 |
| 摄影与相机 | [[camera-tools.md]] · [[camera-tools-virtuallens2.md]] · [[camera-tools-vrclens.md]] · [[integral-camera-params.md]] | 玩家本地相机工具与参数 |
| 边界与跨域 | [[../../concepts/hybrid/avatar-world-boundaries.md]] · [[../../concepts/hybrid/avatar-world-contact-bridge.md]] · [[../../concepts/hybrid/player-observation-boundaries.md]] | Avatar 与 World 交互边界 |

---

## 核心文档

| 文档 | 说明 | 来源 |
|------|------|------|
| **[animator-system.md](animator-system.md)** | Write Defaults、Avatar Mask、Playable Layers、参数类型、Direct Blend Tree、§4.5 设计哲学、§4.6 Parameter Driver 能力边界、**§1.0.1 帽子示例**、**§5.1.1 逻辑门** | VRCLibrary + VRCD + VRChat 官方 |
| **[performance-rank.md](performance-rank.md)** | PC/Quest 性能等级标准、各指标限制 | 官方文档 |
| **[thry-avatar-evaluator-metrics.md](thry-avatar-evaluator-metrics.md)** ⭐NEW | Thry 工具检测的 7 项指标完整阈值(VRAM/GrabPass/Blendshape/AnyState/Layer/WriteDefaults/EmptyStates)+ 优化决策树 + 与官方 Performance Rank 对照表 | Thryrallo/VRC-Avatar-Performance-Tools v1.3.7 源码 (2026-06-17) |
| **[ndmf-tools.md](ndmf-tools.md)** ⭐更新 2026-08-13 | NDMF 工具生态、VPM 链接、执行顺序、MA vs VRCFury 工具选型决策树、NDMF 1.14.0 API/兼容性 | Kuriko 笔记 + VRCFury 官网 + NDMF GitHub |
| **[meshia-mesh-simplification.md](meshia-mesh-simplification.md)** ⭐NEW | Meshia 完整技术文档（Burst+Job 算法、API、Options、BlendShape、与 lilNDMF/Mantis 对比） | RamType0 官方 (2026-06-17) |
| **[lac-avatar-compressor.md](lac-avatar-compressor.md)** ⭐NEW | **Avatar Compressor (LAC)** - 全自动纹理压缩工具（4 策略/5 预设/平台自动/VRAM 预估/类型感知）| Limitex 官方 (2026-06-17) |
| **[avatar-optimizer.md](avatar-optimizer.md)** ⭐更新 2026-08-13 | AAO: Avatar Optimizer 完整知识库 + v1.9.13 PhysBone/SDK/EyeLook 修复 | 官方文档 + GitHub release |
| **[optimization-guide.md](optimization-guide.md)** | 完整最佳化实操指南（Light→Particle→Texture→Mesh→Material→PhysBone→Bones→Polygon） | Kuriko 笔记 |
| **[teaching-methodology.md](teaching-methodology.md)** | Avatar 改模教学法、问题诊断框架、玩家常见踩坑分类(34 条原则 + 9 句式 + 6 禁忌 + 升级检查清单 + MA 5 大玩家友好设计 + **§MM-PP problems 章节教学法** 2026-06-17) | vrnavi.jp + vrcmaster.com + **Kuriko HackMD** 三源 + **MA 官方教程原文** + **MA 官方 problems 章节** (2026-06-17) |
| **[modular-avatar.md](modular-avatar.md)** ⭐更新 2026-08-13 | Modular Avatar 完整知识库 + 1.17.0/1.18.0-alpha.0 的 Floor Adjuster、VRCRaycast 参数、Blendshape 曲线和 Mesh Cutter 更新 | 官方文档 + GitHub releases |
| **[modular-avatar-tutorials-detailed.md](modular-avatar-tutorials-detailed.md)** ⭐NEW | MA 6 个教程的官方原文精读 + 玩家视角操作分解 + 验证步骤 + 易错点 + 教学衔接路径 | 官方教程 (2026-06-17) |
| **[tex-trans-tool.md](tex-trans-tool.md)** ⭐NEW | **TexTransTool (TTT)** - 非破坏纹理改写工具完整知识库(15 组件 + AtlasTexture 完整参数表 + Quest 适配 + 与 AAO 协作 + v0.10.0/v1.0.0 破坏性变更 + 故障排查) | ReinaS-64892 官方 + ttt.rs64.net (2026-06-17) |
| **[vrcfury-reference.md](vrcfury-reference.md)** ⭐深度更新 | VRCFury 完整参考：Parameter Compressor (16 bit 压参数) + Direct Tree Optimizer (减层数) + Blendshape Optimizer (减 VRAM) + Fix Write Defaults + Actions 系统 + 60+ 自动修复 + SPS + 全组件清单 + Quest 兼容 + MA/d4rk 共存 + 优化场景对比矩阵(含 d4rk) | 官网 + 仓库 + PR#238 (2026-06-17) |
| **[ma2bt.md](ma2bt.md)** ⭐NEW | **MA2BT** (Modular Avatar to BlendTree) - 将 MA 响应式层合并为 Direct BlendTree 减少 FX 层数（Compact Mode / Multi-State / Scan All Layers 选项 + 完整跳过原因枚举 + 与 AAO 互补关系 + 16 节完整知识库） | Null-K 官方仓库 v2.0.2 (2026-06-17) |
| **[avatar-fallback-system.md](avatar-fallback-system.md)** ⭐NEW 2026-06-30 | Avatar Fallback 系统完整指南（5 种 Fallback 触发原因 + 自定义 Fallback 上传流程 + Grandfathered 规则 + 完整 FAQ + 文件大小 vs VRAM 区分） | VRChat 官方 docs (2026-06-30) |
| **[avatar-dynamic-bone-limits.md](avatar-dynamic-bone-limits.md)** ⭐NEW 2026-06-30 | 用户端 Dynamic Bone Limits 系统（默认值 32/8 + config.json 完整配置 + 与 Performance Rank 区分 + Quest 行为） | VRChat 官方 docs (2026-06-30) |
| **[avatar-particle-system-limits.md](avatar-particle-system-limits.md)** ⭐NEW 2026-06-30 | 用户端 Particle System Limits 系统（11 个配置变量 + Penalty 计算公式 + Quest 默认启用且不可禁用 + PlayerLocal 碰撞限制） | VRChat 官方 docs (2026-06-30) |
| **[full-body-tracking.md](full-body-tracking.md)** ⭐NEW 2026-06-30 | Full-Body Tracking 完整指南（8 tracker 限制 + SteamVR 配置 + 标准/Legacy 校准 + 10 条 Rigging 要求 + Avatars 3.0 集成 + 实验性状态） | VRChat 官方 docs (2026-06-30) |
| **[ik-2.0.md](ik-2.0.md)** ⭐NEW 2026-06-30 | IK 2.0 完整特性（IK Legacy 切换 + Avatar Measurement + Lock Types + Locomotion toggle + 5 个启动参数） | VRChat 官方 docs (2026-06-30) |
| **[avatar-interaction-permissions.md](avatar-interaction-permissions.md)** ⭐NEW 2026-06-30 | Avatar 交互权限设置（Mode/Allow-Pause/Self-Interact + 玩家级覆盖 + PANIC 按钮） | VRChat 官方 docs (2026-06-30) |
| **[public-avatar-cloning.md](public-avatar-cloning.md)** ⭐NEW 2026-06-30 | Public Avatar 克隆（克隆流程 + Allow Avatar Cloning 设置 + Private 化） | VRChat 官方 docs (2026-06-30) |
| **[accessories.md](accessories.md)** ⭐NEW 2026-06-30 | **Avatar Accessories 配件系统** - 2026.2.1+ 新功能，PC=240/Quest=80 全局渲染限制，Look Editor 整合，MA 集成，常见错误修复 | VRChat 2026.2.1 Release Notes |
| **[vrcraycast.md](vrcraycast.md)** ⭐NEW 2026-06-30 | **VRCRaycast 骨骼 raycast** - SDK 3.10.3+ 引入，2026.2.1 改进（自动剥离/Animator 修复/第一人称对称） | VRChat 2026.2.1 Release Notes |
| **[avatar-30-emulator.md](avatar-30-emulator.md)** ⭐NEW 2026-07-04 | **Avatar 3.0 Emulator (lyuma)** - Unity 内模拟 Avatar3 运行时的测试工具（PlayableGraph API + VPM `lyuma.av3emulator` + 571+ stars + "Animator To Debug" + "Create Non Local Clone"） | lyuma GitHub 官方 |
| **[avatar-30-toggle-tools.md](avatar-30-toggle-tools.md)** ⭐NEW 2026-07-04 | **Avatar 3.0 Toggle 创建工具（非 MA 路径）** - VRChat Toggle Assistant + Auto-Toggle-Creator + VRCToggleToolkit + AV3ToggleUtil 的工具生态对比 + 工具选择决策树 | Shatteredfur / CascadianVR / SuperFlue / d4rkc0d3r |
| **[expression-menu.md](expression-menu.md)** ⭐更新 2026-07-04 | Action/Expression/Puppet Menu 三层结构 + **§9.5 Expression Parameters 字段详解**（Saved/Default/Synced） + **§9.6 自定义 Menu 替换默认 Emotes 关键警告** | VRChat 官方 + 社区 |
| **[integral-camera-params.md](integral-camera-params.md)** ⭐NEW 2026-07-04 | Integral 相机参数详解（Aperture/Zoom/Focus/Exposure/SS + MF/AF/Visual AF 三模式对焦 + 抖动合成景深 + 与 VirtualLens2/VRCLens 对比）| Integral 笔记整理 (2026-07-04) |
| **[camera-tools.md](camera-tools.md)** ⭐更新 2026-07-13 | 玩家本地相机工具总览（VirtualLens2 / VRCLens 设计哲学对比 + 选型） | 社区评估 |
| **[camera-tools-virtuallens2.md](camera-tools-virtuallens2.md)** ⭐NEW 2026-07-13 | VirtualLens2 子工具文档（依赖、工作流、Remote / Drone / Focus / Non-Destructive 细节） | 社区整理 |
| **[camera-tools-vrclens.md](camera-tools-vrclens.md)** ⭐NEW 2026-07-13 | VRCLens 子工具文档（Apply 逻辑、原相机前提、HDR / White Balance / Invasive 工作流） | 社区整理 |
| **[blender-shape-keys.md](blender-shape-keys.md)**
| **[legal-and-licensing.md](legal-and-licensing.md)** ⭐NEW 2026-07-04 | **VRC 创作者法律合规总览** - VN3 协议、量贩模型法律红线、Filian/竜胆案例（社区转述） | 86vrchat + VN3 官网 |
| **[commission-pricing.md](commission-pricing.md)** ⭐NEW 2026-07-04 | **VRC 委托模型价格区间** - 三档预算（低/中/高）+ 三种工具链对比（Blender/VRoid/恋活） | 86vrchat |
| **[world-fixed-objects.md](world-fixed-objects.md)** ⭐NEW 2026-07-04 | **Avatar 固定世界物品结构** - 5 空物体 + 3 Constraint 精确参数(0.5/-1)+ 物品骨骼陷阱 + 适用场景评估 | yexcadocs §Bone Binding (P1) |
| **[self-mmd.md](self-mmd.md)** ⭐NEW 2026-07-04 | **自身跳 MMD 舞蹈** - Action+FX 层配置 + Locomotion 配对 + 顺序/随机音乐 + 出场动画 + VRC 工具箱工作流 | yexcadocs §自身跳 MMD (P1/P2) |
| **[avatar-size-limits.md](avatar-size-limits.md)** ⭐NEW 2026-07-04 | **VRChat Avatar 大小限制与定义** - 3 大小指标区分（VRAM vs Uncompressed Size）+ PC 500MB/200MB + Quest 10MB/40MB + 强制时间线（2024-07-16 / 2024-11-01）+ 玩家侧运行时控制 | VRChat 官方 Dev Update (2024-03-14) + VRCD 18-vrchat-CI4 |
| **[avatar-audio-optimization.md](avatar-audio-optimization.md)** ⭐NEW 2026-07-04 | **VRChat Avatar 音频优化** - 4 压缩格式（PCM/ADPCM/Vorbis/MP3）+ 3 加载类型 + 量化数据 + Load In Background VRChat SDK 强制 + 选型决策树 | Unity 官方 Audio Clip Manual (2022.3) + VRCD 18-vrchat-CI4 |
| **[optimization-guide.md](optimization-guide.md)** ⭐更新 2026-07-04 | 完整最佳化实操指南（...+ **6 贴图导入设置** + **§Animation Optimization** + **§Audio Optimization**） | Kuriko 笔记 + VRCD 18-vrchat-CI4 |
| **[zh-translation-glossary.md](zh-translation-glossary.md)** ⭐NEW 2026-07-04 | **VRChat Avatar 中英术语对照表** - 36 条系统对照 + 9 条欠标准译名警示 + 4 大术语辨析（Texture/Material、State/Animation、Avatar/头像、Animator/动画师）+ 翻译规范建议 | VRCD 21 附录（[Avatar 文档（vrcd.org.cn）](https://docs.vrcd.org.cn/books/avatar)） |
| **[shader/poiyomi/shader-locking-mechanism.md](shader/poiyomi/shader-locking-mechanism.md)** ⭐NEW 2026-07-04 | **Poiyomi Shader Locking 机制与动画化** - VRChat Shader Locking 自动机制 + Poiyomi `Animated (when locked)` / `Renamed (when locked)` 标记 + 4 种 Copy Property Name 选项 + VRCFury Material Property 集成 + d4rkAO 工具链冲突 + VRC School Hue Shift 工作流 + Poiyomi C# Lock API | Poiyomi 官方 + VRCFury 官方 + VRC School + d4rkAO Issue #182 (2026-04-01) |
| **[bone-binding-workflow.md](bone-binding-workflow.md)** ⭐NEW 2026-07-04 | **Avatar 骨绑定工作流决策树** - 3 种方法（手动 / Merge Armature / Bone Proxy）+ 决策矩阵 + 部分骨架失效【推断】根因 + 4 个实战示例 + FAQ | VRCD 视频蒸馏 (video_6) + MA 官方 + Unity 官方 |
| **[blend-shape-troubleshooting.md](blend-shape-troubleshooting.md)** ⭐NEW 2026-07-04 | **Blend Shape 系统性排障 4 步法** - 模型/MA 组件/内置菜单/FX Layer 4 步定位 + Shape Changer 冲突 + FX Layer Weight 排查 + WD 检查 + 功能分离原则 + 瞳关闭动画特殊流程 + FAQ | VRCD 视频蒸馏 (video_SP09) + MA 官方 + Unity 官方 |
| **[avatar-particle-system-creation.md](avatar-particle-system-creation.md)** ⭐NEW 2026-07-04 | **Avatar 粒子系统创建工作流** - 纹理准备（黑底+白图案）+ 材质配方（Standard Unlit + Additive + Two Sided）+ Main Module + 附加模块 + Parent Object 设计 + 镜像双眼 + 挂接眼睛流程 + 关键帧核对记录 | VRCD 视频蒸馏 (video_8) + Unity Particle 官方 + VRChat Limits 官方 |
| **[avatar-modding-guide.md](avatar-modding-guide.md)** ⭐更新 2026-07-04 | VRChat Avatar 改模基础知识（...+ **§3.5 模型导入工作流** + **§4 SDK 面板诊断流程**）| 86vrchat + VRCD 蒸馏 + VRChat 官方 |
| **[faceemo.md](faceemo.md)** ⭐NEW 2026-08-19 | **FaceEmo** - 面部表情创建与配置工具（Pattern/手势条件/Blink/LipSync/眼动；官方 1.7.0 条件系统：手形×手势；生成式非破坏；经 MA 安装） | suzuryg 官方文档+仓库 |
| **[lilycal-inventory.md](lilycal-inventory.md)** ⭐NEW 2026-08-19 | **lilycalInventory** - 非破坏衣柜/衣装切换工具（LI 组件族/物品切换/参数语义/NDMF 插件；v1.5.2） | lilxyzw 官方文档 |
| **[dressing-tools.md](dressing-tools.md)** ⭐NEW 2026-08-19 | **DressingTools (DT2)** - poi-vrc 非破坏衣橱/cabinet 装配系统（One-click Dressing/Cabinet 动画/骨骼映射；v2.6.0） | poi-vrc 官方文档 |
| **[gesture-manager.md](gesture-manager.md)** ⭐NEW 2026-08-19 | **Gesture Manager** - Unity 内 Avatar 动画预览/调试（simulator；证据须限定 Unity/GM preview；v3.9.8/3.9.9） | BlackStartx 官方 |
| **[unity-mcp.md](unity-mcp.md)** ⭐NEW 2026-08-19 | **Unity MCP (MCP for Unity)** - AI↔Unity Editor 桥（三层架构/UPM 安装/Editor 状态≠运行时证据；CoplayDev v10） | MCP for Unity 官方 |
| **[avatar-audit-methodology.md](avatar-audit-methodology.md)** ⭐NEW 2026-08-19 | **Avatar 审计方法论** - 7 级证据分层 + TOOLCHAIN_PROFILE + 玩家菜单优先审计 + 授权边界 | 本地整理+官方锚点 |

---

## 子域

| 子域 | 内容 | 状态 |
|------|------|------|
| **Animator** | Write Defaults、Avatar Mask、Playable Layers、参数类型、Direct Blend Tree | ✅ 已收录 |
| **Dynamics** | PhysBone, Contact, Constraint, VRC Constraints | ✅ 已收录 |
| Expression | Expression Menu, Expression Parameter | ✅ 已收录 (expression-menu.md + §9.5/§9.6 字段详解) |
| **Testing** | Avatar 3.0 Emulator (lyuma) | ✅ 已收录 (avatar-30-emulator.md) ⭐NEW 2026-07-04 |
| **Toggle Tools** | MA Object Toggle + 第三方工具生态 | ✅ 已收录 (modular-avatar.md + avatar-30-toggle-tools.md) ⭐NEW 2026-07-04 |
| **SDK** | Avatar SDK, Modular Avatar | ✅ 已收录 |
| **工具链与审计** | FaceEmo, lilycalInventory, DressingTools, Gesture Manager, Unity MCP | ✅ 已收录 ⭐2026-08-19 (faceemo.md / lilycal-inventory.md / dressing-tools.md / gesture-manager.md / unity-mcp.md) |
| **审计与证据** | avatar-audit-methodology.md — 7 级证据分层 + TOOLCHAIN_PROFILE + 授权矩阵 | ✅ 已收录 ⭐2026-08-19 |
| **Optimization** | 性能等级, VRAM, NDMF 工具, 最佳化步骤 | ✅ 已收录 |
| **Shader** | lilToon, Poiyomi, SCSS, Avatar Shader, 特殊效果 | ✅ 已收录 (liltoon/ + poiyomi/ + scss.md) |
| **Audio** | Avatar Audio 压缩格式/加载类型/量化数据/Load In Background | ✅ 已收录 (avatar-audio-optimization.md) ⭐NEW 2026-07-04 |
| **Size Limits** | VRAM/Uncompressed Size 区分 + 500MB/200MB 限制 + 时间线 | ✅ 已收录 (avatar-size-limits.md) ⭐NEW 2026-07-04 |

---

## 快速入门

### 懒人包

```
1. 复制 Avatar 的 FX Layer、Expression Menu、Expression Parameter
2. 删除所有跟开关物件相关的设定（保留表情 Blend Shape）
3. 复制 Avatar 本身
4. 导入 AAO + Meshia Mesh Simplification
5. 运行 AAO Trace And Optimize
6. 面数还太多 → Remove Mesh / Meshia 减面 / 脱掉
7. 把 FX、参数、选单塞回复制出来的 Avatar
```

### 性能目标

| 目标等级 | 面数 | 材质内存 | VRAM |
|----------|------|----------|------|
| Excellent | 32,000 | 40MB | < 40MB |
| Good | 70,000 | 75MB | < 75MB |
| Medium | 70,000 | 110MB | < 110MB |
| Poor | 70,000 | 150MB | < 150MB |

> 💡 **2026 推荐做法**：使用 Avatar Compressor (LAC) 自动压缩贴图达成 VRAM 目标，详见 `lac-avatar-compressor.md`

---

## 工具 VPM 链接速查

```
Modular Avatar:
  vcc://vpm/addRepo?url=https://vpm.nadena.dev/vpm.json

AvatarOptimizer (AAO):
  https://vpm.anatawa12.com/add-repo

Avatar Compressor (LAC):  ← ⭐NEW 2026-06-17
  https://vpm.limitex.dev/

Meshia Mesh Simplification:
  https://ramtype0.github.io/VpmRepository/index.json

TexTransTool:
  vcc://vpm/addRepo?url=https://vpm.rs64.net/vpm.json

MA2BT:  ← ⭐NEW 2026-06-17
  https://null-k.github.io/vpm-listing/index

Avatar 3.0 Emulator (Av3Emulator):  ← ⭐NEW 2026-07-04
  VPM: lyuma.av3emulator
  https://lyuma.github.io/Av3Emulator/vpm.json

FaceEmo:  ← ⭐NEW 2026-08-19
  https://suzuryg.github.io/vpm-repos/vpm.json

lilycalInventory:  ← ⭐NEW 2026-08-19
  https://lilxyzw.github.io/vpm-repos/vpm.json

DressingTools (DT2):  ← ⭐NEW 2026-08-19
  https://vpm.chocopoi.com/index.json
```

> ⚠️ `lilNDMFMeshSimplifier` 已于 2024-2025 被官方废弃，请改用 Meshia（详见 `meshia-mesh-simplification.md`）

---

## MA 与 VRCFury 兼容性

> ⚠️ 执行顺序错误会导致出错

```
1. MA + 其他新增内容 NDMF 工具
2. Fury
3. 所有 NDMF 最佳化工具
```

---

## 待收录内容

- [x] ~~Avatar Audio 空间音效最佳实践~~ → 已纳入 `avatar-audio-optimization.md` (2026-07-04)
- VRChat Avatar 官方文档深入研究
- Quest 兼容性约束表（细化）
- 常见 Avatar 审核失败原因
- 跨 Avatar 共享贴图策略

---

## 跨域引用（2026-07-04 更新）

> 以下文档虽位于 `platform/` 和 `sources/` 域，但与 Avatar 域紧密相关：

- `../platform/account-registration-cn.md` — VRChat 账号注册（国内网络环境）
- `../platform/unity-install-cn.md` — Unity Hub 国内安装指南
- `../platform/backup-strategy.md` — VRChat 创作者备份策略
- `../sources/vn3-license.md` — VN3 协议详解
- `../sources/blender-preprocessing.md` — Blender 预处理工具链（CATS + VRM）

> 上述文档由 86vrchat 笔记提炼（2026-07-04 纳入），详见 `../../../../（见源库 raw/ 目录）参考文献/86vrchat-对比分析/99-Memory纳入指南.md`
