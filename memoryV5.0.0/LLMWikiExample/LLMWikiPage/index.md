---
title: "KB Index — Multi-Domain 路由地图"
category: misc
knowledge_level: applied
status: active
source: 本地知识库整理
source_type: community
version: 3.0
last_review: 2026-08-13
confidence: High
tags:
  - misc
  - index
  - navigation
aliases:
  - 知识库总入口
  - "Knowledge Base Index"
  - 导航
  - 总索引
related:
  - "[[FACT]]"
  - "[[_always-load]]"
  - [[entities/api/index.md]]
  - [[entities/avatar/index.md]]
  - [[entities/world/index.md]]
  - [[entities/world/udon/udonsharp-linter.md]]
  - [[entities/platform/index.md]]
  - [[concepts/hybrid/index.md]]
  - [[concepts/rules/index.md]]
  - [[concepts/rules/runtime-texture-compression.md]]
  - [[concepts/patterns/index.md]]
  - "源库：Sources 目录（见 raw/）"
  - "源库：References 目录（见 raw/）"
  - [[entities/vrchatsdk/index.md]]
  - [[meta/obsidian-governance]]
type: navigation
created: 2026-07-01
sources: 本地知识库整理
updated: 2026-08-13
---
# KB Index — Multi-Domain 路由地图

> Obsidian 总入口：本页保留原 Domain Router，同时新增可点击的 Obsidian MOC（Map of Content）。核心知识仍以 FACT / INFERENCE / UNKNOWN 分层，参考层与治理层按 [[meta/obsidian-governance]] 管理。

---

## Obsidian MOC — 可点击总入口

### 核心启动入口

| 入口 | 用途 |
|---|---|
| [[FACT|FACT.md - 知识库核心事实]] | 知识库结构、核心约束、长期经验 |
| [[_always-load|_always-load.md - 全领域核心约束]] | Agent 启动时的核心约束速查 |
| [[meta/obsidian-governance]] | Obsidian Vault 分层、排除规则、完整性验收 |

### 一级知识域

| Domain | 入口 | 说明 |
|---|---|---|
| World | [[entities/world/index.md]] | Udon、Networking、Persistence、World 性能、组件、示例 |
| Avatar | [[entities/avatar/index.md]] | Avatar 3.0、Animator、VRC Constraints、Shader、改模与优化 |
| API | [[entities/api/index.md]] | Networking、Persistence、Player API、Dynamics、Animator 等 API 参考 |
| Platform | [[entities/platform/index.md]] | Android / Quest / Cross-platform / Unity 工具链 |
| Hybrid | [[concepts/hybrid/index.md]] | OSC、AudioLink、Avatar↔World 边界与桥接 |
| Rules | [[concepts/rules/index.md]] | UdonSharp、Networking、Performance、VM、陷阱规则 |
| Patterns | [[concepts/patterns/index.md]] | 同步、状态机、消息总线、工程设计模式 |
| VRChatSDK | [[entities/vrchatsdk/index.md]] | HTTP API、WebSocket、数据模型 |
| Sources | 源资料归档层（不随发布包分发） | 官方/开源/工具来源与案例研究 |
| References | 源资料归档层（不随发布包分发） | 参考文献、对比分析、待晋升资料 |
| Misc | [[concepts/misc/index.md]] | 后处理、无障碍等杂项知识 |
| Comparisons | [[comparisons/index.md]] | 跨实体/主题对比分析（Shader、工具、架构） |
| Queries | [[queries/index.md]] | 优质查询答案归档 |
| Meta | [[meta/index]] | 知识库协议、工作模式、Obsidian 治理 |

### 参考与治理入口

| 层级 | 入口 | 说明 |
|---|---|---|
| Reference Notes | （来源：本地参考笔记） | 参考/整合资料，不直接等同核心事实 |
| VPM Mirrors | 源资料层（vpm-mirrors 镜像归档，不随发布包分发） | VPM 仓库镜像与样本来源 |
| Curator Tools | ~（2026-08-18 归档：``辅助文件`` 已移入 `辅助文件/报告/curator-tools/`，物理保留、不入 Git） | manifest、baseline、审计脚本与报告，默认不进入主知识图谱 |

---

## 启动协议

```
1. Domain Detection（确定领域）
2. 读本文件 → 确定相关域目录
3. 读 _always-load.md（30s 核心约束）
4. Goto 具体文件
```

## 按 Domain 路由

### World Domain ← 当前主力 ✅

