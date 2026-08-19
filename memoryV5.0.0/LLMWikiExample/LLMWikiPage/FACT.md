---
title: FACT.md - VRChat 技术知识库
category: misc

knowledge_level: applied
status: active

tags:
  - misc
  - knowledge-graph
  - meta

aliases:
  - FACT
  - 知识库事实库
  - "Knowledge Base Facts"
  - 顶层索引

related:
  - "[[index]]"
  - "[[_always-load]]"
  - "源库：Sources 目录（见 raw/）"
  - [[concepts/patterns/index.md]]
  - [[entities/avatar/avatar-fallback-system.md]]
  - [[entities/avatar/avatar-dynamic-bone-limits.md]]
  - [[entities/avatar/avatar-particle-system-limits.md]]
  - [[entities/avatar/full-body-tracking.md]]
  - [[entities/avatar/ik-2.0.md]]
  - [[entities/avatar/shader/poiyomi/shader-locking-mechanism.md]]
  - [[entities/world/community-labs.md]]

source: 本地知识库整理
source_type: community
version: 3.3
last_review: 2026-08-20
confidence: High
type: summary
created: 2026-07-04
sources: 本地知识库整理
updated: 2026-08-20
---
# FACT.md - VRChat 技术知识库

> 长期有效的知识库结构、核心约束、设计模式索引与工程经验。
>
> **本文件保留原则**:仅存储长期有效的**知识性内容**(结构/约束/经验),不记录过程性整理日志。详细工作模式、阶段历史、规则沉淀等过程性产物见 `特殊Agent提示词/知识库整理Agent_V3.1.md`。

---

## 知识库结构

```
LLMWikiPage/                     # 知识页根（现行知识库）
├── FACT.md                      # 本文件：Fact Index（事实索引）
├── index.md                     # 顶层索引
├── _always-load.md              # 每会话必载
├── comparisons/                 # 对比分析（6 个文件）
├── concepts/                    # 概念域
│   ├── hybrid/                  # Hybrid/OSC 系统（12 个文件）
│   ├── misc/                    # 杂项概念（4 个文件）
│   ├── patterns/                # 核心设计模式（21 个文件）
│   └── rules/                   # UdonSharp 语言/工程规则（10 个文件）
├── entities/                    # 实体域
│   ├── api/                     # API 参考（19 个文件）
│   ├── avatar/                  # Avatar 域（63 个文件 + shader 子域）
│   │   └── shader/              # Shader 知识库
│   │       ├── filamented/      # Filamented PBR 着色器（3 个文件）
│   │       ├── liltoon/         # lilToon 详细文档（17 个文件）
│   │       ├── orl/             # ORL Shaders World 通用着色器（4 个文件）
│   │       ├── poiyomi/         # Poiyomi Shaders 8 主题知识（8 个文件）
│   │       └── unlitwf/         # UnlitWF Unlit 扩展专业效果（1 个文件）
│   ├── platform/                # 跨平台开发（12 个文件）
│   ├── vrchatsdk/               # VRChatSDK（HTTP API，19 个文件）
│   └── world/                   # World 域（19 个文件 + 子域）
│       ├── udon/                # Udon 域（18 个文件 + 深层子域）
│       │   ├── data-containers/ # 数据容器（7 个文件）
│       │   ├── graph/           # Udon Graph（6 个文件）
│       │   ├── midi/            # MIDI（3 个文件）
│       │   ├── networking/      # 网络同步（12 个文件，含 tools/）
│       │   ├── persistence/     # 持久化（5 个文件，含 patterns/）
│       │   ├── players/         # 玩家（7 个文件，含 drones/）
│       │   ├── udonsharp/       # UdonSharp（6 个文件）
│       │   ├── video-players/   # 视频播放器（3 个文件）
│       │   ├── vrc-graphics/    # VRC Graphics（4 个文件）
│       │   └── vrctween/        # VRCTween（7 个文件）
│       ├── scene-components/    # Scene Components（46 个文件）
│       ├── examples/            # SDK 内置示例（11 个文件）
│       │   ├── udon-example-scene/  # Udon Example Scene（6 个文件）
│       │   ├── obstacle-course/     # 障碍赛道示例（6 个文件）
│       │   └── persistence/         # 持久化示例（10 个文件）
│       ├── clientsim/           # ClientSim 模拟器（含 systems/）
│       ├── bakery/              # Bakery 光照烘焙（2 个文件）
│       ├── development/         # 开发相关内容（2 个文件）
│       ├── patterns/            # World 模式（2 个文件）
│       └── shader/              # World Shader（2 个文件）
├── meta/                        # 元协议/元数据（6 个文件）
└── queries/                     # 查询（1 个文件）

> 📌 raw 源材料位于工作区根 `memory/raw/`（哈希账本锚定、不可变，未随知识页发布包分发）；知识页根为 `LLMWikiExample/LLMWikiPage/`，`memory/raw/` 仅作不可变源材料。
```

