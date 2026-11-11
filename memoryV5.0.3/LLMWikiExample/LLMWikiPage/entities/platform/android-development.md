---
title: "Android/Quest Development"
category: platform
knowledge_level: applied
status: active
source: 本地知识库整理
source_type: community
version: 1.1
last_review: 2026-07-04
changelog: "2026-07-04 新增 §SDK 面板 Switch Build Target + §Quest 透明度禁用"
confidence: Medium
tags:
  - platform
  - shader
  - light
  - avatar
aliases:
  - ../../Android/Quest Development
  - android-development.md
related:
  - ../avatar/performance-rank.md
  - ../avatar/optimization-guide.md
  - ../world/performance-guide.md
  - mobile-ui-optimization.md
  - cross-platform-content.md
  - unity-install-cn.md
  - ../../concepts/rules/quest-constraints.md
type: entity
created: 2026-07-04
sources: 本地知识库整理
updated: 2026-07-04
---
# Android/Quest Development

> 来源: VRChat 官方文档 (creators.vrchat.com/platforms/android/)
> 置信度: High
---

## Overview

VRChat 支持 Android 平台（Quest + 移动设备）。开发跨平台内容需要特殊考虑。

---

## 平台设置

### Unity Build Settings
```
Edit → Build Settings → Android → Switch Platform
```

### VRChat SDK Control Panel
```
Select Platform: Android
```

### VRChat SDK 面板的 Switch Build Target 按钮（2026-07 补充）

> **来源**: creators.vrchat.com/platforms/android/setting-up-unity-for-creating-quest-content
> **状态**: ✅ FACT-OFFICIAL

#### 官方 3 步流程

**Step 1: Open the VRChat SDK Panel**
1. VRChat SDK → Show Control Panel
2. 切换到 "Builder" 标签

**Step 2: Switch Platform to Android**
1. 在 Build section,点击 "Platform" 下拉
2. 取消选中 "Windows",选中 "Android"
3. 关闭窗口,在弹出窗口点击 "Confirm" 开始切换

**Step 3: Publish a New Build**
- "Build and Publish" 按钮
- 参考 Android Content Limitations + Android Content Optimization

#### 转换过程的预期管理

| 阶段 | 耗时 | 说明 |
|------|------|------|
| **首次转换** | 长(数分钟到数十分钟) | 取决于资产复杂度:纹理/音频/模型数量 |
| **后续转换** | 短 | 只处理**新增/修改**的资产(Unity 缓存机制) |

#### 转换后的必要操作

- **World 项目**: 切换后**必须重新烘焙光照**(光烘焙数据与平台相关)
- **Avatar 项目**: 一般无需特殊处理(除非 PC ↔ Android 资产结构差异大)

#### 常见错误

| 错误信息 | 原因 | 解决 |
|---------|------|------|
| "Android" 选项不可选 | 未安装 Android Build Support | 参考 `unity-install-cn.md` §5 |
| 转换卡住超过 30 分钟 | 大型项目首次转换正常 | 等待,勿中断 |

---

## 内容限制

### Avatar 着色器限制 ⚠️（2026-07-04 完善）

> **来源**: VRChat 官方 creators.vrchat.com/platforms/android/quest-content-limitations
> **状态**: ✅ FACT-OFFICIAL (2025-10-08)

Avatar **只能**使用 `VRChat/Mobile/` 目录下的官方 Shader（SDK 内置）。

**完整可用 Shader 列表**:

| Shader Name | 类型 | 适用场景 |
|------------|------|---------|
| **Toon Standard** ⭐NEW (SDK 3.8.1+) | 卡通 | **最强大的 Toon Shader** - 多数情况下推荐 |
| **Standard Lite** | PBR | Unity Standard 轻量版，支持 Metallic/Occlusion/Smoothness 通道 |
| **Bumped Diffuse** | 卡通 | Diffuse + normal map + vertex colors |
| **Bumped Mapped Specular** | 卡通 | Diffuse + specular map + normal map |
| **Diffuse** | 卡通 | 纯 Diffuse + vertex colors |
| **Matcap Lit** | 卡通 | Diffuse + matcap input + vertex colors（适合模拟金属）|
| **Toon Lit** | 卡通 | 卡通着色 + 阴影（适合平面卡通）|
| **Particles/Additive** | 粒子 | Additive 混合粒子 |
| **Particles/Multiply** | 粒子 | Multiply 混合粒子 |
| **Lightmapped** | 卡通 | 仅 World，Avatar 不可用 |
| **Skybox** | 天空盒 | 仅 World |
| **Supersampled UI** | UI | 仅 World |