| 需求 | 路径 |
|---|---|
| UdonSharp 代码 | `concepts/rules/udonsharp-language-limits.md` |
| **Udon 官方文档本地化** | `entities/world/udon/` ← ⭐ 11 个核心单页 + 子目录(2026-06-21 +VRCTween) |
| **Udon 总览** | `entities/world/udon/index.md` ← ⭐ 概念 + 3 种创建方式 + VM 关系 |
| **Udon 事件执行顺序** | `entities/world/udon/event-execution-order.md` ← ⭐ _onEnable→_start 无间隔 |
| **Udon Persistence 实战** | `entities/world/udon/persistence/` ← ⭐ 8 文档(PlayerData vs PlayerObject + 100KB + 3 实战模式)NEW 2026-06-21 |
| **Udon VM 字节码** | `entities/world/udon/vm-and-assembly.md` ← 9 Opcodes 官方规范 |
| **Udon 调试** | `entities/world/udon/debugging-udon-projects.md` |
| **Animation/Avatar/Input Events** | `entities/world/udon/animation-events.md` + `avatar-events.md` + `input-events.md` |
| **VRCTween(官方补间)** | `entities/world/udon/vrctween/index.md` ← ⭐ DOTween 封装 + 7 大类补间 + 虚拟补间 + 复用模式 ⭐NEW 2026-06-21 (2026-06-22 已校验与官方 5-28 一致) |
| **VRCTween 原始英文 / DOTween 底层** | VRCTween 官方原始快照（源资料层，不随发布包分发） |
| **Image/String Loading** | `entities/world/udon/image-loading.md` + `string-loading.md` + `external-urls.md` |
| **AI Navigation** | `entities/world/udon/ai-navigation.md` ← NavMeshAgent + Owner 计算 |
| **Using Build & Test** | `entities/world/udon/using-build-test.md` ← 本地测试 |
| **Udon 版主工具** | `entities/world/udon/udon-moderation-tool-guidelines.md` |
| 网络同步 | `concepts/rules/networking-rules.md` → `concepts/patterns/manual-sync-state.md` |
| 性能优化 | `concepts/rules/performance-rules.md` → `concepts/rules/udon-vm-architecture.md` |
| 陷阱排查 | `concepts/rules/udonsharp-deep-pitfalls.md`（源资料层 common-failures，不随发布包分发） |
| API 参考 | `entities/api/` 目录（grep 关键词） |
| **VRCBillboard（原生公告板）** | `entities/api/vrc-billboard.md` ← ⭐ 2026-08-13 官方预告（PLANNED）：`Register`/`Unregister`，原生替代 update loop |
| 设计模式 | `concepts/patterns/` 目录 |
| 脚本审查 | 源资料层 review-checklist（不随发布包分发） |
| 后处理 | `concepts/misc/postprocessing-principles.md` → `concepts/misc/postprocessing-usage.md` |
| **光照系统** | `entities/world/vrc-light-volumes.md` ← ⭐ 体素光照 + Point Light Volumes |
| **光照烘焙** | `entities/world/bakery/light-baking-guide.md` ← ⭐ Bakery + Glim 烘焙指南 |
| **遮挡剔除** | `entities/world/occlusion-culling-guide.md` ← 性能优化 |
| **反射探针** | `entities/world/reflection-probes.md` ← 光照系统 |
| **World Shader** | `entities/world/shader/` ← ⭐ Graphlit 节点编辑器 |
| **VRCGraphics** | `entities/world/udon/vrc-graphics/index.md` ← ⭐ 着色器全局变量 |
| **VRCCameraSettings** | `entities/world/vrc-camera-settings.md` ← ⭐ 屏幕/手持相机 + VR 双眼 ⭐NEW |
| **VRCQualitySettings** | `entities/world/vrc-quality-settings.md` ← ⭐ 阴影距离覆盖 + 质量只读 ⭐NEW |
| **Data Containers** | `entities/world/udon/data-containers/index.md` ← ⭐ JSON 序列化 |
| **Data Tokens** | `entities/world/udon/data-containers/data-tokens.md` ← DataToken 类型系统 + 隐式装箱 |
| **Data Lists** | `entities/world/udon/data-containers/data-lists.md` ← DataList 完整 API + 性能基准 |
| **Data Dictionaries** | `entities/world/udon/data-containers/data-dictionaries.md` ← DataDictionary 完整 API + 键值对 |
| **VRCJSON** | `entities/world/udon/data-containers/vrcjson.md` ← JSON 序列化/反序列化 + 错误处理 |
| **Creator Economy** | `entities/world/creator-economy.md` ← ⭐ 产品集成 |
| **Community Labs(World 发布)** | `entities/world/community-labs.md` ⭐NEW 2026-06-30 ← Stage 2 + 每周 1 个 + Trust Rank User |
| **Udon Example Scene** | `entities/world/examples/udon-example-scene/` ← ⭐SDK 内置标准示例(13+ Prefab)⭐NEW |
| **Obstacle Course** | `entities/world/examples/obstacle-course/` ← ⭐World Jam 2 官方套件(Time Trial 模式 + 6 文档)⭐NEW 2026-06-15 |
| **LuraSwitch2 通用开关套件 (v1.06)** | `entities/world/luraswitch2.md` ← ⭐ 9+ Prefab + 3 同步模式 + PC/Quest 自动适配 + VN3 License 免费商用 + Slider_Video/Slider_Night/PlatformOverride ⭐REWRITTEN 2026-06-21 (2026-03-06 最新版) |
| **VizVid (VVMW) 视频播放器** | `entities/world/vvmw.md` ← ⭐ 5 大模块 + 9 大集成 + 13 个 Releases + PC/Quest 跨平台 + 12+ 语言本地化 ⭐NEW 2026-06-20 (A2/C16 双重身份) |
| **Companions vs Props (Items)** | `entities/world/companions.md` ⭐NEW 2026-06-30 ← Items 分裂(2026.2.3) + World Creator 视角 |
| **Steam Audio (替换 ONSP)** | `entities/world/audio-steam.md` ⭐NEW 2026-06-30 ← 2025.4.2+ 音频系统迁移 + Voice 滑块 65% 等效 |

### Platform Domain ← ✅ 新增