---

## ⚠️ 核心约束（绝对规则）

### 渲染管线

> **🔴 最高优先级：VRChat 只支持 BRP (Built-in Rendering Pipeline)**

| 规则 | 说明 |
|------|------|
| **只支持 BRP** | VRChat **只支持** Built-in Rendering Pipeline |
| **禁止切换管线** | 任何更改渲染管线的行为都是**致命的** |
| **禁止 URP/HDRP** | 项目中不得使用 Universal 或 High Definition Render Pipeline |
| **Unity 版本绑定** | SDK 3.5.0+ 绑定 Unity 2022.3.22f1（LTS）；SDK 3.4.2 及更早绑定 Unity 2019.4.31f1 |

**违反后果**：切换渲染管线会导致 VRChat 项目完全无法工作。

### Editor 脚本与构建

> 🔴 **任何引用 `UnityEditor.*` 命名空间的脚本必须放在 `Editor` 文件夹内**

| 规则 | 说明 |
|------|------|
| **根因** | 脚本先在编辑器下编译（包含 `UnityEditor` 程序集），再为目标平台重新编译（**不包含** `UnityEditor` 程序集） |
| **触发条件** | `using UnityEditor;` 等编辑器命名空间 |
| **解决路径 1** | 将脚本放到**父目录为 `Editor`** 的任意路径下，构建时自动排除 |
| **解决路径 2** | 创建 asmdef，**Include Platforms** 仅勾选 Editor |

**常见场景**（VRChat World 开发中）：
- 自定义 Inspector (`[CustomEditor]`)
- Gizmo 调试 (`OnDrawGizmos`)
- Build Pipeline 后处理 (`IPreprocessBuildWithReport`)
- AssetPostprocessor
- AssetDatabase / EditorUtility / EditorPrefs 调用

**易错点**：
- ❌ 脚本必须挂在 GameObject 上才能编译 → 实际由**文件夹路径**决定
- ❌ 用 `Assets/Scripts/MyEditor.cs` → 跨平台构建时找不到 `UnityEditor` 程序集，报错
- ✅ 用 `Assets/Editor/MyEditor.cs` 或 `Assets/Scripts/Editor/MyEditor.cs` → 正常

**参考**：
- Unity 特殊文件夹：https://docs.unity3d.com/6000.3/Documentation/Manual/SpecialFolders.html

### 第三方工具链兼容性矩阵（2026-07-04 补充）

> 来源: VirtualLens2 + VRCLens 评估（外部验证）

| 工具 | VRCSDK 最低 | Unity 版本 | MA 最低 | 表情参数需求 | 平台 |
|------|-------------|------------|---------|--------------|------|
| **VRChat SDK 官方** | 3.5.0+ | 2022.3.22f1 LTS | N/A | N/A | 全平台 |
| **Modular Avatar (MA)** | 与 SDK 一致 | 2022 LTS | N/A | N/A | 全平台 |
| **VRCFury** | 与 SDK 一致 | 2022 LTS | N/A | N/A | 全平台 |
| **AvatarOptimizer (AAO)** | 与 SDK 一致 | 2022 LTS | 1.x | N/A | 全平台 |
| **VirtualLens2** | 3.7.0+ | 2022 LTS | 1.8.0+ | 8 bit | PCVR + Remote Mode (Android) |
| **VRCLens** | 3.0.8+ | 2022 LTS | (集成 AV3Manager) | 8 bit | Desktop + VR |
| **VVMW** | 与 SDK 一致 | 2022 LTS | N/A | N/A | World 工具 |
| **UVU** | 与 SDK 一致 | 2022 LTS | N/A | N/A | World 工具 |
| **Shader Locking 机制** | N/A (VRChat 强制) | 2022 LTS | N/A | N/A | Poiyomi Animated/Renamed, LilToon 需手动 Lock, d4rkAO 不支持 Renamed |

