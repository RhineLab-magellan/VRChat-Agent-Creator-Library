---
title: "VRChat Avatar 3.0 Emulator (Av3Emulator) — Unity 测试工具"
category: avatar

knowledge_level: applied
status: active

tags:
  - avatar
  - testing
  - debugging
  - emulator
  - expression-menu
  - animator

aliases:
  - Av3Emulator
  - Avatar 3.0 Emulator
  - Lyuma Av3 Emulator
  - AV3 Emulator
  - Av3 Emu
  - 头像 3.0 模拟器

related:
  - expression-menu.md
  - playable-layers.md
  - animator-system.md
  - modular-avatar.md
  - avatar-30-toggle-tools.md
  - avatar-modding-guide.md

source: https://github.com/lyuma/Av3Emulator
source_type: community
version: 3.4.13+ (持续更新)
last_review: 2026-07-04
confidence: High
type: entity
created: 2026-07-04
sources: https://github.com/lyuma/Av3Emulator
updated: 2026-07-04
---

# VRChat Avatar 3.0 Emulator (Av3Emulator)

> **来源**:
> - GitHub: https://github.com/lyuma/Av3Emulator
> - Booth: https://booth.pm/en/items/3480610
> - VPM Catalog: https://vpm-catalog.vercel.app/packages/lyuma.av3emulator/
> - VPM 包名: `lyuma.av3emulator`
>
> **本地化日期**: 2026-07-04
> **状态**: ✅ FACT (lyuma 官方 + 571+ stars + VPM Catalog 收录 + 多源引用)

---

## 概述

> [FACT] **Avatar 3.0 Emulator** 是在 Unity 编辑器中**模拟 VRChat Avatar 3.0 系统**的工具，基于 Unity PlayableGraph API 实现。

> **🔴 关键事实**:
> - **不模拟 VRChat 客户端**，**模拟的是 Unity PlayableGraph 上的 Avatar3 重新实现**
> - 571+ stars，活跃维护
> - **被官方推荐**（VRChat 创作者生态标准测试工具）
> - 与 **GestureManager** 共用可获得美观的径向菜单
> - **设计目标**：在不上传 Avatar 到 VRChat 的情况下，提前测试 Expression Menu / 动画 / 参数 / Tracking 等行为

---

## 1. 核心功能

### 1.1 模拟功能

> [FACT] 来源: https://github.com/lyuma/Av3Emulator

| 功能 | 说明 |
|------|------|
| 模拟 Avatar3 的大部分特性 | 基于 PlayableGraph 重实现 |
| **非本地同步测试** | 通过复制或勾选 "Create Non Local Clone" |
| **Animator 窗口实时查看和编辑** | 使用 "Animator To Debug" 下拉 |
| Inspector 显示 Tracking / Animation 状态 | |
| **Gesture 左右权重测试** | 模拟 Fist 手势强度 |
| 自定义 Expression Menu 测试 | 通过 Lyuma Av3 Menu |
| 参数查看和编辑 | Animator 窗口 / Blend Tree / Parameter Driver / 手动 |
| Viseme 测试 | 参数 + 内置 blend shape（0% / 100%） |

### 1.2 输入自定义 Stage 参数

- 使用 **Lyuma Av3 Menu** 下的 Expression Menu
- 使用 Animator 窗口 Parameters 标签页（先选 "Animator To Debug"）
- 手动：alt-click 展开 Floats / Ints 段

### 1.3 API 原理