| 需求 | 路径 |
|---|---|
| Android/Quest 开发 | `entities/platform/android-development.md` |
| 跨平台内容 | `entities/platform/cross-platform-content.md` |
| 移动端 UI | `entities/platform/mobile-ui-optimization.md` |
| **Unity Recorder 截图** | `entities/platform/unity-recorder.md` ⭐NEW 2026-07-04 (Editor-only + VRChat Unity LTS 兼容) |
| **Unity 管理员警告** | `entities/platform/unity-admin-warning.md` ⭐NEW 2026-07-04 (UAC + 兼容性 + UUM-42508) |

### VRChatSDK Domain ← ✅ 已本地化

| 需求 | 路径 |
|---|---|
| SDK 概览 | `entities/vrchatsdk/homepage.md` |
| 认证 API | `entities/vrchatsdk/api-authentication.md` |
| 用户/世界/Avatar API | `entities/vrchatsdk/api-{users,worlds,avatars}.md` |
| 好友/通知/收藏 API | `entities/vrchatsdk/api-{friends,notifications,favorites}.md` |
| 实例/文件/群组 API | `entities/vrchatsdk/api-{instances,files,groups}.md` |
| WebSocket 实时事件 | `entities/vrchatsdk/websocket-api.md` |
| 数据模型 | `entities/vrchatsdk/model-{user,current-user}.md` |

### Sources Domain ← ✅ 新增

| 需求 | 路径 |
|---|---|
| Example Central | `源资料层《example-central》` |
| **VPM Package Template** | `源资料层《vpm-package-template》` ← ⭐NEW |
| **HoshinoLabs ULocalization 源(A8)** | `源资料层《ulocalization》` ← ⭐NEW 2026-06-20 |
| **HoshinoLabs Sardinal 源(A9)** | `源资料层《sardinal》` ← ⭐NEW 2026-06-20 (通用消息总线) |
| **Unity 动画路径重映射** | `源资料层《unity-animation-hierarchy-editor》` ⭐NEW 2026-07-04 (主推 hfcRed/Animation-Repathing) |

### Avatar Domain ← ✅ 已建设

| 需求 | 路径 |
|---|---|
| **Animator 系统** | `entities/avatar/animator-system.md` ← ⭐ 参数驱动系统 |
| **VRC Constraints** | `entities/avatar/vrc-constraints.md` ← ⭐ 6 种约束类型 |
| **PhysBones** | `entities/avatar/physbones.md` ← ⭐ VRCPhysBone 组件参考(Avatar+World) ⭐2026-08-19 |
| **Playable Layers** | `entities/avatar/playable-layers.md` ← ⭐ 5 层详解 |
| 性能等级标准 | `entities/avatar/performance-rank.md` |
| NDMF 工具生态 | `entities/avatar/ndmf-tools.md` |
| 最佳化实操指南 | `entities/avatar/optimization-guide.md` |
| **Modular Avatar** | `entities/avatar/modular-avatar.md` ← ⭐ VRChat改模必备插件(§0 速查 40 条 + §9 高级) |
| **MA 组件教学卡** | `entities/avatar/ma-component-cards.md` ← ⭐ 28 个组件完整教学卡(含 Global Collider) |
|| **玩家本地相机工具** | `entities/avatar/camera-tools.md` ⭐NEW 2026-07-04 ← VirtualLens2 + VRCLens + 工具设计哲学对比 |
|| **VirtualLens2 子工具** | `entities/avatar/camera-tools-virtuallens2.md` ⭐NEW 2026-07-13 ← 安装/依赖/Remote/Drone/Focus/Non-Destructive 细节 |
|| **VRCLens 子工具** | `entities/avatar/camera-tools-vrclens.md` ⭐NEW 2026-07-13 ← Apply/原相机前提/HDR/White Balance/Invasive 细节 |
|| **Avatar Fallback System** | `entities/avatar/avatar-fallback-system.md` ⭐NEW 2026-06-30 ← 5 原因 + 上传流程 + Grandfathered |
| **用户端 Dynamic Bone Limits** | `entities/avatar/avatar-dynamic-bone-limits.md` ⭐NEW 2026-06-30 ← 默认 32/8 + config.json |
| **用户端 Particle System Limits** | `entities/avatar/avatar-particle-system-limits.md` ⭐NEW 2026-06-30 ← 11 变量 + Quest 不可禁用 |
| **Full-Body Tracking** | `entities/avatar/full-body-tracking.md` ⭐NEW 2026-06-30 ← 8 tracker + Rigging 要求 |
| **IK 2.0** | `entities/avatar/ik-2.0.md` ⭐NEW 2026-06-30 ← Lock Types + 启动参数 |
| **Avatar 交互权限** | `entities/avatar/avatar-interaction-permissions.md` ⭐NEW 2026-06-30 |
| **Public Avatar Cloning** | `entities/avatar/public-avatar-cloning.md` ⭐NEW 2026-06-30 |
| **Avatar Accessories** | `entities/avatar/accessories.md` ⭐NEW 2026-06-30 ← 配件系统(2026.2.1+) + Look Editor 整合 + PC=240/Quest=80 |
| **VRCRaycast (Avatar)** | `entities/avatar/vrcraycast.md` ⭐NEW 2026-06-30 ← 骨骼 raycast + 2026.2.1 自动剥离 |
| **Blender 形态键** | `entities/avatar/blender-shape-keys.md` ⭐NEW 2026-07-04 ← FBX 导出 + Unity 导入 + 4 步避坑 |
| **Shader 索引** | `entities/avatar/shader/index.md` ← ⭐ 完整对比矩阵 + 选择指南 |
| **Shader (lilToon)** | `entities/avatar/shader/liltoon/` ← ⭐ 16 个文档，30%+ VRChat 使用 |
| **Shader (Poiyomi)** | `entities/avatar/shader/poiyomi/` ⭐NEW 2026-07-01 ← 8 主题知识 + 65 原始文档(VRChat 主流 Toon,5 变体 + 9 Lighting Type + Modular Pro) |
| **Shader (SCSS)** | `entities/avatar/shader/scss.md` ← ⭐ 双阴影系统 + UV 服装切换 |
| **Shader (ORL)** | `entities/avatar/shader/orl/` ← ⭐ World 通用模块化着色器（4个文档）|
| **Shader (Filamented)** | `entities/avatar/shader/filamented/` ← ⭐ Google Filament PBR 替代（4个文档）|
| **Shader (UnlitWF)** | `entities/avatar/shader/unlitwf/` ← ⭐ Unlit 扩展专业效果（Fur/Water/Grass/Gem）|
| **Shader (其他)** | `entities/avatar/shader/other-shaders.md` ← Graphlit 等 |
| 入口文件 | `entities/avatar/index.md` |

