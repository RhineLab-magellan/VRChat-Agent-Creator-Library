---
title: Avatar 固定世界物品结构(World-Fixed Objects)
category: avatar
subcategory: accessories
knowledge_level: applied
status: active
source: yexcadocs §Bone Binding / 固定世界(2026-07-04)
source_type: community
version: 1.0
last_review: 2026-07-04
confidence: High
tags:
  - avatar
  - constraints
  - vrc-constraints
  - accessories
  - world-fixed
  - bone-binding
aliases:
  - 固定世界物品
  - "World-Fixed Objects"
  - "Bone Binding"
  - 物品固定世界
  - 5空物体结构
related:
  - vrc-constraints.md
  - accessories.md
  - playable-layers.md
  - expression-menu.md
type: entity
created: 2026-07-04
sources: yexcadocs §Bone Binding / 固定世界(2026-07-04)
updated: 2026-07-04
---

# Avatar 固定世界物品结构(World-Fixed Objects)

> 来源:yexcadocs §Bone Binding / 固定世界(P1)
> 适用场景:让 Avatar 上的物品(尾巴、翅膀、配饰等)在世界坐标下保持位置
> 本文档为社区实操经验,非 VRChat 官方文档。

---

## 1. 概述

**固定世界物品** = 让 Avatar 上的某些物品在玩家移动时**保持世界坐标位置**(而非跟随 Avatar 移动)。

### 典型应用

- 跟随玩家的尾巴
- 翅膀(玩家移动时不"穿透"身体)
- 长袍、披风(避免穿模)
- 拖在身后的配饰

### 核心原理

使用 **VRC Constraints**(Position/Rotation/Parent Constraint)创建动画驱动的世界坐标跟随系统:
- 通过空物体 + Constraint 组合实现"局部跟随"与"世界固定"的切换
- FX 层动画控制 Constraint 权重
- 物品本身作为 Constraint 目标

---

## 2. 核心结构:5 个空物体 + 3 种 Constraint

### 2.1 物体清单

| # | 名称 | 类型 | 挂载组件 | 关键参数 |
|---|------|------|---------|---------|
| 1 | **世界位置源** | 空物体 | 无 | 固定在原点 |
| 2 | **更新位置源** | 空物体 | 无 | 接收动画数据 |
| 3 | **世界位置** | 空物体 | Position/Rotation Constraint | 权重 = **0.5**,源权重 = **-1** |
| 4 | **更新位置** | 空物体 | Parent Constraint | 源 = 更新位置源 |
| 5 | **物品** | 空物体 | Parent Constraint | 源 = 更新位置,物品子集,位置 = (0, 0, 0) |

### 2.2 精确参数配置

#### 世界位置源(空物体 1)
```
Transform:
  Position: (0, 0, 0)
  Rotation: (0, 0, 0)
组件: 无
用途: 提供世界坐标的"基准源"
```

#### 更新位置源(空物体 2)
```
Transform:
  Position: 任意(可由动画驱动)
组件: 无
用途: 接收 FX 层动画的位置数据
```

#### 世界位置(空物体 3)
```
Transform:
  Position: (0, 0, 0)
组件:
  - Position Constraint
    - Sources: [世界位置源]
    - Weight: 0.5
  - Rotation Constraint
    - Sources: [世界位置源]
    - Weight: 0.5
    - Source Weight: -1
用途: 通过权重混合实现"动画跟随"或"世界固定"
```

#### 更新位置(空物体 4)
```
Transform:
  Position: (0, 0, 0)
组件:
  - Parent Constraint
    - Sources: [更新位置源]
    - Weight: 1
用途: 跟随"更新位置源"的动画数据
```

#### 物品(空物体 5)
```
Transform:
  Position: (0, 0, 0)  ← 必须是 (0, 0, 0)
组件:
  - Parent Constraint
    - Sources: [更新位置]
    - Weight: 1
子集: 模型物品(尾巴/翅膀/配饰等)
用途: 最终物品的容器
```

### 2.3 关键陷阱 ⚠️

- **物品位置必须为 (0, 0, 0)**,否则会出现偏移/旋转错误
- **5 个空物体顺序不能错**,否则 Constraint 无法正确解析
- **Source Weight = -1** 是关键参数,实现"反向跟随"

---

## 3. 动画配置(关闭 vs 打开)

### 3.1 关闭动画(物品在世界固定)

