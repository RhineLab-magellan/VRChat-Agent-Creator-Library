---
title: VRChat 全领域核心约束速查
category: misc

knowledge_level: applied
status: active

tags:
  - misc
  - reference
  - core-constraints

aliases:
  - 核心约束
  - "Critical Constraints"
  - 速查表
  - Cheatsheet

related:
  - [[entities/world/bakery/index.md]]
  - [[entities/world/items.md]]
  - [[entities/world/supported-assets.md]]
  - [[entities/world/creator-economy.md]]
  - [[entities/world/shader/index.md]]
  - [[entities/avatar/shader/poiyomi/index.md]]
  - [[entities/avatar/shader/liltoon/index.md]]

source: 本地知识库整理
source_type: community
version: 3.0
last_review: 2026-07-01
confidence: High
type: navigation
created: 2026-07-01
sources: 本地知识库整理
updated: 2026-07-01
---
# VRChat 全领域核心约束速查


---

## 领域识别（回答前必做）

```
Avatar? → Animator/PhysBone/Contact/Constraint/Shader/Optimization
World?  → UdonSharp/Networking/Gameplay/Performance/Security
Hybrid? → OSC/Avatar↔World/External
不确定  → 必须询问用户
```

## 回答前自检

```
□ 领域识别完成? □ 知识检索完成? □ FACT/INFERENCE 区分?
□ 风险标记? □ 未知项标记? □ 知识充足?
→ 不足则输出 Missing Information，不强行回答
```

## 知识优先级

```
L1 官方规范 > L2 官方发布 > L3 社区标准 > L4 优秀案例 > L5 工程推理
L5 必须标注【推断】; 证据不足必须标注【未确认】
```

---

## World Domain — 立即检查（代码生成前 30s）

```
□ 是否使用了 List/Dictionary/LINQ/lambda?
□ 是否使用了 async/try-catch/coroutine?
□ 是否使用了 internal/NoVariableSync/partial?
□ Manual Sync → RequestSerialization()?
□ 同步了 String/DisplayName?
□ Update 中分配内存/调 EXTERN?
□ 热路径 GetComponent/Find?
□ 非 Owner 写入 UdonSynced?
□ Enum 比较 cast 到 int?
□ API 是否在 Udon 中暴露？ → 检查 entities/api/udon-type-exposure.md
```

## World Domain — 网络同步核心

```
1. Owner 写入 → Manual + RequestSerialization()
2. Late joiner → 必须 [UdonSynced]
3. Continuous → 丢包不重传，仅用于连续数据
4. Manual → 多变量时 OnDeserialization 可能分帧
5. [NetworkCallable] → SDK 3.8.1+，8 参数
6. SetOwner → 本地立即生效
7. 非 Owner 写入 → 静默回退
8. _ 前缀 public → 防网络事件调用
```

## World Domain — VM 性能事实

- 9 条指令，纯解释器，比 C# 慢 200x-1000x
- 每变量 = StrongBox<T>（装箱/拆箱）
- EXTERN = 最昂贵指令
- 10 秒硬限制 → Behaviour 永久停止
- Instantiate Udon prefab ≈ 2ms

## World Domain — Persistence 核心约束(NEW 2026-06-21)

```
1. 100KB 配额/玩家/World(压缩后),PlayerData 与 PlayerObject 独立配额
2. 必须在 OnPlayerRestored 后读写(OnPlayerJoined 写会被服务器拉来的数据覆盖)
3. 写本地玩家(PlayerData)或自己的 PlayerObject(PlayerData 写错 player 静默失败)
4. Key 不可删除,只能 Set 覆盖(用空值/0 软删)
5. PlayerData 写 = 发全部 PlayerData,高频写走 PlayerObject
6. PlayerObject 所有权不可转移(防偷),必须加 VRCEnablePersistence 才持久化
7. 命名空间:用 <Prefab名>-<功能> 前缀避免冲突
8. 字符串值 ~50 字符,Key 名 128 字符(建议上限)
→ 详细:entities/world/udon/persistence/index.md
```