### Hybrid Domain ← ✅ 已建设

| 需求 | 路径 |
|---|---|
| **VCC (Creator Companion)** ⭐NEW 2026-07-01 | `concepts/hybrid/vcc.md` ← 架构/工作流/VPM格式/已知问题/CLI |
| **ALCOM (替代品)** ⭐NEW 2026-07-01 | `concepts/hybrid/alcom.md` ← 跨平台/并行下载/vrc-get CLI |
| **OSC 系统** | `concepts/hybrid/osc-protocol.md` ← ⭐ 完整协议数据库 |
| **AudioLink 系统** | `concepts/hybrid/audio-link.md` ← ⭐ 音频可视化核心 |
|| **Avatar↔World 总纲** | `concepts/hybrid/avatar-world-boundaries.md` ← ⭐ 内部参数边界 / 外显观察层 / 正式桥接层 / 同步层 |
|| **玩家/Avatar 外显观察边界** | `concepts/hybrid/player-observation-boundaries.md` ← Position / Tracking / Bone / Eye Height / Scaling |
|| **Player↔World Trigger 桥** | `concepts/hybrid/player-world-trigger-bridge.md` ← Trigger / Collision / Particle 观测桥 |
|| **Avatar↔World Contact 桥** | `concepts/hybrid/avatar-world-contact-bridge.md` ← Contact / Dynamics 桥接边界 |
|| **Avatar↔World 同步模型** | `concepts/hybrid/avatar-world-sync-model.md` ← Event / Owner / `[UdonSynced]` / Late Joiner |
|| 设计模式借鉴 | [[FACT]] → 核心设计模式 section |

---

## World Domain 文件清单