**关键约束总结**:
- 🔴 **Unity 2022 LTS** 是当前（2026-07）所有主流工具的最低要求
- 🔴 **VRCSDK 3.7.0+** 是 VirtualLens2 等新工具的硬性要求
- 🔴 **表情参数 8 bit** 是 VirtualLens2 / VRCLens 等摄影工具的硬性需求
- ⚠️ **MA 1.8.0+** 是 VirtualLens2 非破坏性工作流的依赖
- ⚠️ **Shader Locking**: VRChat 上传时自动 Lock 材质（剥离未用功能、烘焙常量、属性不可动画）。Poiyomi 提供 `Animated` / `Renamed` 标记让属性保持可动画；LilToon 无内置机制（需手动 Lock）。d4rkAvatarOptimizer **不支持** Poiyomi Renamed 模式（Issue #182 Open）

> 完整对比见 `entities/avatar/camera-tools.md` §1 / §2.1 / §3.1 + `entities/avatar/shader/poiyomi/shader-locking-mechanism.md`

### VRChat Avatar 相机已知问题（2026-07-04 补充）

> 🔴 **来源**: VRChat 官方 Canny feedback 平台
> 状态: tracked by Staff 但**未解决**

| 问题 | 影响 |
|------|------|
| **Avatar Camera 不响应 DepthTextureMode** | DoF 效果异常、shader 行为异常 |
| **DoF 噪声无法关闭** | 视频录制和后期降噪困难 |
| **Avatar 相机使用 stale depth texture** | 深度相关效果失效 |

> 详细分析见 `entities/world/vrc-camera-settings.md` §Avatar 相机限制

---

## 高优先级知识（已完成）

### Avatar Pipeline
- VRCPipelineManager ✅
- Avatar 3.0 四行架构 ✅

### VRC Constraints (Avatar 域)
- 6 种约束类型 ✅
- 高级设置（Local Space/Freeze To World）✅
- 性能分类（Count/Depth）✅
- Constraints API ✅

### Playable Layers
- Humanoid vs Generic ✅
- 5 层详解（Base/Additive/Gesture/Action/FX）✅
- Avatar Mask 规则 ✅
- T-Pose/IK Pose/Sitting Pose ✅
- **Avatar Animator = 参数驱动系统** ✅（核心：Expression Menu → Parameter → Playable Layer → Animator）

### Unity Animator（World 上下文）
- **World Animator = 逻辑驱动系统** ✅（核心：Udon Event → SetBool/SetTrigger → State Machine）
- Udon Animator API（SetBool/SetFloat/SetTrigger/SetInteger/Play/CrossFade）✅
- 网络同步注意事项 ✅

### World 性能优化
- 预算规划 ✅
- 材质/Shader/纹理管理 ✅
- 光照系统（烘焙/动态）✅
- 测试方法 ✅

### Bakery 光照烘焙 ✅
- 系统要求（Windows + NVIDIA Kepler+）✅
- 6 种 Render Mode ✅
- 5 种 Directional Mode ✅
- 8 种组件 ✅
- 材质兼容性 ✅
- FAQ + 故障排除 ✅

### OSC 协议
- 完整协议数据库 ✅