## World Domain — 禁止清单

| 类别 | 禁止项 |
|---|---|
| 集合 | List<T>, Dictionary, LINQ |
| 异步 | async/await, try/catch, coroutine |
| 泛型 | lambda, delegate, AddListener |
| 关键字 | internal, partial, NoVariableSync |
| 热路径 | GetComponent, Find, Debug.Log, 字符串拼接, 分配内存, EXTERN |
| 同步 | String, DisplayName, Collision Ownership Transfer |
| 持久化 | OnPlayerJoined 写数据,无前缀 Key,PlayerData 存数组/字典(用 byte[]+VRCJson) |

## World Domain — Enum 5 陷阱

1. ToString() → 输出数字
2. cast 不能与操作符写一起
3. 可选参数不能用 Enum 默认值
4. 非用户 Enum 比较 → cast 到 int
5. `var max = TWO; max >= TWO` → false（先转 int）

---

## Avatar Domain — Animator 核心约束

### Playable Layers 播放顺序

```
Base → Additive → Gesture → Action → FX（最后，最高优先级）
```

### 关键规则

```
1. 前四层（Base/Additive/Gesture/Action）只控制 Transform 和 GameObject 开关
2. Blendshape/材质替换/着色器动画 → 必须在 FX 层
3. 禁止同一控制器中混用 Write Defaults On/Off
4. WD Off 状态必须有填充动画（空属性 + 至少 2 帧）
5. FX 层用 WD Off 控制 Transform → 覆盖 Gesture 层所有 Transform
6. Direct Blend Tree → 必须 Write Defaults On
7. 单 State 的 Direct BT 例外，可单独使用任意 WD
```

### Quest/Android Avatar 性能硬约束（2026-07-04 新增）

> 来源: VRChat 官方 creators.vrchat.com/avatars/avatar-performance-ranking-system/

```
□ Quest 面数 ≤ 20,000 (Poor), 推荐 ≤ 10,000
□ Quest 材质内存 ≤ 40 MB (Poor)
□ Quest 骨骼 ≤ 150 (Medium), 推荐 ≤ 75
□ Quest PhysBone 组件 ≤ 8 (Poor)
□ Quest 粒子系统 ≤ 2 (Poor)
□ 只能使用 VRChat/Mobile/ 目录下的 Shader
□ 只能使用 VRChat Constraints (Unity Constraints 禁用)
□ 🔴 Very Poor 评级未来可能被完全删除
```
→ 详细: `entities/avatar/performance-rank.md` §Quest 平台标准

### Expression Parameter 类型兼容

| Animator \ Expression | Bool | Int | Float |
|---|---|---|---|
| Bool → | 直接传递 | 0/1 | 0.0/1.0 |
| Int → | >0 = True | 直接传递 | 直接转换 |
| Float → | >0 = True | 四舍五入 | 直接传递 |

### Avatar Mask 行为

- 启用 Humanoid Muscle → 可修改该值
- 禁用 Humanoid Muscle → 不能修改
- 启用 Transform → 可修改位置/旋转/缩放 + 替换第一材质球
- 禁用 Transform → 不能修改
- 有 Avatar Mask → 不能控制除第一个外的材质球

---

## Avatar Domain — Shader 关键约束(NEW 2026-07-01 Poiyomi 入库)

### 渲染管线(绝对规则)

```
🔴 VRChat 只支持 BRP(Built-in Rendering Pipeline)
❌ 禁止 URP / HDRP
❌ 禁止切换渲染管线(致命)
→ 所有 Avatar Shader 都受此约束,包括 Poiyomi / lilToon / SCSS / ORL / UnlitWF
```

### Poiyomi 5 变体速查(2026-07-01 入库)