```
concepts/rules/
  udonsharp-language-limits.md    ← C# 限制 (13 rules)
  networking-rules.md             ← 网络同步 (22 rules)
  performance-rules.md            ← 性能优化 (12 rules)
  multi-vm-rules.md               ← 多 VM 协作 (9 rules)
  udon-vm-architecture.md         ← VM 逆向 (9 指令/EXTERN/Heap)
  udonsharp-deep-pitfalls.md      ← 深度陷阱 (19 rules)
  vrchat-api-exposure.md          ← API 暴露判断

concepts/patterns/
  owner-authoritative-interaction.md
  manual-sync-state.md
  late-joiner-state-restore.md
  bit-packed-flags.md
  event-driven-state-machine.md
  advanced-sync-patterns.md
  unorthodox-patterns.md
  master-follower-syncer.md        ⭐NEW 2026-06-20
  exclusive-control-selector.md    ⭐NEW 2026-06-20
  soft-detent-interpolation.md     ⭐NEW 2026-06-20
  fade-then-snap.md                ⭐NEW 2026-06-20
  editor-preview-component.md      ⭐NEW 2026-06-20
  material-propertyblock-safe-update.md ⭐NEW 2026-06-20

entities/api/
  networking.md  player-api.md  persistence.md  dynamics.md
  animator.md    audio.md       pickups.md      ui.md
  vrchat-advanced-apis.md  official-doc-clarifications.md
  events-reference.md
  udon-type-exposure.md     ← Udon Type Exposure Tree 索引 ⭐NEW
  exposed-types.md         ← 已暴露类型详细清单 ⭐NEW
  not-exposed.md           ← 未暴露 API 黑名单 ⭐NEW
  api-checker.md           ← API 检查器/代码模式 ⭐NEW

entities/world/
  vrc-camera-settings.md      ← VRCCameraSettings（屏幕/手持相机 + VR 双眼）⭐NEW
  vrc-quality-settings.md     ← VRCQualitySettings（阴影距离覆盖 + 质量只读）⭐NEW
  creator-economy.md          ← 创作者经济 SDK
  performance-guide.md        ← World 性能优化
  vrc-light-volumes.md        ← VRCLightVolumes 工具
  occlusion-culling-guide.md  ← 遮挡剔除优化
  reflection-probes.md        ← 反射探针系统
  udon/                       ← Udon 编程完整子目录(可视化 + C# 入口) ⭐NEW 2026-06-15
    graph/                    ← Udon Node Graph(可视化编程)⭐NEW 2026-06-15
      index.md                ← 主页:Interface/Flow/创建节点/编译/运行
      event-nodes.md           ← Event 节点(Interact/Player/Video/...)⭐NEW
      graph-elements.md       ← Graph 元素(Groups/Comments/Noodles)⭐NEW
      searching-for-nodes.md   ← 5 种节点搜索方式 ⭐NEW
      special-nodes.md         ← 特殊节点(Block/Branch/Loop/Variable/...)⭐NEW
      type-nodes.md            ← 类型引用节点(VRCPickup/PlayerApi/...)⭐NEW
    udonsharp/                ← UdonSharp C# 入口子目录
    players/                  ← Players API 子目录
    ui-events.md              ← UI Events(Input Events 单独页)
    using-build-test.md       ← Build & Test 本地测试
    vm-and-assembly.md        ← Udon VM 与 Udon Assembly
    world-debug-views.md      ← World Debug Views
    udon-moderation-tool-guidelines.md
    index.md                  ← Udon 总览(3 种创建方式 + VM 关系)⭐NEW 2026-06-15
    ai-navigation.md          ← AI Navigation(NavMeshAgent + Owner 计算)⭐NEW 2026-06-15
    animation-events.md       ← Animation Events 白名单(12 个方法)⭐NEW 2026-06-15
    avatar-events.md          ← Avatar Events(OnAvatarChanged/EyeHeight)⭐NEW 2026-06-15
    debugging-udon-projects.md← Debugging(Debug 模式 + 错误日志)⭐NEW 2026-06-15
    event-execution-order.md  ← ⭐事件执行顺序(_onEnable→_start 无间隔)⭐NEW 2026-06-15
    external-urls.md          ← External URLs(VRCUrl + 域白名单)⭐NEW 2026-06-15
    image-loading.md          ← Image Loading(VRCImageDownloader)⭐NEW 2026-06-15
    input-events.md           ← Input Events(Button/Axis + UdonInputEventArgs)⭐NEW 2026-06-15
    string-loading.md         ← String Loading(VRCStringDownloader)⭐NEW 2026-06-15
    persistence/              ← ⭐Persistence 实战教程子目录(8 文档)NEW 2026-06-21
      player-data.md          ← PlayerData 完整 API(20+ 方法)
      player-object.md        ← PlayerObject + VRCEnablePersistence 设置
      serialization.md        ← 18 种数据类型 + byte[] 编码模式
      limits-and-quirks.md    ← 100KB 限制 + 8 个陷阱
      concepts/patterns/               ← 3 个实战 Pattern
        leaderboard.md        ← 排行榜
        unlock-items.md       ← 解锁物品
        persistent-pen.md     ← 持久画笔
    vrctween/                 ← ⭐VRCTween 官方补间系统(7 大类 + 虚拟补间)NEW 2026-06-21 (2026-06-22 校验)
      index.md                ← 总览 + 何时使用 + 关键事实(本地性/输入验证/性能) + 底层实现(DOTween)
      basics.md               ← 基础 API(TweenXxx + VirtualTween + Ease)
      tween-types.md          ← 7 大类补间完整列表
      control.md              ← Play/Pause/Kill + 设置 + 复用模式
      sequence.md             ← 序列与并行(OnComplete 链,无原生 Sequence)
      custom-tween.md         ← 自定义虚拟补间 + AnimationCurve 缓动
      patterns.md             ← 12 个实战模式(按钮/UI/相机/Late joiner)
  源资料原始快照（外部资料，不随发布包分发）
    VRCTween-Official-Overview.md         ← 官方主页 2026-05-28
    VRCTween-Official-Tween-Types.md      ← 官方 tween-types
    VRCTween-Official-Virtual-Tweens.md   ← 官方 virtual-tweens
    VRCTween-Official-Settings.md         ← 官方 settings
    DOTween-Underlying-Engine.md          ← DOTween 底层库介绍
  items.md                    ← Items in Udon Worlds (Layer 3 + Udon 不可引用) ⭐NEW 2026-06-15
  layers.md                   ← Unity Layers (32 Layer 分配 + Physics 安全模式) ⭐NEW 2026-06-15
  sdk-prefabs.md              ← SDK Prefabs (8 个 Udon 示例 Prefab) ⭐NEW 2026-06-15
  supported-assets.md         ← Supported Scripted Assets (TMP/PPSv2/FinalIK/DynamicBone) ⭐NEW 2026-06-15
  whitelisted-world-components.md  ← 完整 Script 白名单 (10 个组件集合) ⭐NEW 2026-06-15
  companions.md               ← ⭐Companions vs Props (Items 分裂 2026.2.3) NEW 2026-06-30
  audio-steam.md              ← ⭐Steam Audio 替换 ONSP (2025.4.2+) NEW 2026-06-30
  scene-components/           ← ⭐Scene Components 子分类(VRC 组件 9 + Unity 原生 24)⭐NEW 2026-06-15 / 2026-08-17 扩充
    index.md                  ← Scene Components 总览(组件索引 + PhysBone/Contact 1024 限制)
    textmeshpro.md            ← TextMesh Pro 集成/字体/Fallback 字体/本地化
    vrc-avatarpedestal.md     ← Avatar Pedestal 展示与切换(Blueprint ID + Scale)
    vrc-cameradolly.md        ← Camera Dolly 相机轨道(Animation/Path/Point 3 层)
    vrc-mirrorreflection.md   ← Mirror Reflection 实时反射(Quest 性能警告)
    vrc-objectsync.md         ← ⭐Object Sync 物理对象同步(SetKinematic/SetGravity/Respawn)
    vrc-portalmarker.md       ← Portal Marker 传送门(World ID/None/Home/Hub)
    vrc-scenedescriptor.md    ← ⭐Scene Descriptor World 必含核心(Spawn/Respawn/Order)
    vrc-station.md            ← Station 玩家固定座位(OnStationEntered + Avatar Station)
    vrc-enablepersistence.md  ← Enable Persistence 持久化(PlayerObject + 100KB 限制)
    rigidbody.md              ← ⭐Unity 原生:Rigidbody 刚体(VRC Pickup 依赖/isKinematic 同步坑) NEW 2026-08-17
    boxcollider.md            ← Unity 原生:BoxCollider NEW 2026-08-17
    capsulecollider.md        ← Unity 原生:CapsuleCollider NEW 2026-08-17
    spherecollider.md         ← Unity 原生:SphereCollider NEW 2026-08-17
    meshcollider.md           ← Unity 原生:MeshCollider(medium) NEW 2026-08-17
    characterjoint.md         ← Unity 原生:CharacterJoint(medium) NEW 2026-08-17
    configurablejoint.md      ← Unity 原生:ConfigurableJoint(medium) NEW 2026-08-17
    particlesystem.md         ← ⭐Unity 原生:ParticleSystem(Quest/ps_* 限制) NEW 2026-08-17
    particlesystemforcefield.md ← Unity 原生:PSForceField(build 904 World 白名单/Tupper 确认) NEW 2026-08-17
    light.md                  ← ⭐Unity 原生:Light(烘焙优先/粒子挂光陷阱) NEW 2026-08-17
    lightprobegroup.md        ← Unity 原生:LightProbeGroup NEW 2026-08-17
    linerenderer.md           ← Unity 原生:LineRenderer(笔刷/轨迹) NEW 2026-08-17
    trailrenderer.md          ← Unity 原生:TrailRenderer NEW 2026-08-17
    meshrenderer.md           ← Unity 原生:MeshRenderer NEW 2026-08-17
    skinnedmeshrenderer.md    ← Unity 原生:SkinnedMeshRenderer(BlendShape) NEW 2026-08-17
    audiosource.md            ← ⭐Unity 原生:AudioSource(VRC_SpatialAudioSource 配置) NEW 2026-08-17
    audioreverbzone.md        ← Unity 原生:AudioReverbZone NEW 2026-08-17
    audiolowpassfilter.md     ← Unity 原生:AudioLowPassFilter(AVPro 失效坑) NEW 2026-08-17
    navmeshagent.md           ← ⚠️Unity 原生:NavMeshAgent(2022 兼容坑/时效) NEW 2026-08-17
    navmeshobstacle.md        ← ⚠️Unity 原生:NavMeshObstacle(时效) NEW 2026-08-17
    offmeshlink.md            ← ⚠️Unity 原生:OffMeshLink(时效) NEW 2026-08-17
    videoplayer.md            ← Unity 原生:VideoPlayer(VOD/URL 白名单) NEW 2026-08-17
    canvas.md                 ← ⭐Unity 原生:Canvas(VRC_UIShape 交互条件) NEW 2026-08-17
    canvasgroup.md            ← Unity 原生:CanvasGroup NEW 2026-08-17
  concepts/patterns/                 ← ⭐NEW 2026-07-04 World 域设计模式库
    index.md                ← World Patterns 索引(UI / Player / Scene / Audio)
    dynamic-player-list.md  ← ⭐NEW 2026-07-04 动态生成玩家传送按钮模式
  bakery/                     ← Bakery 光照烘焙
    index.md
    light-baking-guide.md      ← Bakery + Glim 烘焙指南
  shader/                     ← World Shader 知识库
    index.md
    graphlit.md               ← Graphlit 节点编辑器
  examples/                   ← SDK 内置示例 ⭐NEW
    udon.md                   ← Udon 基础 4 模式 (旋转/交互/传送/事件) ⭐NEW 2026-06-15
    ai-navigation.md          ← AI Navigation 寻路 + Manual Sync ⭐NEW 2026-06-15
    detect-controller-collide.md ← OnControllerColliderHit + HitPlayer ⭐NEW 2026-06-15
    image-loading.md          ← VRCImageDownloader + GitHub Pages ⭐NEW 2026-06-15
    midi-playback.md          ← VRCMidiPlayer + Note On/Off 12 音位 ⭐NEW 2026-06-15
    minimap.md                ← VRCGraphics.Blit + RenderTexture + Shader ⭐NEW 2026-06-15
    mute-others.md            ← SetVoiceDistanceFar(0) 模拟静音 ⭐NEW 2026-06-15
    player-join-zones.md      ← DataList + FieldChangeCallback + 3 Mode 状态机 ⭐NEW 2026-06-15
    screen-canvas.md          ← IsUserInVR + VRC_UIShape + TeleportTo ⭐NEW 2026-06-15
    obstacle-course/          ← ⭐Obstacle Course(World Jam 2 官方套件)NEW 2026-06-15
      index.md                ← Overview + Quick Start + Player/PowerUp 配置
      build-from-demo-parts.md
      build-from-custom-parts.md
      uoc-flythrough.md
      uoc-how-stuff-works.md
      uoc-window.md
    persistence/              ← Persistence Examples 子目录(Example Central 9 示例)
    udon-example-scene/       ← Udon Example Scene(13+ Prefab)
      index.md                ← 主页 13+ Prefab 详解 + 5 种同步模式
      avatar-scaling-settings.md
      player-mod-setter.md
      simple-pen-system.md
      udon-video-sync-player.md
      world-audio-settings.md

源资料 review 快照（外部，不随发布包分发）
  review-checklist / common-failures / severity-model（原始快照）

concepts/misc/
  postprocessing-principles.md    ← 后处理原理（现实参照）
  postprocessing-usage.md        ← Unity 参数与最佳实践
  accessibility-guide.md         ← 无障碍设计指南

entities/platform/                      ← ⭐NEW 跨平台开发
  android-development.md
  cross-platform-content.md
  mobile-ui-optimization.md
  easyquestswitch.md
  unity-recorder.md            ← ⭐NEW 2026-07-04 Unity 截图/录屏工具
  unity-admin-warning.md       ← ⭐NEW 2026-07-04 Unity 管理员警告排查

源资料 source 快照（外部，不随发布包分发）
  example-central.md           ← Example Central 使用
  vpm-package-template.md      ← VPM Package 开发模板 ⭐NEW
  clientsim.md                  ← ClientSim 编辑器模拟工具
  open-source-projects.md      ← 开源项目参考
  quickbrown-luraswitch2.md    ← QuickBrown LuraSwitch2 案例研究 ⭐NEW 2026-06-20
  udonvoiceutils.md            ← TLP UdonVoiceUtils 案例研究 ⭐NEW 2026-06-20
  ulocalization.md             ← HoshinoLabs ULocalization 案例研究 ⭐NEW 2026-06-20
  sardinal.md                  ← HoshinoLabs Sardinal 案例研究 ⭐NEW 2026-06-20 (通用消息总线)
  unity-animation-hierarchy-editor.md ← ⭐NEW 2026-07-04 动画路径重映射工具

entities/vrchatsdk/                     ← VRChatSDK HTTP API (18 个文档) ✅
  index.md                    ← VRChatSDK 总览 + 端点分类 ⭐NEW 2026-06-15
```