**重要性能提示**:
- ✅ 启用 `Enable GPU Instancing` 在所有材质上
- ❌ 不要使用 Bumped 变体（如果没有 normal map）
- ❌ **完全避免透明度**（Alpha fill rate 是 mobile GPU 性能杀手）

**选型决策树**:
```
[需要什么效果?]
   │
   ├─ 标准卡通 + 复杂功能 → Toon Standard (SDK 3.8.1+ 推荐)
   ├─ 标准 PBR + 多通道 → Standard Lite
   ├─ MatCap 金属效果 → Matcap Lit
   ├─ 平面卡通 + 阴影 → Toon Lit
   └─ Diffuse + 简单法线 → Bumped Diffuse
```

**禁止**:
- ❌ 自定义着色器（会导致上传失败）
- ❌ Liltoon / Poiyomi 等社区 Shader（不在 `VRChat/Mobile/` 目录）

**Toon Standard 关键特性** (SDK 3.8.1+ 新增):
- Detail + Emission maps (UV0/UV1)
- Custom shadow ramps
- Rim lighting + Matcaps
- Maskable hue-shift
- Normal maps + tilable detail normals
- **重要**: Outline 版本仅 PC 支持，移动端自动 fallback
- **重要**: 不支持实时阴影

**引用**:
- [FACT-OFFICIAL] https://creators.vrchat.com/platforms/android/quest-content-limitations (2025-10-08)
- [FACT-OFFICIAL] https://creators.vrchat.com/releases/release-3-8-1/ (2025-05-22, Toon Standard 新增)
- VRCD 中文文档: https://docs.vrcd.org.cn/books/vrchat-pc-android

### Android 插件兼容性原则（2026-07-04 新增）

> **来源**: VRChat 官方 Quest Content Limitations + VRCD 中文文档
> **状态**: ✅ FACT-OFFICIAL

#### 兼容性原则

| 插件类型 | 建议 | 原因 |
|---------|------|------|
| **姿势控制类** | ✅ 保留 | 通常使用 Animator + VRChat Constraints |
| **特殊 Shader 插件** | ❌ 完全移除 | 使用的 Shader 不在 `VRChat/Mobile/` 目录 |
| **使用禁用组件的插件** | ❌ 完全移除 | 组件在 Android 上不可用 |
| **不确定兼容性的插件** | ❌ 保险起见移除 | 减少上传失败风险 |

#### Android/Quest 完整禁用组件列表

> **来源**: creators.vrchat.com/platforms/android/quest-content-limitations §Components

| 组件 | 状态 | 备注 |
|------|------|------|
| **Dynamic Bones** | ❌ 完全禁用 | 使用 PhysBones 替代 |
| **Cloth** | ❌ 完全禁用 | — |
| **Cameras** | ❌ Avatar 禁用 | World 可用但需谨慎 |
| **Lights** | ❌ Avatar 禁用 | World 可用但需谨慎 |
| **Video Players** | ⚠️ 限制使用 | 详见 video-players 文档 |
| **Post-Processing** | ❌ 完全禁用 | GPU 不适合 |
| **Audio Sources** | ❌ 完全禁用 | CPU 消耗大，voice 优先 |
| **Physics Objects** | ❌ Avatar 禁用 | Rigidbodies/colliders/joints |
| **Particle Systems** | ⚠️ 严重限制 | 设置镜像 PC Particle Limits |
| **Unity Constraints** | ❌ Avatar 禁用 | 使用 VRChat Constraints 替代 |
| **FinalIK** | ❌ 完全禁用 | 无限制资源消耗 |

#### 常用插件兼容性参考

| 插件 | Android 兼容性 | 备注 |
|------|--------------|------|
| **GOGOLOCO** | ✅ 推荐保留 | 姿势控制类，使用 Animator |
| **Modular Avatar** | ✅ 兼容 | NDMF 工具链核心 |
| **AAO (Avatar Optimizer)** | ✅ 兼容 | NDMF 工具 |
| **VRCFury** | ✅ 兼容 | Non-Destructive 工具 |
| **VRCLens** | ⚠️ 主要 PC | 部分功能可能受限 |
| **VirtualLens2** | ✅ 有 Remote Only Mode | 详见 §VirtualLens2 Remote Only Mode |
| **Liltoon / Poiyomi 等第三方 Shader** | ❌ 不兼容 | 使用 VRChat/Mobile 替代 |

**实践建议**:
- 制作 Android 版本前，**逐个检查**插件的组件
- 不确定时**完全移除**（保险起见）
- 保留的插件应**尽量少**，减少兼容性风险

