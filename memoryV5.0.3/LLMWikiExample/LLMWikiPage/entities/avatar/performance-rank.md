---
title: Avatar Performance Ranking System
category: avatar

knowledge_level: applied
status: active

tags:
  - avatar
  - physbone
  - performance
  - light

aliases:
  - "Performance Rank"
  - "Avatar 性能分级"
  - 性能等级
  - 性能排名

related:
  - optimization-guide.md
  - ndmf-tools.md
  - playable-layers.md
  - ../world/performance-guide.md
  - ../api/dynamics.md
  - ../api/networking.md
  - vrc-constraints.md
  - modular-avatar.md
  - ../../concepts/rules/performance-rules.md
  - avatar-dynamic-bone-limits.md
  - avatar-particle-system-limits.md
  - avatar-fallback-system.md

source: 本地知识库整理
source_type: community
version: 1.2
last_review: 2026-07-04
changelog: "2026-07-04 §Quest 平台标准 追加 2026 官方数字确认注释"
confidence: High
type: entity
created: 2026-07-04
sources: 本地知识库整理
updated: 2026-07-04
---
# Avatar Performance Ranking System


---

## PC 平台 Performance Rank 标准

| 指标 | Excellent | Good | Medium | Poor |
|------|-----------|------|--------|------|
| **面数 (Polygons)** | 32,000 | 70,000 | 70,000 | 70,000 |
| **材质内存 (Texture Memory)** | 40MB | 75MB | 110MB | 150MB |
| **Skinned Mesh Renderer** | 1 | 2 | 8 | 16 |
| **材质球 (Material Slots)** | 4 | 8 | 16 | 32 |
| **PhysBone 元件数** | 4 | 8 | 16 | 32 |
| **PB 影响骨头数** | 16 | 64 | 128 | 256 |
| **PB 碰撞器数** | 4 | 8 | 16 | 32 |
| **碰撞检测数** | 32 | 128 | 256 | 512 |
| **骨头数 (Bones)** | 75 | 150 | 256 | 400 |
| **光源 (Lights)** | 0 | 0 | 0 | 1 |
| **粒子系统数** | 0 | 4 | 8 | 16 |
| **粒子总数量** | 0 | 300 | 1,000 | 2,500 |
| **粒子系统面数** | 0 | 1,000 | 2,000 | 5,000 |
| **粒子系统轨迹** | 禁用 | 禁用 | 启用 | 启用 |
| **粒子系统碰撞** | 禁用 | 禁用 | 启用 | 启用 |

> ⚠️ **Very Poor**: 超过 Poor 标准即为 Very Poor

---

## Quest (Mobile) 平台标准

> Quest 标准极为严格，Very Poor 会强制不显示
> 📅 数据更新时间: 2026-07-04 (来源: VRChat 官方 docs + VRCD 中文文档)

| 指标 | Excellent | Good | Medium | Poor |
|------|-----------|------|--------|------|
| **面数 (Triangles)** | 7,500 | 10,000 | 15,000 | 20,000 |
| **Bounds Size** | 2.5m³ | 4m³ | 5m×6m×5m | 5m×6m×5m |
| **材质内存 (Texture Memory)** | 10 MB | 18 MB | 25 MB | 40 MB |
| **Skinned Meshes** | 1 | 1 | 2 | 2 |
| **Basic Meshes** | 1 | 1 | 2 | 2 |
| **Material Slots** | 1 | 1 | 2 | 4 |
| **Animators** | 1 | 1 | 1 | 2 |
| **Bones** | 75 | 90 | 150 | 150 |
| **PhysBones Components** | 0 | 4 | 6 | 8 |
| **PB 影响骨头数 (Affected Transforms)** | 0 | 16 | 32 | 64 |
| **PhysBones Colliders** | 0 | 4 | 8 | 16 |
| **碰撞检测数 (Collision Check Count)** | 0 | 16 | 32 | 64 |
| **Avatar Dynamics Contacts** | 2 | 4 | 8 | 16 |
| **Constraint Count** ⭐NEW | 30 | 60 | 120 | 150 |
| **Constraint Depth** ⭐NEW | 5 | 15 | 35 | 50 |
| **Particle Systems** | 0 | 0 | 0 | 2 |
| **Total Particles Active** | 0 | 0 | 0 | 200 |
| **Mesh Particle Active Polys** | 0 | 0 | 0 | 400 |
| **Particle Trails Enabled** | 关 | 关 | 关 | 开 |
| **Particle Collision Enabled** | 关 | 关 | 关 | 开 |
| **Trail Renderers** | 0 | 0 | 0 | 1 |
| **Line Renderers** | 0 | 0 | 0 | 1 |
| **Raycasts** ⭐NEW | 1 | 2 | 4 | 8 |