### 域索引文件总览(2026-06-15 二次审计补全)

| 域 | index.md | 文件数 | 状态 |
|---|---------|-------|------|
| `entities/world/udon/` | ✅ | ~60 | 已就位 |
| `entities/world/scene-components/` | ✅ | 35 | 已就位(2026-08-17 +24 Unity 原生组件) |
| `entities/world/bakery/` | ✅ | 2 | 已就位 |
| `entities/world/clientsim/` | ✅ | 8 | 已就位 |
| `entities/world/shader/` | ✅ | 2 | 已就位 |
| `entities/world/examples/` | ✅ | 31 | 标准 `index.md` + README 详细页并存 |
| `entities/avatar/` | ✅ | 9 | 已就位 |
| `entities/avatar/shader/` | ✅ | 4 + 子目录 | 已就位 |
| `entities/avatar/shader/liltoon/` | ✅ | **17**(新增 outline.md)| ✅ 已修复 outline 缺失 |
| `entities/avatar/shader/orl/` | ✅ | 5 | 已就位 |
| `entities/avatar/shader/filamented/` | ✅ | 4 | 已就位 |
| `entities/avatar/shader/unlitwf/` | ✅ | 1 | 已就位 |
| `concepts/hybrid/` | ✅ | 3 | 已就位 |
| **`entities/vrchatsdk/`** | ✅ ⭐NEW | 19(+1) | **二次审计补全** |
| **`entities/platform/`** | ✅ ⭐NEW | 6(+2) | **二次审计补全** ⭐2026-07-04 +2 (Recorder + Admin) |
| **`concepts/misc/`** | ✅ ⭐NEW | 4(+1) | **二次审计补全** |
| **`源资料 reference 快照`** | ✅ ⭐NEW | 2(+1) | **二次审计补全** |
| `源资料 source 快照` | ✅ | 12 (+2) | 已就位(2026-06-20 +1 Sardinal, 2026-07-04 +1 AnimationHierarchyEditor) |
| `concepts/rules/` | ✅ | 8 | 已就位 |
| `concepts/patterns/` | ✅ | 31 (+17) | 已就位(2026-06-20 +3 Sardinal 独有) |
| `entities/world/patterns/` | ✅ ⭐NEW 2026-07-04 | 1 (+1) | **新建子目录**(dynamic-player-list) |
| `entities/api/` | ✅ | 17 | 已就位 |
| 源资料 review 快照 | ✅ | 4 (32 FAILs) | 已就位(2026-06-20 +4 FAILs) |
| `meta/` | ✅ | 4 | 工作模式、KB维护协议、网络搜索准入协议已就位 |