**引用**:
- [FACT-OFFICIAL] https://creators.vrchat.com/platforms/android/quest-content-limitations (2025-10-08)
- VRCD 中文文档: https://docs.vrcd.org.cn/books/vrchat-pc-android

### PC→Android / Quest 工程组织建议（社区实践补充）

> **来源**: VRCD 中文文档（社区）
> **定位**: 这是高实操价值的社区推荐，不是 VRChat 官方唯一指定流程。

#### 强烈推荐：项目副本策略

| 策略 | 优点 | 缺点 |
|------|------|------|
| ❌ 同工程双版本 | 共享资源 | 目录结构混乱，修改易冲突 |
| ✅ **独立工程副本** | 结构清晰，修改隔离 | 资源复制成本 |

**推荐流程**:
1. 先完成 PC 版本并上传 / 验证。
2. 复制 PC 项目为 `AvatarName-Quest/` 或等价副本。
3. 在副本中删除 Quest 高风险插件、第三方 Shader、非必要外观件与高成本资源。
4. 只在副本内进行 Android / Quest 适配与上传前检查，保持 PC 主工程不受影响。

#### Avatar 裁剪与上传前准备（压缩版）
- 优先删除 **不是“必须保留”的附加外观件**，而不是一开始就微调所有资源。
- 对不确定兼容性的插件，**保险起见先移除**，再逐步回加验证。
- 上传前至少检查：
  - 是否仍有第三方 Shader
  - 是否仍残留 Android 禁用组件
  - 是否存在透明度依赖
  - 是否仍为 Very Poor 且超出移动端可接受范围

### Quest 透明度禁用（2026-07 补充）

> **来源**: VRChat Canny feedback (feedback.vrchat.com/avatar-30/p/unhelpful-error-message-about-alpha-blended-shader-not-supported-on-android)
> **状态**: ✅ FACT-CANNY（VRChat Staff 官方回复）

> **VRChat 官方 staff 明确回复**:
> "This is intentional. Quest avatars don't support transparency for performance reasons."

#### 历史与背景

- `VRChat/Mobile/Particles/Alpha Blended` shader 原本在 `VRChat/Mobile` 分类下
- 但在 Quest 上传时被 SDK 拦截并报错
- VRChat 官方决定**完全禁用** Avatar 端的透明度
- 错误信息已被官方承认为"不准确"（shaders 不应出现在可用列表），正在改进

#### 替代方案

| 需求 | 替代方案 | 平台 |
|------|---------|------|
| **粒子透明** | Particle System + GPU 粒子 | ✅ Quest 支持 |
| **硬边透明** | Cutout Shader（不透明/全透明二值） | ✅ Quest 支持 |
| **半透明** | 无（Quest 禁用） | ❌ 不支持 |
| **Mesh 渐隐** | 无 | ❌ 不支持 |

#### 未来动向（2025-04-24 Developer Update）

- 即将推出 `Toon Standard` Shader（移动端兼容）
- **官方明确**："Performance is a major concern of course, so it will still not support transparency for the time being."
- 即：新 Shader 仍**不支持透明度**（性能是主因）

### 🔴 Very Poor Avatar 未来风险警告（2026-07-04 新增）

> **来源**: VRChat 官方 creators.vrchat.com/avatars/avatar-performance-ranking-system/ §Mobile Default Performance Rank Blocking
> **状态**: ✅ FACT-OFFICIAL DANGER 级别警告

> **VRChat 官方 DANGER 警告**:
> 
> > "Show Avatar" for Very Poor avatars functionality may be removed in the future, and Very Poor avatars may be removed from Android and iOS entirely. Please keep this in mind when creating avatars for VRChat on mobile devices.

**关键风险**:

| 风险 | 状态 | 含义 |
|------|------|------|
| **Show Avatar 强制显示功能** | 🔴 未来可能删除 | 用户不能强制显示被阻止的 Very Poor Avatar |
| **Very Poor Avatar 平台存在** | 🔴 可能从 Android/iOS 完全移除 | 不只是"难以显示"，而是"完全禁止" |

**实践含义**:
- 创作者**不能**依赖"现在能看 = 未来能看"
- 移动端 Avatar 制作应**目标 Excellent/Good**
- 创作者应将此风险告知客户（如果接商业委托）

**官方补充信息**:
- 默认 Minimum Displayed Performance Rank = **Medium**（移动端）
- Medium 意味着**不能看到 Poor 和 Very Poor Avatar**
- 移动端 Show Avatar 一次**只能显示 3-5 个 Very Poor Avatar**

