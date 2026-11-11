---
title: "Platform Domain — 跨平台开发知识库"
category: platform
knowledge_level: applied
status: active
source: 本地知识库整理
source_type: community
version: 1.0
last_review: 2026-06-20
confidence: Medium
tags:
  - misc
  - index
  - navigation
aliases:
  - "Platform Domain — 跨平台开发知识库"
related:
  - "[[android-development.md]]"
  - "[[cross-platform-content.md]]"
  - "[[mobile-ui-optimization.md]]"
  - "[[easyquestswitch.md]]"
  - "[[unity-recorder.md]]"
  - "[[unity-admin-warning.md]]"
  - ../../[[../avatar/optimization-guide.md]]
  - ../../[[../avatar/ndmf-tools.md]]
  - ../../[[../avatar/modular-avatar.md]]
  - ../../[[../world/performance-guide.md]]
  - ../../[[../world/vrc-light-volumes.md]]
type: entity
created: 2026-06-20
sources: 本地知识库整理
updated: 2026-06-20
---
# Platform Domain — 跨平台开发知识库

**Domain**: Platform(跨平台开发)
**Type**: Knowledge Base
**本地化日期**: 2026-07-04
**文件数**: 12

---

## 概述

**关键事实**:
- VRChat 跨平台 ≠ 一次构建到处运行
- PC 和 Quest 是**两套独立的性能预算**(Quest 严格得多)
- 资源、Shader、UI 都需要平台特定优化
- 推荐使用 **EasyQuestSwitch** 自动化平台切换

**Platform Domain** 涵盖 VRChat 跨平台(PC / Android / Quest / iOS)开发的关键知识,包括 Quest 性能约束、跨平台资源策略、移动端 UI 优化、PC/Quest 自动化切换工具。


---

## 文档分类

### Quest / Android 平台(1)

| 文档 | 内容 |
|------|------|
| [android-development.md](android-development.md) | Quest 开发、着色器限制、Armature 要求、工程副本策略、上传前准备 |
| [[entities/platform/meta-quest-troubleshooting|meta-quest-troubleshooting]] ⭐NEW 2026-07-13 | Meta Quest 设备 / 系统 / 运行层社区排障候选专题 |

### 跨平台内容策略(1)

| 文档 | 内容 |
|------|------|
| [cross-platform-content.md](cross-platform-content.md) | 平台切换、Avatar 一致性要求 |

### 移动端 UI 优化(1)

| 文档 | 内容 |
|------|------|
| [mobile-ui-optimization.md](mobile-ui-optimization.md) | VRC_UIShape、Focus View、触摸目标 |

### PC/Quest 自动化切换工具(1)

| 文档 | 内容 |
|------|------|
| [easyquestswitch.md](easyquestswitch.md) | **EasyQuestSwitch** PC/Quest 平台切换(VRChat 官方)|

### Unity 工具与故障排查 (2) ⭐NEW 2026-07-04

| 文档 | 内容 |
|------|------|
| [unity-recorder.md](unity-recorder.md) | **Unity Recorder** 截图/录屏(Editor-only 工具 + VRChat Unity LTS 版本兼容) |
| [unity-admin-warning.md](unity-admin-warning.md) | **Unity 管理员权限警告** 排查(UAC + 兼容性 + UUM-42508) |

### 其他专题(2)

| 文档 | 内容 |
|------|------|
| [[entities/platform/account-registration-cn|VRChat 账号注册 — 国内网络环境指南]] | 国内网络环境下的 VRChat 账号注册流程与注意事项 |
| [[entities/platform/backup-strategy|VRChat 创作者备份策略]] | VRChat 创作者项目与数据备份策略 |

---

## 关键概念

### 1. Quest 性能预算 vs PC

| 维度 | PC | Quest(Android)|
|------|-----|---------------|
| **GPU** | 桌面级 | Adreno 650(约等于 GTX 1050 的 60%)|
| **CPU** | 桌面级 | Snapdragon XR2 Gen 2 |
| **内存** | 8-16 GB+ | 8 GB 共享 |
| **目标 FPS** | 90 FPS | 72-90 FPS |
| **Draw Call 预算** | 高 | < 100(中等复杂度)|
| **Tri 预算** | 高 | < 500K |
| **Shader** | 完整 PBR | Quest 优化版(Lite / Mobile)|

### 2. Avatar 跨平台要求

- 完整版 + Lite 版(Quest 降级)
- 材质数量限制
- Shader Feature 限制
- 骨骼数量限制
- 表达式参数限制

### 3. World 跨平台要求

- 动态光源数量限制
- 实时光照关闭,改用 Lightmap / Light Volumes
- 后处理简化
- 纹理大小限制

---

## 推荐工作流

### Avatar 跨平台

```
1. 设计 PC 版(LilToon 完整功能)
   ↓
2. 复制为 Lite 版本(Quest 兼容 Shader)
   ↓
3. 简化材质数量 / 骨骼数
   ↓
4. 使用 EasyQuestSwitch 自动化切换
   ↓
5. 在 Quest 真机测试
```

### World 跨平台

```
1. PC 烘焙 Lightmap(Bakery)
   ↓
2. Quest 端使用 Light Volumes + 简化材质
   ↓
3. 移除昂贵后处理(Bloom/DoF 等)
   ↓
4. 验证目标 FPS
```

---

## 相关知识库

- **`avatar/optimization-guide.md`**:Avatar 性能优化
- **`avatar/ndmf-tools.md`**:NDMF 工具链(支持跨平台)
- **`avatar/modular-avatar.md`**:Modular Avatar 插件
- **`world/performance-guide.md`**:World 性能优化
- **`world/vrc-light-volumes.md`**:VRCLightVolumes 跨平台光照
- **`world/bakery/`**:Bakery GPU Lightmapper

---

## 工具与插件

| 工具 | 类型 | 说明 |
|------|------|------|
| **EasyQuestSwitch** | VRChat 官方 | PC/Quest 平台切换自动化 |
| **Modular Avatar** | 社区 | Avatar 自动化工具 |
| **NDMF** | Non-Destructive Modular Framework | 改模工具链 |

---

**最后更新**:2026-06-15