```
目标: 关闭"物品" → 打开"更新位置"

FX 层动画:
1. "物品" Parent Constraint Weight: 1 → 0
2. "更新位置" Parent Constraint Weight: 0 → 1

结果: 物品保持在世界坐标,不跟随 Avatar 移动
```

### 3.2 打开动画(物品跟随 Avatar)

```
目标: 打开"物品" → 关闭"更新位置"

FX 层动画:
1. "物品" Parent Constraint Weight: 0 → 1
2. "更新位置" Parent Constraint Weight: 1 → 0

结果: 物品跟随 Avatar 移动
```

### 3.3 录制步骤

```
1. 在 FX 层新建图层,权重 = 1
2. 创建两个状态:
   - 关闭状态(默认): 关闭物品的 Parent Constraint Weight(从 1 到 0)
   - 打开状态: 打开物品的 Parent Constraint Weight(从 0 到 1)
3. 录制两个动画:
   - 关闭动画: 录制到"关闭状态",设置 Constraint 权重
   - 打开动画: 录制到"打开状态",设置 Constraint 权重
4. 通过 Bool/Int 参数控制状态切换
5. 在 Expression Menu 添加 Toggle 控件
```

---

## 4. 物品骨骼处理(易错陷阱)

### 4.1 模型物品作为"物品"子集

- 模型的物品部分(尾巴/翅膀/配饰的 Mesh)必须作为"物品"空物体的**子集**
- 物品的 `Transform.Position` 必须为 `(0, 0, 0)`
- 否则会出现**偏移**或**旋转错误**

### 4.2 父子层级正确性

```
[正确] 物品 (空) → 物品子 Mesh
[错误] 物品子 Mesh 作为空物体的兄弟节点
```

### 4.3 调试技巧

- 选中"物品"空物体,Gizmo 应显示在正确位置
- 如果出现"物品飞走",检查 `Position = (0, 0, 0)` 设置
- 如果出现"旋转错误",检查 Rotation Constraint 的 Source Weight 是否为 -1

---

## 5. 与 VRC Constraints 的关系

本结构使用的 3 种 Constraint 均为 **VRC Constraints**(非 Unity Constraint):

| Constraint | 用途 |
|------------|------|
| **Position Constraint** | 限制物体位置(用于"世界位置") |
| **Rotation Constraint** | 限制物体旋转(用于"世界位置") |
| **Parent Constraint** | 多源混合父级约束(用于"更新位置"和"物品") |

详细 VRC Constraints 知识见 [[entities/avatar/vrc-constraints]]。

### 为什么不用 Unity Constraint

- VRC Constraints 对 VRChat 性能优化
- 避免 Unity Constraint 的 Sync Transform 开销
- VRC Constraints 在 Quest 端兼容性更好

---

## 6. 已知问题

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| 物品飞走 | 物品 Position ≠ (0, 0, 0) | 重置物品 Transform |
| 旋转错误 | Rotation Constraint 缺少 Source Weight = -1 | 重新配置 Constraint |
| 关闭/打开不响应 | FX 层动画未正确录制 | 重新录制动画 |
| Performance Rank 变差 | 5 个空物体 + 3 Constraint 增加复杂度 | 接受(结构必要) |

---

## 7. 适用场景评估

| 场景 | 推荐度 | 备注 |
|------|--------|------|
| 长尾巴(物理模拟) | 🟢 强烈 | 拖地效果显著 |
| 翅膀(玩家走动) | 🟢 强烈 | 避免穿透身体 |
| 长袍/披风 | 🟡 中 | 视具体效果 |
| 短配饰(耳环等) | 🔴 不推荐 | 复杂度高于收益 |
| 头发 | 🔴 不推荐 | 用 PhysBones 更合适 |

---

## 8. 相关文档

- [[entities/avatar/vrc-constraints]] — VRC Constraints 完整文档
- [[entities/avatar/accessories]] — VRChat 原生 Accessories 系统(不同概念)
- [[entities/avatar/playable-layers]] — FX 层动画驱动
- [[entities/avatar/expression-menu]] — 菜单 Toggle 控件

---

## 9. 文档元信息

- **源文档**:yexcadocs §Bone Binding / 固定世界
- **本地化时间**: 2026-07-04
- **知识等级**: Applied
- **可信度**: High(社区实操经验,与 VRC Constraints 文档交叉验证)
- **风险点**:yexcadocs 作者已声明 2023.11.27 后不再维护,部分细节需结合 VRC Constraints 最新文档验证

---

## 相关页面

[[vrc-constraints.md]] · [[accessories.md]] · [[playable-layers.md]] · [[expression-menu.md]]