## VRChatSDK Domain 文件清单

```
entities/vrchatsdk/
  homepage.md                  ← 层级概览
  typescript-sdk.md            ← SDK 多语言支持
  websocket-api.md             ← WebSocket 实时事件
  instances.md                 ← 实例管理
  tags.md                      ← 标签系统
  faq.md                       ← 常见问题
  api-authentication.md        ← 认证 API
  api-users.md                 ← 用户 API
  api-worlds.md                ← 世界 API
  api-avatars.md               ← Avatar API
  api-friends.md               ← 好友 API
  api-notifications.md         ← 通知 API
  api-favorites.md             ← 收藏 API
  api-instances.md             ← 实例 API
  api-files.md                 ← 文件 API
  model-user.md                ← User 数据模型
  model-current-user.md        ← CurrentUser 数据模型
  api-groups.md                ← 群组 API
```

## Avatar Domain 文件清单

```
entities/avatar/
  index.md                     ← 域名索引
  animator-system.md           ← Write Defaults/Avatar Mask/Playable Layers
  vrc-constraints.md           ← VRC Constraints 6 种类型
  playable-layers.md           ← 5 层详解
  performance-rank.md          ← PC/Quest 性能等级
  performance-benchmarks.md    ← 性能基准测试
  ndmf-tools.md                ← NDMF 工具生态
  optimization-guide.md        ← 最佳化实操指南
  avatar-modding-guide.md      ← Bike Pose 修复等改模知识
  modular-avatar.md            ← Modular Avatar 插件
  accessories.md               ← ⭐Avatar Accessories 配件系统(2026.2.1+) NEW 2026-06-30
  vrcraycast.md                ← ⭐VRCRaycast 骨骼 raycast(SDK 3.10.3+) NEW 2026-06-30
  camera-tools.md              ← ⭐NEW 2026-07-04 玩家本地相机工具 (VirtualLens2 + VRCLens)
  avatar-parameter-staged-sync.md
  blender-shape-keys.md        ← ⭐NEW 2026-07-04 Blender 形态键 → Unity BlendShapes
  shader/
    index.md                   ← Shader 索引（完整对比矩阵 + 选择指南）⭐2026-06-11
    liltoon/                   ← lilToon 详细文档（16个文件）
    scss.md                    ← SCSS 详细分析报告（双阴影/UV切换）⭐2026-06-11
    orl/                       ← ORL Shaders World 通用（4个文档）⭐NEW
    filamented/                ← Filamented PBR 替代（4个文档）⭐NEW
    unlitwf/                   ← UnlitWF Unlit 扩展（1个文件 + 详细分析）⭐NEW
    other-shaders.md           ← 其他 Shader（Graphlit 等）
```