| 变体 | 平台 | 性能 | 适用 |
|------|------|------|------|
| **Nano** | PC+Quest | ⭐⭐⭐⭐⭐ | 极简 Avatar |
| **Micro** | Quest 首选 | ⭐⭐⭐⭐ | Quest Avatar |
| **Mega** | PC 通用 | ⭐⭐⭐ | 标准 PC Avatar |
| **Giga** | PC 高端 | ⭐⭐ | 复杂 PC Avatar |
| **Tera** | 顶级 PC | ⭐ | 顶级视觉效果 |

### Poiyomi Pro vs Toon 关键差异

| 功能 | Toon | Pro |
|------|------|-----|
| 5 变体 + 基础效果 | ✅ | ✅ |
| Modular Shader System | ❌ | ✅ |
| Poiyomi Fur(1-31 层) | ❌ | ✅ |
| ShatterWave / Geometric Dissolve | ❌ | ✅ |
| Constellation(星空)/ Voronoi 3D | ❌ | ✅ |
| Global Themes(4 主题) | ❌ | ✅ |
| Pro 价格 | 免费 | Patreon $10+/月 |

→ 详细:[[entities/avatar/shader/poiyomi/index]](8 主题文档)⭐NEW

### Quest Avatar 硬性规则

```
✅ 必须用 Micro 或 Nano 变体
❌ 禁用 Grab Pass / ShatterWave / Geometric Dissolve
❌ 禁用 Voronoi 3D / Constellation / Internal Parallax
⚠️ AudioLink 限 4 频段(Micro)
⚠️ Material 数量 ≤ 5
→ 详见 entities/avatar/shader/poiyomi/quest-optimization.md
```

### Shader 选择决策(2 步)

```
1. 风格化 + VFX(星空/几何) → Poiyomi Pro
2. 通用 + 大量预设 + 开源 → lilToon
3. 极致 Toon + UV 服装切换 → SCSS
4. 国内访问稳定 → lilToon / SCSS(Poiyomi mirror 失败)
```

---

## Hybrid Domain — ✅ 已建设

```
OSC 协议数据库 ✅ / AudioLink 系统架构 ✅ / Avatar↔World 主骨架已建立（首轮入库完成，仍待补证）
Tracking 已并入 Avatar↔World 观察层 / External Integration 仍待建设
```

---

## 文件速查

