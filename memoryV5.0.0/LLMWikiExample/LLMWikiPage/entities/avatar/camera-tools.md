---
title: "玩家本地相机工具 - VirtualLens2 + VRCLens"
category: avatar
knowledge_level: applied
status: active
tags:
  - avatar
  - virtual-lens
  - vrclens
  - virtual-photography
  - hybrid-tool
  - non-destructive
aliases: [玩家本地相机工具, camera-tools, VirtualLens2, VRCLens]
related:
  - vrcfury-reference.md
  - modular-avatar.md
  - performance-rank.md
  - playable-layers.md
  - camera-tools-virtuallens2.md
  - camera-tools-vrclens.md
  - ../world/vrc-camera-settings.md
  - ../../concepts/hybrid/vcc.md
  - ../platform/android-development.md
  - ../../concepts/patterns/index.md
source: "VirtualLens2 本地知识整理（MEMORY_INCLUSION_GUIDE）"
source_type: community
version: 1.0
date_added: 2026-07-04
last_review: 2026-07-04
confidence: High
type: entity
created: 2026-07-04
sources: "VirtualLens2 本地知识整理（MEMORY_INCLUSION_GUIDE）"
updated: 2026-07-04
---

# 玩家本地相机工具 - VirtualLens2 + VRCLens

> 🔴 **本文档整合来源**: VirtualLens2 + VRCLens 跨工具评估（2026-07-04）
> 🔴 **核心价值**: 知识库首次形成"VRChat 玩家本地相机工具"完整图谱

---

## 1. 概述与定位

### 1.1 为什么需要第三方相机工具？

VRChat 原生相机在 Avatar 摄影中存在多个**官方已知问题**（详见 `world/vrc-camera-settings.md` §Avatar 相机限制）：

- Avatar Camera 无 DepthTextureMode 支持（Canny tracked）
- DoF 噪声无法关闭
- Avatar 相机使用 stale depth texture
- 缺乏高级摄影功能（HDR、White Balance、Stereographic 3D 等）

第三方相机工具通过覆盖 PostProcess、自定义 Shader、提供额外功能，填补了这一空白。

### 1.2 工具谱系

> 📌 **结构说明**: 本文件保留“总览 / 对比 / 选型”层信息；更细的安装、操作、版本迁移与已知兼容性细节，优先拆入：
> - `avatar/camera-tools-virtuallens2.md`
> - `avatar/camera-tools-vrclens.md`

| 维度 | VirtualLens2 | VRCLens |
|------|--------------|---------|
| **作者** | ろじらぼ (logi_9) | hirabiki |
| **分发** | BOOTH (PCVR) | Gumroad 独家 |
| **价格** | 付费 | Pay What You Want ($11+) |
| **Unity** | 2022 LTS | 2022 LTS (≥1.7.9 需 VRCSDK3 3.0.8+) |
| **VRCSDK** | 3.7.0+ | 3.0.8+ |
| **表情参数需求** | 8 bit | 8 bit |
| **平台** | PCVR + Remote Mode (Android) | Desktop + VR |
| **设计哲学** | **Non-Destructive** (MA + NDMF) | **Invasive** (直接改 Avatar 资源) |
| **关键依赖** | Modular Avatar 1.8.0+ | VRLabs/AV3Manager + Filament PBR + DoF Shader |

### 1.3 性能排名与本地对象（关键问题）

> 🔴 **核心问题**: VRChat Avatar Performance Rank **不区分本地对象和远程对象**

- 像 VirtualLens2 / VRCLens 这样复杂的本地小把戏会**显著拉低**性能排名
- 但这些本地对象**对其他玩家无任何影响**
- 这是 **200+ voters** 的 Canny 反馈但**未解决**的议题
- 来源: <https://feedback.vrchat.com/avatar-30/p/feedback-mark-local-objects-explicitly-and-exclude-them-from-performance-rank-ca>
- 详细见 `avatar/performance-rank.md` §本地对象污染性能排名

---

## 2. 工具案例 A: VirtualLens2

### 2.1 作者元信息