### 音频同步系统架构 ✅ (参考工程)
- **纹理编码数据传递**：音频数据通过 CustomRenderTexture 传递 Shader，零网络同步开销
- **时间同步系统**：Master 权威时间锚点 + 双时间源融合 + 漂移校正
- **同步策略**：Manual Sync + Owner Transfer 混合模式
- **位域压缩**：byte 存储多个 bool(`_flags`)→ 详见 `entities/world/udon/data-containers/byte-and-bit-operations.md`
- **性能热点**：SendAudioOutputData 每帧 8 次 SetFloatArray(约 2ms 开销)
- **实验性功能保护**：audioDataToggle 默认关闭(VRCAsyncGPUReadback 成本高)

> **📚 参考实现**:本节提炼自开源 VRChat 音频同步框架,见 （来源：Open Source Projects，本地整理） §音频同步类目

### 多机位导演系统网络同步架构 ✅ (参考工程)
- **5 层同步体系**：Manual + Continuous + NetworkCallable + SafeMod(排除) + NoVariableSync
- **NetworkCallable RPC**：参数化远程调用 + 批量延迟提交
- **双缓冲预览模式**：`_isPreviewActive` + `_serializationPending`
- **指数衰减插值**：Continuous 值平滑(Compensation 因子)
- **Slerp/Slerp 本地缓动**：位置+旋转独立插值
- **所有权分层**：操作者 vs 被跟踪者所有权分离
- **UI 转发模式**：NoVariableSync + SetProgramVariable

> **📚 参考实现**:本节提炼自开源 VRChat 多机位导演系统,见 （来源：Open Source Projects，本地整理） §多机位导演类目

---

## 核心设计模式与案例研究索引（精简版）

> 以下为知识库中已验证的核心设计模式和案例研究索引。
> 完整实现代码、详细架构分析、对比表格均在对应文档中，按需检索。

| 类别 | 数量 | 索引位置 |
|------|------|---------|
| 核心设计模式 | 14 个 | `concepts/patterns/index.md` + 各模式独立文档 |
| 玩家本地相机工具 | 2 个 (VirtualLens2 + VRCLens) | `entities/avatar/camera-tools.md` |
| **设计哲学：Non-Destructive vs Invasive** | 1 个模式 | `entities/avatar/camera-tools.md` §4 + `entities/avatar/vrcfury-reference.md` |
| **VRChat Avatar 相机已知问题** | 1 个主题 | `entities/world/vrc-camera-settings.md` §Avatar 相机限制 |
| **第三方工具链兼容性矩阵** | 1 个矩阵 | [[FACT]] §核心约束 + `entities/avatar/camera-tools.md` §1 |
| **Shader Locking 机制** | 1 个完整文档 | `entities/avatar/shader/poiyomi/shader-locking-mechanism.md` (2026-07-04) |
| LuraSwitch2 实战模式 | 6 个 | `concepts/patterns/index.md` #8-#13 |
| UdonVoiceUtils 工程化模式 | 10 个 | `concepts/patterns/index.md` #14-#23 |
| ULocalization 沙箱适配模式 | 5 个 | `concepts/patterns/index.md` #24-#28 |
| Sardinal 消息总线模式 | 3 个 | `concepts/patterns/index.md` #29-#31 |
| 案例研究 A1-A9 | 9 个 | （来源：Open Source Projects，本地整理） |
| 推荐工具 C1-C16 | 16 个 | `concepts/hybrid/udon-world-plugins.md` |
| Avatar 端摄影工具 C17 | 1 个 | `concepts/hybrid/avatar-camera-plugins.md` (Flex Fisheye Lens,2026-07-04) |