| 需要 | 去 |
|---|---|
| World 代码/网络/性能 | [[concepts/rules/index]] |
| World API | [[entities/api/index]] (grep) |
| **Udon 官方文档本地化** | [[entities/world/udon/index]] ⭐ 11+ 单页(2026-06-21 +VRCTween 子目录) |
| **Udon 事件执行顺序** | `entities/world/udon/event-execution-order.md` ⭐_onEnable→_start 无间隔 |
| **Udon Persistence 实战** | [[entities/world/udon/persistence/index]] ⭐ 8 文档(PlayerData/PlayerObject/100KB/3 模式)NEW 2026-06-21 |
| **Udon VM 字节码** | `entities/world/udon/vm-and-assembly.md` 9 Opcodes 官方规范 |
| **VRCTween(官方补间)** | `entities/world/udon/vrctween/index.md` ⭐ DOTween 封装,7 大类补间 + 虚拟补间 + 复用模式 ⭐NEW 2026-06-21 (2026-06-22 已校验与官方 5-28 一致) |
| **VRCTween 原始英文 / DOTween 底层** | VRCTween 官方原始快照（源资料层，不随发布包分发）(2026-06-22 抓取) |
| World 设计模式 | [[concepts/patterns/index]] ⭐ 31 个(7 原有 + 6 LuraSwitch2 + 10 UVU + 5 ULocalization 沙箱适配 + 3 Sardinal 独有) |
| **Master-Follower Syncer** | `concepts/patterns/master-follower-syncer.md` ⭐NEW |
| **Exclusive Control Selector** | `concepts/patterns/exclusive-control-selector.md` ⭐NEW |
| **Soft Detent Interpolation** | `concepts/patterns/soft-detent-interpolation.md` ⭐NEW |
| **Avatar Accessories (2026.2.1+)** | `entities/avatar/accessories.md` ⭐NEW 2026-06-30 |
| **VRCRaycast (Avatar, SDK 3.10.3+)** | `entities/avatar/vrcraycast.md` ⭐NEW 2026-06-30 |
| **Companions vs Props (2026.2.3)** | `entities/world/companions.md` ⭐NEW 2026-06-30 |
| **Steam Audio (替换 ONSP, 2025.4.2+)** | `entities/world/audio-steam.md` ⭐NEW 2026-06-30 |
| **Fade-Then-Snap** | `concepts/patterns/fade-then-snap.md` ⭐NEW |
| **Editor Preview Component** | `concepts/patterns/editor-preview-component.md` ⭐NEW |
| **MaterialPropertyBlock Safe Update** | `concepts/patterns/material-propertyblock-safe-update.md` ⭐NEW |
| **Hash-Based Method Dispatch** | `concepts/patterns/hash-based-dispatch.md` ⭐NEW 2026-06-20 |
| **IID Object Identity** | `concepts/patterns/iid-object-identity.md` ⭐NEW 2026-06-20 |
| **Slot-Based Parameter Passing** | `concepts/patterns/slot-parameter-passing.md` ⭐NEW 2026-06-20 |
| **Code Generation with Type Erasure** | `concepts/patterns/code-generation-type-erasure.md` ⭐NEW 2026-06-20 |
| **Build-Time vs Runtime Separation** | `concepts/patterns/build-time-vs-runtime-separation.md` ⭐NEW 2026-06-20 |
| **QuickBrown LuraSwitch2 源（案例研究 A6,v3.00 历史）** | [LuraSwitch2（Lura Booth）](https://lura.booth.pm/) ⭐NEW |
| **LuraSwitch2 工具使用指南（C15,v1.06 重写版）** | `entities/world/luraswitch2.md` ⭐REWRITTEN 2026-06-21 (2026-03-06 最新版) |
| **VizVid (VVMW) 源（案例研究 A2）** | [VVMW（GitHub）](https://github.com/JLChnToZ/VVMW) ⭐NEW 2026-06-20（视频播放器） |
| **VizVid (VVMW) 工具使用指南（C16）** | `entities/world/vvmw.md` ⭐NEW 2026-06-20（推荐创作者使用） |
| **HoshinoLabs ULocalization 源** | [ULocalization（GitHub）](https://github.com/ikuko/ULocalization) ⭐NEW 2026-06-20 |
| **HoshinoLabs Sardinal 源** | [Sardinal（GitHub）](https://github.com/ikuko/Sardinal) ⭐NEW 2026-06-20 (通用消息总线) |
| **lilxyzw 工具集源** | [lilToon 官方](https://lilxyzw.github.io/lilToon/) ⭐UPDATED 2026-07-04(lilAvatarUtils v1.0.0 → v2.1.2 + 9 Tab 完整列表 + UI 操作规则) |
| **Channel-Based Pub/Sub Routing** | `concepts/patterns/channel-routing.md` ⭐NEW 2026-06-20 (Sardinal 独有) |
| **Inherited Subscriber** | `concepts/patterns/inherited-subscriber.md` ⭐NEW 2026-06-20 (Sardinal 独有) |
| **Hybrid Static+Dynamic Subscription** | `concepts/patterns/hybrid-subscription-modes.md` ⭐NEW 2026-06-20 (Sardinal 独有) |
| World 审查 | 源资料归档层（不随发布包分发） |
| **光照系统** | `entities/world/vrc-light-volumes.md` |
| **光照烘焙** | `entities/world/bakery/light-baking-guide.md` |
| **遮挡剔除** | `entities/world/occlusion-culling-guide.md` |
| **World Shader** | [[entities/world/shader/index]] ← Graphlit 节点编辑器 |
| **VRCGraphics / Shader Globals** | `entities/world/udon/vrc-graphics/index.md` |
| **Udon Node Graph(可视化)** | [[entities/world/udon/graph/index]] ← Interface/Flow/Event/搜索/类型 ⭐NEW 2026-06-15 |
| **VRCCameraSettings** | `entities/world/vrc-camera-settings.md` ⭐NEW |
| **VRCQualitySettings** | `entities/world/vrc-quality-settings.md` ⭐NEW |
| **ClientSim(World 测试)** | `entities/world/clientsim/index.md` ⭐NEW 2026-06-15 |
| **Obstacle Course(World Jam 2)** | [[entities/world/examples/obstacle-course/index]] ⭐NEW 2026-06-15 |
| **Items in Udon Worlds** | `entities/world/items.md` ← ⭐ Layer 3 + Udon 不可引用 Items ⭐NEW 2026-06-15 |
| **VRChat Community Labs** | `entities/world/community-labs.md` ⭐NEW 2026-06-30(World 发布系统 + Trust Rank User + 每周 1 个) |
| **Unity Layers in VRChat** | `entities/world/layers.md` ← ⭐ 32 Layer 分配 + Physics 安全模式 ⭐NEW 2026-06-15 |
| **SDK Prefabs** | `entities/world/sdk-prefabs.md` ← ⭐ 8 个 Udon 示例 Prefab ⭐NEW 2026-06-15 |
| **Supported Scripted Assets** | `entities/world/supported-assets.md` ← ⭐ TMP/PPSv2/FinalIK/DynamicBone 白名单 ⭐NEW 2026-06-15 |
| **Allowlisted World Components** | `entities/world/whitelisted-world-components.md` ← ⭐ 完整 Script 白名单 (10 集合) ⭐NEW 2026-06-15 |
| **Video Players** | [[entities/world/udon/video-players/index]] ← ⭐ VRCUnityVideoPlayer / AVPro + URL 白名单 ⭐NEW 2026-06-15 |
| Avatar Animator | `entities/avatar/animator-system.md` ⭐2026-07-04 补充 VRC School（§0/§1.1/§4.1/§5.1/§7/§8/§9）|
| **Avatar 自动纹理压缩 (LAC)** | `entities/avatar/lac-avatar-compressor.md` ⭐NEW 2026-06-17 |
| **MA 响应式层合并 (MA2BT)** | `entities/avatar/ma2bt.md` ⭐NEW 2026-06-17 |
| **Avatar 优化器 (AAO)** | `entities/avatar/avatar-optimizer.md` ⭐NEW 2026-06-17 |
| **TexTransTool (TTT)** | `entities/avatar/tex-trans-tool.md` ⭐NEW 2026-06-17 |
| **VRCFury 客观参考** | `entities/avatar/vrcfury-reference.md` ⭐NEW 2026-06-17（2026-07-04 补充 VRC School 工具链对比）|
| **VCC 创作者助手（安装 + 项目创建）** | [VCC 文档（官方）](https://vcc.docs.vrchat.com/) ⭐NEW 2026-07-04（VRC School 整理）|
| **Meshia 减面** | `entities/avatar/meshia-mesh-simplification.md` ⭐NEW 2026-06-17 |
| **Thry 性能检测指标** | `entities/avatar/thry-avatar-evaluator-metrics.md` ⭐NEW 2026-06-17 |
| **NDMF 工具生态** | `entities/avatar/ndmf-tools.md` ⭐执行顺序 ⭐2026-07-04 补充 lilAvatarUtils(§UI 操作 + §性能指标体系 + PhysBones 5 指标 + Lighting 6 模式 + Safety 拼写纠正) ⭐2026-07-04 补充 Avatar Clothing Tool |
| **Quest/Android Avatar 性能标准** | `entities/avatar/performance-rank.md` §Quest (Mobile) 平台标准 ⭐2026-07-04 修正（修正 9+ 项关键数据 + 新增 Constraint/Raycast 指标）|
| **Android Shader 完整列表** | `entities/platform/android-development.md` §Avatar 着色器限制 ⭐2026-07-04 完善（含 Toon Standard）|
| **Android 插件兼容性原则** | `entities/platform/android-development.md` §Android 插件兼容性原则 ⭐2026-07-04 新增 |
| **PC→Android 工程工作流** | `entities/platform/cross-platform-content.md` §PC→Android 工程组织建议 ⭐2026-07-04 新增 |
| **Very Poor 未来风险** | `entities/platform/android-development.md` §Very Poor Avatar 未来风险警告 ⭐2026-07-04 新增（官方 DANGER 警告）|
| **VRChat Trust Rank** | `entities/avatar/trust-rank.md` ⭐(2026-06-30) 7 等级 + 上传权限 + Nuisance |
| **VRChat Safety System** | `entities/avatar/safety-system.md` ⭐(2026-06-30) Shield Level + 8 特性 + Safe Mode |
| **Avatar Fallback System** | `entities/avatar/avatar-fallback-system.md` ⭐NEW 2026-06-30(5 原因 + 上传流程 + Grandfathered) |
| **用户端 Dynamic Bone Limits** | `entities/avatar/avatar-dynamic-bone-limits.md` ⭐NEW 2026-06-30(默认 32/8 + config.json) |
| **用户端 Particle System Limits** | `entities/avatar/avatar-particle-system-limits.md` ⭐NEW 2026-06-30(11 变量 + Penalty + Quest 不可禁用) |
| **Full-Body Tracking** | `entities/avatar/full-body-tracking.md` ⭐NEW 2026-06-30(8 tracker + Rigging + 实验性) |
| **IK 2.0** | [[entities/avatar/ik-2.0]] ⭐NEW 2026-06-30(Lock Types + 5 启动参数) |
| **Avatar 交互权限** | `entities/avatar/avatar-interaction-permissions.md` ⭐NEW 2026-06-30(Mode + PANIC) |
| **Public Avatar Cloning** | `entities/avatar/public-avatar-cloning.md` ⭐NEW 2026-06-30 |
| **Playable Layers（5 层详解 + Mask 规则）** | `entities/avatar/playable-layers.md` ⭐2026-07-04 补充 VRC School（§1.1/§8/§9）|
| Avatar 其他 | [[entities/avatar/index]] |
| Hybrid/OSC | [[concepts/hybrid/index]] |
| **VRChatSDK API** | `entities/vrchatsdk/index.md` ⭐NEW 2026-06-15 域索引 |
| 跨平台开发 | `entities/platform/index.md` ⭐NEW 2026-06-15 域索引 |
| **VCC 创作者助手** | [VCC 文档（官方）](https://vcc.docs.vrchat.com/) ⭐NEW 2026-07-04（安装 + 项目创建 + Avatar 上传）|
| 杂项(后处理/无障碍) | `concepts/misc/index.md` ⭐NEW 2026-06-15 域索引 |
| 知识版本 | [[FACT]] |
| **玩家本地相机工具** | `entities/avatar/camera-tools.md` ⭐NEW 2026-07-04（VirtualLens2 + VRCLens）|

---

## 2026-07-04 更新追加

### Avatar 域核心约束（追加）

- 🔴 **Expression Parameters 配额**: synced 总计 **256 bits**，含 unsynced 总计 8192 个（已知 bug：> 256 时加载失败）。任何添加新参数的组件都需要检查余量。第三方工具通常需要 8-16 bit 缓冲。
- 🔴 **第三方工具链兼容性**: VirtualLens2 需要 VRCSDK 3.7.0+ + MA 1.8.0+ + Unity 2022 LTS + 8 bit 表情参数缓冲
- 🔴 **工具设计哲学对比**: Non-Destructive (VRCFury/MA/VirtualLens2) vs Invasive (VRCLens)，选择需考虑 Avatar 复用场景

### World 域工具补充

- VRCLens 修复 Avatar 相机 DepthTextureMode 失效问题（Canny tracked）
- VirtualLens2 提供 Remote Only Mode 优化 Android 移动平台部署
- VRChat 原相机景深错误是官方已知问题（详见 `entities/world/vrc-camera-settings.md` §Avatar 相机限制）
| 文档对比记录 | 源资料归档层（不随发布包分发） ⭐NEW 2026-06-15 域索引 |
