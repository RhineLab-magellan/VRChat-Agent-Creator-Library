---
title: Cross-Platform Content
category: platform

knowledge_level: applied
status: active

tags:
  - platform
  - avatar
  - udonsharp

aliases:
  - "Cross Platform"
  - 跨平台
  - ../../PC/Quest 兼容
  - Cross-Platform

related:
  - easyquestswitch.md
  - mobile-ui-optimization.md
  - ../avatar/optimization-guide.md
  - ../world/performance-guide.md
  - android-development.md

source: 本地知识库整理
source_type: community
version: 1.1
last_review: 2026-07-04
changelog: "2026-07-04 新增 §跨平台资源差异化（Override per platform 机制）"
confidence: High
type: entity
created: 2026-07-04
sources: 本地知识库整理
updated: 2026-07-04
---
# Cross-Platform Content


---

## Overview

VRChat 支持多平台（PC VR/Desktop、Android Quest、移动设备）。创建跨平台内容需要考虑平台差异。

---

## 平台支持

| 平台 | VR/Desktop | Quest | Mobile |
|------|------------|-------|--------|
| **Windows** | ✅ | - | - |
| **Quest** | - | ✅ | - |
| **Android Phone** | - | - | ✅ |
| **iOS** | - | 实验性 | ✅ |

---

## 平台切换

### Unity Build Settings
```
Edit → Build Settings
选择目标平台 → Switch Platform
```

### VRChat SDK
```
SDK Control Panel → Select Platform
```

## 跨平台资源差异化（Override per platform）（2026-07 补充）

> **来源**: VRCD 20-vowgan.md (COMMUNITY) + Unity Manual
> **状态**: ⚠️ COMMUNITY (Unity 标准机制)

### Override per platform 机制

Unity 提供 `Override per platform` 功能,允许为不同平台**单独设置资产导入参数**。

**路径**: Inspector 选中资源 → 在 Import Settings 中找到对应平台选项 → 启用 Override

### 典型用法: World 纹理降采样

| 平台 | Max Size | Compression |
|------|----------|-------------|
| **PC（默认）** | 2048 (2K) | 自动 |
| **Android Override** | 1024 (1K) | ASTC |

**效果**: 同一纹理资产在 PC 和 Android 上自动使用不同分辨率。

### Avatar 跨平台建议

| 资源类型 | PC 设置 | Quest 设置 |
|---------|---------|-----------|
| **主贴图** | 2048×2048 | 1024×1024 |
| **法线贴图** | 1024×1024 | 512×512 |
| **头发贴图** | 1024×1024 | 512×512 |
| **Mesh 复杂度** | 100% | 70-80% |

### 优势

- ✅ PC 端不受影响（保持高质量）
- ✅ Quest 端自动降级（满足大小限制）
- ✅ 单一项目同时维护两个版本

### 工具辅助

- **EasyQuestSwitch**: 自动处理材质/光照/活动 GameObject
- **lilAvatarUtils**: 批量处理 Avatar 贴图大小
- **VRWorld Toolkit**: 构建后文件分析

---

## Avatar 跨平台要求

### Skeleton 一致性 ⚠️
PC 和 Android avatar 的 **armature 路径必须完全相同**：

```
✓ Correct
Armature
└── Hips ← root bone (scale/rotation must match)
    └── Spine
        └── Chest
            └── ...

✗ Incorrect  
Armature
└── Root (different name!)
    └── Hips
        └── ...
```

### Root Bone 规则
- **Hips** 是标准的 root bone
- PC 和 Android 版本的 Hips **scale 和 rotation 必须相同**
- 可以移除非必要骨骼（裙子、头发、尾巴等）
- **禁止**改变基础骨架层级结构

### 最佳实践
1. 从同一源文件创建 PC 和 Android 版本
2. 移除而非修改基础骨骼
3. 使用相同命名规范
4. 导出前验证 armature 结构

### PC→Android 工程组织建议（2026-07-04 新增）

> **来源**: VRCD 中文文档 https://docs.vrcd.org.cn/books/vrchat-pc-android
> **状态**: 🔶 社区推荐（VRChat 官方有相关页面但未明确推荐）

#### 强烈推荐：项目副本策略

| 策略 | 优点 | 缺点 |
|------|------|------|
| ❌ 同工程双版本 | 共享资源 | 目录结构混乱，修改易冲突 |
| ✅ **独立工程副本** | 结构清晰，修改隔离 | 资源复制成本 |

**具体步骤**:
1. 复制 PC 项目为 `AvatarName-Quest/`
2. 在副本上制作 Android 版本
3. 保持 PC 版本不变
4. 两个项目独立维护、分别上传

**经验提醒**:
- 该策略的核心收益不是“技术上唯一可行”，而是**把跨平台改造风险隔离**，避免 Quest 适配反向污染 PC 主工程。
- 如果项目大量使用第三方 Shader、插件和特殊外观件，副本策略通常比“同工程双版本”更稳妥。

#### 可选进阶：资源级复制工具

**工具**: アセットを依存関係を保ったまま一括コピーするやつ

| 字段 | 值 |
|------|-----|
| 作者 | narazaka |
| 链接 | https://narazaka.booth.pm/items/2181882 |
| 作用 | 为所有被模型引用的资源创建副本 |
| 价值 | 解决两套资源系统的依赖问题 |

**适用场景**:
- 想在同一项目保留两套资源
- 不想用项目副本策略
- 需要共享部分基础资源（如基础 Shader）

#### 官方相关页面

- **Setting up Unity for Creating Quest Content** - https://creators.vrchat.com/platforms/android/setting-up-unity-for-creating-quest-content
- **Cross-Platform Setup** - https://creators.vrchat.com/platforms/android/cross-platform-setup
- **官方未明确推荐独立工程还是同工程双版本**

---

## World 跨平台

### 内容回退
VRChat 尝试自动加载最兼容的内容版本：
- Android 玩家加载 Android 版本
- PC 玩家加载 PC 版本

### 最佳化差异

| 方面 | PC | Android |
|------|-----|---------|
| 着色器 | 无限制 | Mobile Shaders |
| 几何 | 高复杂度 | 低复杂度 |
| 纹理 | 高分辨率 | 1K 最大 |
| 光照 | 实时可用 | 必须烘焙 |

---

## iOS 支持

VRChat 正在实验 iOS 支持系统：
- World 可自动回退到 Android 版本
- Avatar 需要上传 iOS 专用版本以获得最佳效果

---

## 工具链

### VCC Package Manager
管理跨平台项目：
```
VRChat SDK → Manage Packages
```

### EasyQuestSwitch
自动调整内容的平台适配：
```
VCC → Add Package → EasyQuestSwitch
```

功能：
- 平台特定的材质切换
- 骨骼简化
- 纹理降级

---

## 测试清单

- [ ] PC 平台正常运行
- [ ] Android/Quest 平台正常运行
- [ ] Avatar 在所有平台正确显示
- [ ] World 性能在所有平台可接受
- [ ] 跨平台查看无异常

---

## 相关文档

- [[entities/platform/android-development]] - Android 开发
- [[entities/avatar/performance-rank]] - 性能标准
- （来源：Example Central，本地整理） - 示例中心
---

## 相关页面

[[easyquestswitch.md]] · [[mobile-ui-optimization.md]] · [[../avatar/optimization-guide.md]] · [[../world/performance-guide.md]] · [[android-development.md]]