> [FACT] 基于 Unity 的 PlayableGraph API：
> - [AnimationControllerPlayable](https://docs.unity3d.com/2018.4/Documentation/ScriptReference/Animations.AnimatorControllerPlayable.html)
> - [AnimationLayerMixerPlayable](https://docs.unity3d.com/2018.4/Documentation/ScriptReference/Animations.AnimationLayerMixerPlayable.html)
> - **PlayableGraph** 是 Unity 2018+ 引入的底层动画合成 API

### 1.4 Local Parameters 支持（v3.1.0+）

> [FACT] 来源: https://github.com/lyuma/Av3Emulator/issues/76
> 2023-06-02: 3.1.0+ 支持 **local parameters**（PR #102 修复 issue #76）
> - 修复前：未在 ExpressionParameters 注册的参数会被 Emulator 静默修改 Animator 状态
> - 修复后：Expression Menu 在非注册参数上不响应（与 VRChat 行为一致）

---

## 2. 使用方法

### 2.1 启用 Emulator

```
1. Tools 菜单 → Avatar 3.0 Emulator
2. 工具会向场景添加一个对象（可删除）
3. 设置默认 VR 模式、Tracking 类型、Animator to Debug
```

### 2.2 在 Animator 窗口调试

```
1. 选中 Avatar
2. Windows > Animation > Animator
3. 选 controller
4. "Animator To Debug" 下拉选择
5. 在 controller 内交互：
   - 2D Blend Tree 移动红点
   - Upright slider（Crouch/Prone）
   - 坐姿测试
   - AFK 测试
```

### 2.3 测试非本地同步

```
1. 复制 Avatar（或勾选 "Create Non Local Clone"）
2. 单独操作一份
3. 观察另一份的状态变化
```

### 2.4 配合 GestureManager 使用

> [FACT] 来源: https://github.com/lyuma/Av3Emulator/releases
> "GestureManager, installed from VCC, will provide the nicer looking radial menu."

- 安装 GestureManager（VCC）
- 配合 Av3Emulator 提供美观径向菜单
- Creator Companion 不是必须的，Standalone Avatar SDK 也支持

---

## 3. 已知功能改进

| 版本/日期 | 改进 | 来源 |
|---------|------|------|
| 2023-06-02 (v3.1.0+) | 支持 **local parameters**（PR #102 修复 issue #76） | Issue #76 |
| 2024-01+ | Inspector 包含 "User Inputs" 段（手动驱动 synced parameters） | Issue #143 |
| 持续 | OSC 支持（可与 VRCFaceTracking 配合） | Issue #143 |
| 当前 (v3.4.13+) | 持续维护中 | VPM Catalog |

---

## 4. 兼容性

| 项目 | 要求 | 来源 |
|------|------|------|
| Unity | 2019.4 或更新 | VPM Catalog |
| SDK | VRChat SDK - Avatars（推荐 VCC 安装） | GitHub |
| 配套 | GestureManager（提供美观径向菜单） | GitHub |

---

## 5. 与 Avatar 创建工作流的集成

### 5.1 测试 Toggle 流程

```
创建 Toggle（VRChat Toggle Assistant / MA Object Toggle / 手动）
  ↓
Av3Emulator 启用
  ↓
选中 Avatar
  ↓
"Animator To Debug" 选择 FX Layer
  ↓
打开 Lyuma Av3 Menu
  ↓
点击 Toggle → 观察 Animator 状态变化
  ↓
满意后上传 VRChat
```

### 5.2 调试常见问题

| 症状 | Av3Emulator 调试方法 |
|------|---------------------|
| Toggle 不响应 | 检查 Parameter 是否在 Expression Parameters 中注册 |
| 动画不播放 | 检查 FX Layer 中是否有 State 切换 |
| Blendshape 不变 | Animator To Debug 选对应 layer |
| Synced 不同步 | 复制 Avatar 测试非本地同步 |
| OSC 不响应 | 启用 OSC + 配合 VRCFaceTracking 测试 |

### 5.3 调试与生产环境差异

> ⚠️ Av3Emulator **不模拟** VRChat 客户端的全部行为：
> - 真实同步延迟
> - 真实玩家数量
> - 真实 Quest 性能
> - 真实 Safe Mode / 错误处理

→ **Av3Emulator 是开发期工具，最终必须上传测试**

---

## 6. 与 Modular Avatar 工作流

- **MA Object Toggle** 创建的 toggle 在 Av3Emulator 中**完全可测**
- 编译后（Apply On Play）的 avatar 与 Av3Emulator 行为一致
- 与 **GestureManager** 共用测试菜单交互

---

## 7. 安装方式

### 7.1 VCC / ALCOM（推荐）

```
包名: lyuma.av3emulator
VPM 源: https://lyuma.github.io/Av3Emulator/vpm.json
```

### 7.2 Booth

```
下载链接: https://booth.pm/en/items/3480610
导入 .unitypackage 到 Unity 项目
```

### 7.3 GitHub Release

```
https://github.com/lyuma/Av3Emulator/releases
下载最新版本
```

---

## 8. Missing Information（【未确认】项）

1. ❓ 当前最新版本（截至 2026-07-04 未深入核对）
2. ❓ 是否支持 New Input System 完整手势模拟
3. ❓ 与 VRCFaceTracking OSC 集成的具体配置
4. ❓ Quest 平台 Avatar 在 Editor 中的模拟精度
5. ❓ 与 VRCFury Parameter Compressor 的兼容性

---

## 来源

- GitHub: https://github.com/lyuma/Av3Emulator
- Booth: https://booth.pm/en/items/3480610
- VPM Catalog: https://vpm-catalog.vercel.app/packages/lyuma.av3emulator/
- VPM 包名: `lyuma.av3emulator`
- Issues: https://github.com/lyuma/Av3Emulator/issues
- 相关: [[entities/avatar/expression-menu]] - Expression Menu 三层结构
- 相关: [[entities/avatar/avatar-30-toggle-tools]] - Toggle 创建工具生态
- 相关: [[entities/avatar/modular-avatar]] - MA 路径
- 本地化参考: `笔记/09-avatar-30-toggle/K03-avatar-30-emulator.md`

---

## 相关页面

[[expression-menu.md]] · [[playable-layers.md]] · [[animator-system.md]] · [[modular-avatar.md]] · [[avatar-30-toggle-tools.md]] · [[avatar-modding-guide.md]]