> **官方新增指标说明** (2026-07-04 补充):
> - **Constraint Count / Depth**: VRChat Constraints + Unity Constraints 计数
> - **Raycasts**: VRCRaycast 射线检测 (SDK 3.10.3+)，详见 [[entities/avatar/vrcraycast]]

> 📖 参考:
> - [VRChat 官方 Avatar Performance Ranking 文档](https://creators.vrchat.com/avatars/avatar-performance-ranking-system/) (2026-04-21)
> - [VRChat 官方 Quest Content Limitations](https://creators.vrchat.com/platforms/android/quest-content-limitations) (2025-10-08)
> - VRCD 中文文档: https://docs.vrcd.org.cn/books/vrchat-pc-android (2024-2025)

> **2026-07-04 数据修正**:
> - Quest 标准表已用 VRChat 官方 2026-04-21 数据替换
> - 原 Quest 表数据为 PC 标准数据搬运错误，已全面修正
> - 新增 Constraint Count / Depth / Raycasts 等官方指标
> - 详细来源: （来源：本地参考笔记）

> **2026-07 数字确认**:
> - Avatar 推荐: ≤ 10,000 三角形
> - Quest Fallback: Good 评级 = ≤ 10,000 三角形
> - 来源: creators.vrchat.com/platforms/android/quest-content-optimization
>
> 详细 Fallback 硬性规则: [[entities/avatar/avatar-fallback-system]] §1.1

---

## 🔴 DANGER: Very Poor 评级未来可能删除（2026-07-04 新增）

> **来源**: VRChat 官方 creators.vrchat.com/avatars/avatar-performance-ranking-system/ §Mobile Default Performance Rank Blocking
> **状态**: ✅ FACT-OFFICIAL DANGER 级别警告

> **VRChat 官方 DANGER 警告原文**:
> 
> > "Show Avatar" for Very Poor avatars functionality may be removed in the future, and Very Poor avatars may be removed from Android and iOS entirely. Please keep this in mind when creating avatars for VRChat on mobile devices.

**关键风险**:

| 风险 | 状态 | 含义 |
|------|------|------|
| **Show Avatar 强制显示功能** | 🔴 未来可能删除 | 用户不能强制显示被阻止的 Very Poor Avatar |
| **Very Poor Avatar 平台存在** | 🔴 可能从 Android/iOS 完全移除 | 不只是"难以显示"，而是"完全禁止" |

**创作者建议**:
- 目标 **Excellent 或 Good 评级**，避免 Very Poor 风险
- 不要依赖"Show Avatar"作为长期方案
- 移动端 Avatar 必须考虑此长期风险

**官方补充信息**:
- 默认 Minimum Displayed Performance Rank = **Medium**（移动端）
- Medium 意味着**不能看到 Poor 和 Very Poor Avatar**
- 移动端 Show Avatar 一次**只能显示 3-5 个 Very Poor Avatar**

**引用**:
- [FACT-OFFICIAL] https://creators.vrchat.com/avatars/avatar-performance-ranking-system/
- [EXTERNAL] https://help.vrchat.com/hc/en-us/articles/360062658133
- 详细评估: （来源：本地参考笔记）

---

## ⚠️ 与"运行时客户端限制系统"的区分(2026-06-30 新增)

本文档是**上传时** SDK 评估的 Performance Rank(粗粒度指标)。VRChat 客户端还有**独立的运行时限制系统**(用户可配置):

| 限制系统 | 触发时机 | 默认值 | 文档 |
|---------|---------|--------|------|
| **Dynamic Bone Limits** | 运行时 | Max Affected=32, Max Collision=8 | [[entities/avatar/avatar-dynamic-bone-limits]] |
| **Particle System Limits** | 运行时 | ps_max_particles=50000 等 11 变量 + Penalty 公式 | [[entities/avatar/avatar-particle-system-limits]] |
| **Minimum Performance Rank** | 用户配置 | N/A(用户在 Settings 设置) | [[entities/avatar/avatar-fallback-system]] |

**关键区别**:
- **Performance Rank**(本文档): 创作者控制,通过优化 Avatar 改进
- **运行时 Limits**: 玩家控制,通过修改 `config.json` 调整
- 两者独立但相关(默认值匹配 Medium 等级)

---

## 常见导致 Very Poor 的原因

1. **Skinned Mesh Renderer 过多** — 最常见的 CPU 杀手
2. **材质球过多** — 可以填满一座泳池
3. **PhysBone 元件过多** — 走在路上都能踢到
4. **面数过多** — 让 3D 龙看了直摇头
5. **贴图过大** — 非常「肥美」的贴图

---

## 最佳化优先级（从易到难）

```
1. Light (光源)          → 能不要就不要
2. Particle System      → 控制数量
3. Texture Memory       → 降分辨率/压缩
4. Skinned Mesh         → 合并
5. Material Slots       → Atlas 化
6. PhysBones            → 合并 + 精简
7. Bones                → 合并
8. Polygons             → Remove + Simplify
```

---

## 相关文档

- `ndmf-tools.md` — NDMF 工具生态与获取方式
- `optimization-guide.md` — 完整最佳化实操指南

---

## 本地对象污染性能排名（2026-07-04 新增）⭐

> 🔴 **未解决的已知问题**

### 问题描述

- VRChat Avatar Performance Rank **不区分本地对象和远程对象**
- 像 VirtualLens2 / VRCLens 这样复杂的本地小把戏会**显著拉低**排名
- 但本地对象**对其他玩家无任何影响**
- 这是 **200+ voters** 反馈但**未解决**的 Canny issue

### 引用证据

> "Avatars may contain GameObjects that are used only locally. For example, camera extensions and inventory systems based on Avatar Dynamics. I think that they should be excluded from performance rank calculation if they are always disabled in remote environments."
> — logi_9 (VirtualLens2 作者), 反馈发起人
> — 200+ voters, 2022-04-26 创建, **未解决**

### 对创作者的影响

1. **排名污染**: 本地装饰/工具会拉低 Performance Rank
2. **优化悖论**: 创作者为本地体验加的"小把戏"反而影响被他人看到的评分
3. **第三方工具代价**: VirtualLens2 / VRCLens 等工具的使用会显著影响性能排名

### 建议

- 在 Profile 中说明本地对象的存在
- 关注官方反馈页状态
- 教学场景中明确说明这一限制

### 引用

- [EXTERNAL] VRChat Canny - Mark local objects explicitly: <https://feedback.vrchat.com/avatar-30/p/feedback-mark-local-objects-explicitly-and-exclude-them-from-performance-rank-ca>
- 详细背景见 [[entities/avatar/camera-tools]] §1.3

---

## SPS (Super Plug Shader) 性能影响（2026-07-04 新增）

> 🔴 **关键事实**：使用 SPS 会直接触发 Avatar 性能等级降为 **Very Poor**。

### 单 Socket 触发 Very Poor

- **关键事实**：单个 SPS Socket 包含 **2 个 lights**
- 即使 lights 被禁用也不影响渲染性能
- 但 **Socket 本身的存在**就会使 Avatar 性能等级变为 **Very Poor**

### 与 PC/Quest 光源预算的关系

| 平台 | 光源预算 (Poor) | SPS 单 Socket 占用 | 影响 |
|------|----------------|-------------------|------|
| **PC** | 1 light | 2 lights | 直接触发 Very Poor |
| **Quest** | 0 lights | 2 lights | 严重超出（Quest Very Poor 不显示） |

### 性能优化策略

- **尽可能少用 Socket**（每个 Socket 都会触发 Very Poor）
- **不要用 Constraints 切换多个 Socket**（抖动 + 性能损耗 + SPS 菜单功能失效）
- 充分利用 VRCFury 的 Point Light 自动管理机制
- 详细警告见 [[entities/avatar/vrcfury-reference]] §2.4.4

### 引用

- [FACT] https://vrcfury.com/sps/constraints/（VRCFury 官方 Constraints 专题文档）
- [FACT] https://vrcfury.com/sps/（VRCFury 官方 SPS 文档）

---

## 相关页面

[[optimization-guide.md]] · [[ndmf-tools.md]] · [[playable-layers.md]] · [[../world/performance-guide.md]] · [[../api/dynamics.md]] · [[../api/networking.md]] · [[vrc-constraints.md]] · [[modular-avatar.md]] · [[../../concepts/rules/performance-rules.md]] · [[avatar-dynamic-bone-limits.md]] · [[avatar-particle-system-limits.md]] · [[avatar-fallback-system.md]]