**快速查阅入口**：
- 网络同步：`concepts/patterns/manual-sync-state.md` / `advanced-sync-patterns.md`
- 沙箱适配：`concepts/patterns/hash-based-dispatch.md` / `build-time-vs-runtime-separation.md`
- 消息总线：`concepts/patterns/channel-routing.md` / `hybrid-subscription-modes.md`
- 视频同步：[VVMW（GitHub）](https://github.com/JLChnToZ/VVMW)（案例）/ `entities/world/vvmw.md`（工具）
- Shader 选型：`entities/avatar/shader/index.md`
- Avatar 摄影工具：`concepts/hybrid/avatar-camera-plugins.md`（Avatar 端鱼眼相机插件）
- VRChatSDK 概览：`entities/vrchatsdk/index.md`

---

## 知识完整性原则

> ⚠️ **所有知识库索引必须指向现行知识页根 `LLMWikiExample/LLMWikiPage/`**
> - 外部文档随时可能删除或丢失
> - 工具安装链接（VPM/BOOTH）可以保留（操作指引，非知识本身）
> - 知识来源路径必须本地化（`memory/raw/` 为不可变源材料）；raw 源材料与知识页根不在同一分发单元

---

- ⚠️ `vrchatsdk/`: 中英混合命名（建议未来统一）

### 关键工程 Specs（从 2026-06-10 知识库更新中沉淀）

> 这些规格是 VRChat 平台的硬性限制,不属于过程性记录,沉淀在此供长期参考。

#### VRCShader.SetGlobal 限制
- 属性名必须以 `_Udon` 为前缀(用于全局设置)
- 详见 [[entities/world/udon/vrc-graphics/index]]

#### Networking Specs
- **总带宽**: ~11 KB/s
- **Manual sync**: 280,496 bytes/serialization
- **Continuous sync**: ~200 bytes/serialization
- 详见 `entities/api/networking.md`

#### Persistence Compression
- 实际存储: 100 KB/player/world（压缩后）
- 可压缩数据: ~300 KB 原始数据
- 详见 `entities/api/persistence.md`

---

### 2026-07-04 新增参考工程

> 来源：86vrchat 笔记（VRCD 中文社区），P2 级知识点已纳入

#### 创作者工具链参考（86vrchat 视角）

- **国内创作者账号注册流程**：hCaptcha + VPN 检测 + Watt Toolkit 辅助
  - 详见 `entities/platform/account-registration-cn.md`

- **Unity Hub 国内安装**：国际版 vs 中国版 + NoUnityCN 第三方下载源
  - 详见 `entities/platform/unity-install-cn.md`

- **VN3 协议**：日韩圈 VRC 商业模型许可证框架
  - 详见 [VN3（官方）](https://www.vn3.org)

- **Blender 预处理工具链**：CATS（2021 停更，0.19.0）+ VRM Addon for Blender（活跃维护）
  - 详见 [CATS Blender 插件（GitHub）](https://github.com/absolute-quantum/cats-blender-plugin)

- **创作者备份策略**：UnityHub + VCC/ALCOM + VRChat 三个核心存储位置
  - 详见 `entities/platform/backup-strategy.md`

- **VRC 法律合规**：量贩模型法律红线 + Filian/竜胆案例（社区转述）
  - 详见 `entities/avatar/legal-and-licensing.md`

- **VRC 委托价格**：三档预算（低/中/高）+ 三种工具链对比
  - 详见 `entities/avatar/commission-pricing.md`

> **纳入决策**：见 `raw/参考文献/86vrchat-对比分析/99-Memory纳入指南.md`
> **标记体系**：[FACT-OFFICIAL] / [FACT-VN3] / [FACT-OFFICIAL-REPO] / [COMMUNITY-86vrchat] / [INFERENCE] / [UNKNOWN]

---

## 最低限度原则（SOUL / FACT 边界）

> 本文件(SOUL.md + FACT.md)仅存储**长期有效的最低限度原则**,不存储过程性工作协议。

| 内容类别 | 存储位置 | 理由 |
|---------|---------|------|
| 人格身份、语气、核心使命 | `SOUL.md` | 每个会话必须加载 |
| 知识库结构、核心约束、长期经验 | [[FACT]](本文件) | 每个会话必须加载 |
| 6 工作模式、Domain/Knowledge First、Knowledge Priority、Evidence、Failure Policy、Output、Retrieval、Entry Points、Ultimate Goal | `特殊Agent提示词/知识库整理Agent_V3.1.md` | 仅 Curator 模式按需参考 |
| 废弃文件处理规则、多文档同步 Checklist、Phase 历史、A 阶段记录 | `特殊Agent提示词/知识库整理Agent_V3.1.md` | 过程性/规则性,无需每次加载 |