## Hybrid Domain 文件清单

```
concepts/hybrid/
  index.md                ← 域名索引
  osc-protocol.md         ← OSC 协议数据库 ✅
  audio-link.md            ← AudioLink 系统架构 ✅
```

---

## 搜索关键词

| 领域 | Grep pattern | 路径 |
|---|---|---|
| World/网络 | `networking\|sync\|SetOwner\|RequestSerialization` | [[concepts/rules/index]] |
| World/性能 | `EXTERN\|boxing\|Update\|GC\|loop` | [[concepts/rules/index]] |
| World/陷阱 | `FAIL-\|静默\|陷阱` | 源资料归档层（不随发布包分发） |
| World/后处理 | `bloom\|vignette\|grain\|motion blur\|chromatic` | [[concepts/misc/index]] |
| **World/Persistence** | `PlayerData\|PlayerObject\|VRCEnablePersistence\|OnPlayerRestored` | [[entities/world/udon/persistence/index]] |
| API | API 方法名 | [[entities/api/index]] |
| Avatar | `animator\|physbone\|contact\|constraint\|cloth\|benchmark` | [[entities/avatar/index]] |
| **Shader/Avatar专用** | `liltoon\|scss\|cel.*shader\|toon.*shadow` | [[entities/avatar/shader/index]] |
| **Shader/World专用** | `orl.*shader\|filamented\|pbr.*world\|configurable` | [[entities/avatar/shader/index]] |
| **Shader/通用** | `unlitwf\|fur\|water.*shader\|gem.*shader\|grass.*shader` | [[entities/avatar/shader/index]] |
| Hybrid/OSC | `osc\|hybrid\|cross-domain\|AudioLink\|ALPass` | [[concepts/hybrid/index]] |
| **VRCGraphics** | `VRCGraphics\|VRCShader\|SetGlobal\|Blit` | [[entities/world/index]] |
| **VRCCameraSettings** | `VRCCameraSettings\|GetEyePosition\|GetCurrentCamera\|OnVRCCameraSettingsChanged` | [[entities/world/index]] |
| **玩家本地相机工具** | `VirtualLens2\|VRCLens\|vrclens\|virtual-lens\|tonemapper\|DoF.*shader\|camera.*plugin` | `entities/avatar/camera-tools.md` |
| **VRCQualitySettings** | `VRCQualitySettings\|SetShadowDistance\|OnVRCQualitySettingsChanged` | [[entities/world/index]] |
| **Data Containers** | `DataList\|DataDictionary\|DataToken\|VRCJson` | [[entities/world/index]] |
| **Platform** | `android\|quest\|mobile\|cross-platform` | [[entities/platform/index]] |
| **Creator Economy** | `Store\|UdonProduct\|OpenListing` | [[entities/world/index]] |
| **光照系统** | `LightVolume\|PointLight\|SH\|体素` | [[entities/world/index]] |
| **World Shader** | `Graphlit\|node editor\|PBR\|Filament` | [[entities/world/shader/index]] |
| **Udon Example** | `ButtonSync\|SendEventOn\|PlayerTrigger\|UdonSyncPlayer\|VRCWorldSettings` | [[entities/world/examples/udon-example-scene/index]] |

---

## Journal

> ℹ️ V3.0 起,会话记忆(Journal)已弃用,所有整理阶段报告统一写入 `特殊Agent提示词/` 目录。
> 临时记录不再保留,知识库保持长期纯净。