**作者**: ろじらぼ (logi_9 / @logi9_)
**Twitter**: [@logi9_](https://x.com/logi9_)
**BOOTH 主页**: [ろじらぼ BOOTH 店铺](https://logilabo.booth.pm/)

🔴 **元知识关联**:
- 该作者同时是 VRChat 反馈平台 (Canny) 上"**性能排名不区分本地对象**"议题的发起人
- 议题链接: <https://feedback.vrchat.com/avatar-30/p/feedback-mark-local-objects-explicitly-and-exclude-them-from-performance-rank-ca>
- 创建时间: 2022-04-26
- 状态: 200+ voters，**未解决**
- **意义**: 表明 VirtualLens2 的设计者深度参与 VRChat 创作者生态

### 2.2 VirtualLens2 完整功能矩阵

> 来源: VirtualLens2 官方英文文档 vlens2.logilabo.dev

| 功能类别 | 功能 | 用途 | 平台 |
|---------|------|------|------|
| **基础** | 变焦 (Zoom) | 调整焦距 (35mm 等效) | PCVR |
| **基础** | 光圈 (Aperture) | F 值 + 景深模拟 | PCVR |
| **基础** | 曝光 (Exposure) | EV 调整 | PCVR |
| **对焦** | Point AF | 单点自动对焦（食指触摸预览）| PCVR |
| **对焦** | Face AF | 人脸识别自动对焦 | PCVR |
| **对焦** | Selfie AF | 自拍自动对焦（聚焦拍照者）| PCVR |
| **对焦** | Manual Focus | 手动对焦 | PCVR |
| **位置** | Auto Leveling | 自动地平线水平校正 | PCVR |
| **位置** | Stabilizer | 稳定功能（防抖）| PCVR |
| **位置** | Pin to World | 固定到世界空间 | PCVR |
| **位置** | Reposition | 重新定位模式 | PCVR |
| **位置** | Pin[1-4] | 4 个保存的相机位置 | PCVR |
| **位置** | Drone Mode | 无人机模式（2D Puppet）| PCVR |
| **位置** | Quick Selfie | 快速自拍（v2.10.4+）| PCVR |
| **高级** | Quick Call | 预设调用（最多 8 个）| PCVR |
| **高级** | Mesh Hiding | 隐藏指定网格 | PCVR |
| **高级** | Mask System | 遮罩（本地/远程/UI）| PCVR |
| **高级** | Grid (3x3/6x4/Custom) | 网格显示 | PCVR |
| **高级** | Focus Peaking | 对焦峰值 | PCVR |
| **高级** | Preview HUD | 视野中心预览 | PCVR |
| **高级** | Depth Texture Enabler | 深度纹理启用 | PCVR |
| **远程** | Remote Only Mode | 仅显示其他玩家头像 | Android (移动平台优化) |
| **4K** | 4K Capture | 4K 模式 | PCVR (Stream Camera 配合) |
| **API** | TouchOverride* | 模拟屏幕触摸 API | 开发者用 |

**远程同步参数** (VirtualLens2 独有特性):
- 焦距、光圈、曝光、手动对焦距离都支持**与远端玩家同步**
- 这是 Hybrid 域（Avatar ↔ World 跨域）参考价值的功能

### 2.3 相机模型参考对象约定

| 参考对象 | 用途 | 坐标约定 |
|----------|------|----------|
| `VirtualLensOrigin` | 相机组件放置参考 | +Z 前方, +Y 上方 |
| `VirtualLensPreview` | 预览屏幕参考 | 触摸检测基于此 |
| `NonPreviewRoot` | 非预览根（自定义网格子级）| 替换相机模型 |

> **0.05m 规则**: 没有可见的多边形距离 `VirtualLensOrigin` 的视线（+Z 方向）超过 0.05 米，否则可能导致相机网格出现反射。

> **兼容检测**: 第三方预制件中若包含 `VirtualLensOrigin` 和 `VirtualLensPreview` 对象，则该预制件与 VirtualLens2 兼容。

### 2.4 关键技术约束

- **VRCSDK 最低**: 3.7.0+
- **Modular Avatar 最低**: 1.8.0+
- **表情参数需求**: 8 bit 缓冲
- **Unity 版本**: 2022 LTS
- **设计哲学**: Non-Destructive（通过 MA + NDMF）

### 2.5 4K 模式

详见 `world/vrc-camera-settings.md` §4K 模式与分辨率匹配

---

## 3. 工具案例 B: VRCLens

### 3.1 关键技术依赖

| 依赖组件 | 来源 | 许可证 | VRCLens 中的作用 |
|---------|------|--------|------------------|
| **Avatars 3.0 Manager (AV3Manager)** | [VRLabs/Avatars-3.0-Manager](https://github.com/VRLabs/Avatars-3.0-Manager) | MIT | Avatar FX Controller / Menu / Parameters 自动化修改 |
| **EVILS Tonemapper** | Android Open Source Project (Filament PBR) | Apache 2.0 | **HDR 调整 + White Balance** |
| **Single-Pass DoF Shader** | Dennis Gustafsson | (未明确) | **景深效果实现**（与 VRChat 原相机景深区分）|

🔴 **关键发现**:
- VRCLens 集成了三个不同来源的关键技术
- 与 **VRChat 只支持 BRP + Forward Rendering** 的限制直接相关 → VRCLens 不得不使用第三方 Tonemapper 实现 HDR 效果
- 解释了为什么 VRCLens 能提供原相机没有的功能（HDR / White Balance / 高级景深）

**【外部验证来源】**:
- [Hirabiki 官方 Gumroad Third-party Credits](https://hirabiki.gumroad.com/l/rpnel)
- [VRLabs/Avatars-3.0-Manager 独立仓库](https://github.com/VRLabs/Avatars-3.0-Manager)

### 3.2 VRCLens 完整功能矩阵

| 功能类别 | 功能 | 用途 | 官方 Camera OSC 等价 |
|---------|------|------|---------------------|
| **基础** | Desktop/2D 模式 | 无需 VR 头显 | ❌ |
| **基础** | 曝光/缩放/光圈/传感器尺寸 | 模拟真实相机 | `/usercamera/Zoom`、`/usercamera/Exposure`、`/usercamera/Aperture` |
| **图像** | Picture Style 预设 | 多种风格 | ❌ |
| **图像** | **HDR + White Balance** | 色调控制 | ❌（VRCLens 独有）|
| **图像** | Image Stabilization + auto-straighten | 视频防抖 | ❌ |
| **飞行** | **Built-in drone movement** + tracking | 内置无人机 | ⚠️ 官方 VRCDroneApi 有限 |
| **对焦** | Selfie Autofocus | 自拍自动对焦人脸 | ❌ |
| **对焦** | Avatar-detect autofocus | 仅对焦 Avatar 忽略背景 | ❌ |
| **模式** | **Stereographic 3D camera mode** | 立体 3D 相机 | ❌（VRCLens 独有）|
| **模式** | Quick Selfie front camera mode | 快速自拍前置模式 | ❌ |
| **模式** | World-fixed / Avatar-fixed camera mode | 相机参考系选择 | ❌ |
| **控制** | Easy left-handed camera controls | 左手控制适配 | ❌ |
| **UI** | Viewfinder overlay | 取景器叠加层 | ❌ |
| **集成** | Touch pen to move focus point | 触摸笔控制焦点 | ❌ |
| **集成** | Easy automatic setup into avatar's existing animation | 自动接入 Avatar 现有动画 | ❌ |
| **性能** | Minimal performance impact on other players | 对其他玩家性能影响最小 | N/A |

### 3.3 VRCLens 独特能力

| 能力 | 实现方式 | 与官方对比 |
|------|----------|------------|
| **HDR 调整** | EVILS Tonemapper (Filament PBR) | 官方无等价 |
| **White Balance** | 同一 Tonemapper 集成 | 官方无等价 |
| **修复世界光照错误** | 覆盖 PostProcess + 蓝屏 viewfinder 修复 | 官方未提供 |
| **Stereographic 3D** | 自有实现 | 官方无等价 |

### 3.4 相机模型参考对象

VRCLens 安装路径: `Assets/Hirabiki/VRCLens/Prefabs/VRCLens` (Prefab)
**关键约束**: 拖入场景时**不能**作为其他模型的子物体，必须作为独立 GameObject

### 3.5 VRCLens 第三方生态

| 工具 | 作者 | 用途 |
|------|------|------|
| **VRCLens-Addons** | gummidot | 16+ VRCFury 增强 Prefab (Fisheye, Ghost Lens, Chromatic Aberration 等) |
| **VRCLensOSC** | gummidot | OSC 控制集成 + FX Layer 修改 |
| **VRCCameraShyShader** | PeachBight | VRCLens 专用相机隐藏 Shader |
| **Drone-OSC-Controller** | qbitzvr | 无人机 OSC 增强 |
| **VRCPrismStudio** | VRCPrism | 基于 VRCLens 的高级视频工具（独立付费）|
| **OminiControllerX** | elusyia | 手柄控制 VRCLens |

> 生态价值: VRCLens > VirtualLens2（差距明显）。周边工具丰富，特别是 VRCFury 集成让 VRCLens 可以"非破坏式"使用。

### 3.6 VRCLens 销售许可

| 用途 | 是否允许 |
|------|----------|
| 商业使用（含付费视频/直播）| ✅ 允许 |
| 修改 VRCLens 包 | ✅ 允许 |
| 重新分发（修改或未修改）| ❌ 禁止 |
| 嵌入到公共 Avatar | ❌ 禁止 |

### 3.7 已知兼容性问题

#### APS (Avatar Pose System) 兼容性问题

> 来源: docs.vrcd.org.cn VRCLens 文档评论区

- 当 Avatar 同时使用 VRCLens 和 **Avatar Pose System (APS)** 时
- "灵魂出窍"（teleport）后 VRCLens 相机仍固定在原位置
- 表现：相机与 Avatar 分离
- **解决方案**: 在 APS 设置中，将 VRCLens 相关组件添加到 **unfixed objects** 列表

#### 其他已知问题

- 后处理：景深冲突（DoF 双重叠加）
- 后处理：环境光遮蔽（AO）无效
- 后处理：运动模糊无效
- 这些与 VirtualLens2 的已知问题重叠

---

## 4. 工程哲学对比

🔴 **核心对比**: Non-Destructive vs Invasive 工具设计哲学

### 4.1 对比表

| 维度 | VRCFury (Non-Destructive) | Modular Avatar (Non-Destructive) | VirtualLens2 (Non-Destructive) | VRCLens (Invasive) |
|------|---------------------------|----------------------------------|--------------------------------|--------------------|
| **Avatar 资源修改** | 编译时生成（不动源）| 通过合并 Animator 注入 | 通过 MA + NDMF | 直接修改 FX/Menu/Params |
| **共享 Avatar 场景** | ✅ 友好 | ✅ 友好 | ✅ 友好 | ❌ 不友好（绑定单一版本）|
| **重新 Apply 后果** | 无影响 | 无影响 | 无影响 | ⚠️ Prefab 消失，修改丢失 |
| **推荐组合** | N/A | N/A | N/A | 通过 VRCFury Prefab 包装 |

### 4.2 选择决策树

```
我需要给 Avatar 加一个摄影工具:
├── 我有多个 Avatar 共享一套配置？ → VRCFury / MA / VirtualLens2
├── 我的 Avatar 需要变体管理？ → VRCFury / MA / VirtualLens2
├── 我想最低学习成本？ → VRCLens（注意侵入式代价）
├── 我需要 HDR / White Balance？ → VRCLens（VirtualLens2 不支持）
├── 我需要高级景深算法？ → VRCLens（DoF Shader）
└── 我用 Android/Quest？ → VirtualLens2（Remote Mode）
```

### 4.3 工程教训

> **【元知识】**: VRChat 第三方工具的"侵入 vs 非破坏"分歧反映了**"应用层集成 vs 工具链复用"**的两种工程思路。Non-Destructive 工具需要更复杂的依赖链（MA + NDMF），但换来了多场景复用能力。Invasive 工具部署简单，但代价是 Avatar 资源被"绑定"到该工具。

---

## 5. VRChat 原相机已知问题

> 🔴 **重要**: 本节是"为什么需要第三方相机工具"的核心证据

### 5.1 官方承认的 Avatar 相机限制

详见 `world/vrc-camera-settings.md` §Avatar 相机限制

**核心要点**:
- VRChat 官方 Canny 平台（feedback.vrchat.com）确认以下问题为 tracked 状态
- Avatar Camera 不响应 DepthTextureMode 设置
- DoF 噪声无法关闭
- Avatar 相机使用 stale depth texture

### 5.2 第三方工具存在合理性

| 痛点 | VirtualLens2 应对 | VRCLens 应对 |
|------|------------------|--------------|
| 景深错误 | 自有景深模拟 | 修复 VRChat 原相机景深问题 |
| 深度纹理限制 | 启用深度纹理（性能换质量）| 通过 Single-Pass DoF Shader 绕过 |
| DoF 噪声 | 关闭原生 DoF | （文档未明确）|

---

## 6. 关键技术依赖参考

| 技术 | 类别 | 用途 | 知识库位置 |
|------|------|------|------------|
| **Modular Avatar (MA)** | Avatar 工具 | 1.8.0+ 工具链核心 | `avatar/modular-avatar.md` |
| **NDMF** | 框架 | Non-Destructive 工具框架 | `avatar/ndmf-tools.md` |
| **VRLabs/AV3Manager** | Avatar 工具 | FX/Menu/Params 自动化 | （待建独立文档）|
| **Filament PBR EVILS Tonemapper** | Shader | HDR 调整 + White Balance | （关联 BRP 约束）|
| **Single-Pass DoF Shader** | Shader | 高级景深 | （第三方 Shader 索引）|

---

## 7. 选择建议

### 7.1 我应该选哪个？

| 需求 | 推荐工具 | 理由 |
|------|----------|------|
| 我需要 HDR 摄影 | **VRCLens** | EVILS Tonemapper 独有 |
| 我在 Android/Quest 部署 | **VirtualLens2** | Remote Only Mode |
| 我有多个 Avatar 变体 | **VirtualLens2** | Non-Destructive |
| 我想最低学习成本 | **VRCLens** | 一键 Apply |
| 我需要稳定 Drone 模式 | **VirtualLens2** | 2D Puppet 完整控制 |
| 我重视商业合规 | **VirtualLens2** | 嵌入 Avatar 限制更少 |

### 7.2 兼容性列表

| 与其他工具/系统 | VirtualLens2 | VRCLens |
|-----------------|--------------|---------|
| **Modular Avatar** | ✅ 1.8.0+ 必需 | ⚠️ 不需要（直接 AV3Manager）|
| **VRCFury** | ✅ 兼容 | ✅ 兼容（推荐通过 Addons 包装）|
| **AvatarOptimizer** | ✅ 兼容 | ✅ 兼容 |
| **APS (Pose System)** | ⚠️ 需检查 | ⚠️ 需添加到 unfixed objects |
| **后处理: DoF** | ⚠️ 冲突 | ⚠️ 冲突 |
| **后处理: AO** | ⚠️ 无效 | ⚠️ 无效 |
| **后处理: Motion Blur** | ⚠️ 无效 | ⚠️ 无效 |

---

## 8. 引用与外部链接

### 8.1 官方与原始源

- [Hirabiki VRCLens 官方销售](https://hirabiki.gumroad.com/l/rpnel)
- [VirtualLens2 官方英文文档](https://vlens2.logilabo.dev/en/docs/)
- [VRChat Creator Docs - Animator Parameters](https://creators.vrchat.com/avatars/animator-parameters/)
- [VRChat Creator Docs - Write Defaults](https://creators.vrchat.com/avatars/#write-defaults-on-states)
- [VRChat Launch Options](https://docs.vrchat.com/docs/launch-options#resolution-and-screen-choice)

### 8.2 反馈平台（Canny）

- [Canny: 本地对象标记](https://feedback.vrchat.com/avatar-30/p/feedback-mark-local-objects-explicitly-and-exclude-them-from-performance-rank-ca)
- [Canny: DepthTextureMode 不影响 Avatar 相机](https://feedback.vrchat.com/feature-requests/p/vrccamerasettings-depthtexturemode-doesnt-affect-avatar-cameras)
- [Canny: DoF Noise Toggle](https://feedback.vrchat.com/feature-requests/p/toggle-the-noise-from-the-camera)
- [Canny: 256 参数加载 Bug](https://feedback.vrchat.com/avatar-30/p/1332-bug-vrcexpressionparameters-fail-to-load-correctly-with-more-than-256-param)

### 8.3 中文社区源

- [VRCD.org.cn VirtualLens2 汉化](https://docs.vrcd.org.cn/books/virtual-lens2)
- [VRCD.org.cn VRCLens 汉化](https://docs.vrcd.org.cn/books/vrclens)

### 8.4 第三方生态

- [gummidot/VRCLens-Addons](https://github.com/gummidot/VRCLens-Addons)
- [gummidot/VRCLensOSC](https://github.com/gummidot/VRCLensOSC)
- [VRLabs/Avatars-3.0-Manager](https://github.com/VRLabs/Avatars-3.0-Manager)

---

## 9. 知识截止与监控

> ⚠️ **知识截止日期**: 2026-07-04
> 建议监控:
> - VirtualLens2 1.10+ / 2.0+ 版本
> - VRCLens 2.0+ 版本
> - VRChat Canny 景深问题修复进展
> - Modular Avatar 大版本
> - VRLabs/AV3Manager 演进

---

## 相关页面

[[vrcfury-reference.md]] · [[modular-avatar.md]] · [[performance-rank.md]] · [[playable-layers.md]] · [[camera-tools-virtuallens2.md]] · [[camera-tools-vrclens.md]] · [[../world/vrc-camera-settings.md]] · [[../../concepts/hybrid/vcc.md]] · [[../platform/android-development.md]] · [[../../concepts/patterns/index.md]]