**引用**:
- [FACT-OFFICIAL] https://creators.vrchat.com/avatars/avatar-performance-ranking-system/
- [EXTERNAL] https://help.vrchat.com/hc/en-us/articles/360062658133
- 完整评估: （来源：本地参考笔记）

### World 着色器
World 不受着色器限制，但：
- 必须高度优化
- 推荐使用 `Mobile/VRChat/Lightmapped`
- 必须烘焙光照

---

## 跨平台 Avatar 要求

### Armature 匹配 ⚠️
PC 和 Android 版本的 **骨架路径必须完全相同**：
```
Armature (root)
└── Hips (root bone) ← 必须一致
    └── Spine
        └── ...
```

### Root Bone 要求
- **Hips** 是 root bone
- PC 和 Android 的 **scale 和 rotation 必须相同**
- 可以移除非必要骨骼（裙子、头发等）
- **禁止**改变基础骨架结构

### 常见问题
骨架不匹配会导致：
- 奇怪的身体扭曲
- 跨平台查看异常
- 动画错位

---

## 性能优化

### Geometry
| 指标 | Quest 标准 |
|------|-----------|
| **三角形** | ≤70,000 (Excellent: 32,000) |
| **材质数** | ≤16 (Excellent: 2) |
| **Draw Calls** | 尽量减少 |

### Textures
- 最大分辨率: 1024x1024 (1K)
- 使用纹理图集
- 降低 "Max Size" 在导入设置中

### Lighting
- **必须烘焙**光照
- 避免实时灯光
- 使用 Light Probes
- 低 lightmap 分辨率

### Occlusion Culling
- **必须烘焙**遮挡剔除
- 减少 GPU 负载
- 设置简单但有效

---

## Content Optimization Checklist

### Worlds
- [ ] 烘焙光照（不是可选，是必须）
- [ ] 降低几何复杂度
- [ ] 避免透明度
- [ ] 降低纹理分辨率
- [ ] 烘焙遮挡剔除
- [ ] 使用优化的着色器

### Avatars
- [ ] 移除多余组件
- [ ] 减少骨骼数量
- [ ] 降低多边形数量
- [ ] 避免透明度
- [ ] 减小纹理大小
- [ ] 仅使用 Mobile Shaders

---

## 工具推荐

### EasyQuestSwitch
VCC 包，可自动调整跨平台内容：
```
VCC → Manage Packages → EasyQuestSwitch
```

### SDK 示例
SDK 包含 Quest 优化示例。

---

## 发布流程

1. 在 Android 平台开发和测试
2. 优化所有性能指标
3. Build & Publish
4. 世界自动在 Quest 和 Android 移动设备上可用

---

## 移动设备差异

| 设备 | 屏幕分辨率 | 性能 |
|------|-----------|------|
| Quest 2/3 | 高 | 中 |
| Android 手机 | 中-低 | 低 |
| Android 平板 | 中 | 中 |

> 好的 Quest 世界在手机上也会有良好表现

---

## 相关文档

- [[entities/avatar/performance-rank]] - 性能排名标准
- [[entities/avatar/optimization-guide]] - Avatar 优化
- [[entities/world/performance-guide]] - World 优化
- [[entities/platform/mobile-ui-optimization]] - 移动端 UI
- [[entities/platform/cross-platform-content]] - 跨平台内容

---

## VirtualLens2 Remote Only Mode（2026-07-04 补充）

> 来源: VirtualLens2 Changelog v2.10.0

### 概述

VirtualLens2 v2.10.0 引入了 **Remote Only Mode**（远程仅模式），专为 Android 移动平台设计。

### 行为

- 当其他玩家查看使用此模式的 Avatar 时，**仅设置显示头像的必要功能**
- 省略摄影、对焦、无人机等"本地"功能
- 减小 Avatar 在远程渲染时的资源占用

### 适用场景

- 同时面向 PC 和 Android 的 Avatar
- 希望在 Android 上保持低性能开销

### 重要注意

- VirtualLens2 主功能仍是 **PCVR only**（BOOTH 明确说明）
- Remote Only Mode 是一种**部署优化**而非完整的 Android 支持
- 完整的 Android 摄影功能在 VirtualLens2 路线图中尚未支持

### 引用

- [L2] VirtualLens2 Changelog: <https://vlens2.logilabo.dev/en/docs/changelog/>
- 详见 [[entities/avatar/camera-tools]] §2.2
---

## 相关页面

[[../avatar/performance-rank.md]] · [[../avatar/optimization-guide.md]] · [[../world/performance-guide.md]] · [[mobile-ui-optimization.md]] · [[cross-platform-content.md]] · [[unity-install-cn.md]] · [[../../concepts/rules/quest-constraints.md]]
