---
title: "VRChat Constraints 知识库"
category: avatar
knowledge_level: applied
status: active
source: "creators.vrchat.com/common-components/constraints/ + creators.vrchat.com/common-components/constraints/constraints-api/ + creators.vrchat.com/releases/release-3-10-6/ (复核 2026-11-10)"
source_type: community
version: 1.3
last_review: 2026-11-10
confidence: High
tags:
  - avatar
  - constraint
  - udonsharp
aliases:
  - "VRC Constraints"
  - 约束系统
  - "Avatar Constraints"
  - "VRC 约束"
related:
  - animator-system.md
  - modular-avatar.md
  - teaching-methodology.md
  - tex-trans-tool.md
  - accessories.md
type: entity
created: 2026-07-04
sources: "creators.vrchat.com/common-components/constraints/ | creators.vrchat.com/common-components/constraints/constraints-api/ | creators.vrchat.com/releases/release-3-10-6/ (复核 2026-11-10)"
updated: 2026-11-10
---
# VRChat Constraints 知识库


---

## 目录

1. [概述](#1-概述)
2. [约束类型](#2-约束类型)
3. [高级设置](#3-高级设置)
4. [性能](#4-性能)
5. [使用技巧](#5-使用技巧)
6. [Constraints API](#6-constraints-api)

---

## 1. 概述

VRChat 提供自定义约束系统，允许 Avatar 或 World 中的 Transform 相对于其他 Transform 进行移动、旋转和缩放。

### 与 Unity Constraints 的区别

| 方面 | VRChat Constraints | Unity Constraints |
|------|-------------------|-------------------|
| Avatar | ✅ 推荐使用 | 自动转换 |
| World | 均可 | 均可 |
| 性能 | 更好 | 可能性能问题 |
| 功能 | 更多可选功能 | 基础功能 |

**重要**: Avatar 必须使用 VRChat Constraints。如果包含 Unity Constraints，上传时自动转换为 VRChat Constraints。

### 版本信息 ⭐NEW 2026-07-04

> 来源: VRChat 官方 packages 仓库 release 历史
> 链接: https://github.com/vrchat/packages/releases
> 置信度: High (官方资料)

VRChat Constraints 的版本引入时序(按时间顺序):

| SDK 版本 | 类型 | 官方说明 |
|---------|------|---------|
| **3.6.2-constraints.1** | open-beta | "This is an open-beta release for VRChat Constraints." |
| **3.6.2-constraints.2** | open-beta | 同上 |
| **3.6.2-constraints.3** | open-beta | 同上 |
| **3.6.2-constraints.4** | open-beta | 同上 |
| **3.7.0** | **live release** | "This is a live release!"(2024-08-16) |
| **3.7.0-beta.1 / beta.2** | open-beta | 3.7.0 正式版前的 beta 周期 |

**关键里程碑**:
- **2024-08-16**: SDK 3.7.0 正式发布,VRC Constraints 脱离 beta
- **同 release 还修复**: "constraint sources losing their references when selecting multiple VRChat constraints and editing their sources lists at the same time"

**对 Avatar 开发者的影响**:
- **3.7.0 之前**: Avatar 只能使用 Unity Constraints(SDK 上传时自动转换为 VRC Constraints)
- **3.7.0 起**: 创作者可直接使用 VRC Constraints,无需依赖自动转换,功能更可控
- **3.10.0** (2025-10-24): VRC Constraints 进一步扩展到 World(从 Avatar 扩展为通用)
- **3.10.6** (2026-10-30): 新增 **Smoothing Factor**（平滑跟随）与整套**源管理方法**（AddSource/RemoveSource/GetSource/SetSource/GetSources/SetSources/ClearSources/SourceCount）；**Udon 中不再推荐直接改 `Sources` 列表**；SDK 工程的约束在 Play 模式下现在会响应通过 C# 修改的 Target Transform（此前不响应）；编辑器内 Activate/Zero 操作可撤销

> 📖 详细 API 与约束类型见 §2-§6

### 命名空间

```
VRC.SDK3.Dynamics.Constraint.Components
VRC.Dynamics
```

---

## 2. 约束类型

### 2.1 VRCAimConstraint（目标约束）

旋转目标 Transform，使其朝向源。可自定义前进方向。

**参数**:
- `Is Active` - 控制约束是否评估
- `Weight` - 整体权重 (0-1)
- `Aim Vector` - 控制朝向源的轴
- `Up Vector` - 控制向上轴
- `World Up Type`:
  - Scene Up (Y轴)
  - Object Up (指定对象)
  - Object Up Rotation (局部空间)
  - Vector (世界空间向量)
  - None (无)

**Constraint Settings**:
- `Rotation At Rest` - 权重为0时的旋转
- `Rotation Offset` - 结果偏移
- `Lock` - 锁定 At Rest 和 Offset
- `Freeze Rotation Axes` - 冻结特定轴

### 2.2 VRCLookAtConstraint（注视约束）

简化的 Aim Constraint，使目标 Transform 的正Z轴朝向源。

**参数**:
- `Is Active` - 控制约束是否评估
- `Weight` - 整体权重 (0-1)
- `Use Up Object` - 是否使用单独的对象控制翻滚
- `Roll` - 绕Z轴的角度（无Up Object时）
- `World Up Object` - 翻滚目标对象

### 2.3 VRCParentConstraint（父约束）

移动和旋转目标 Transform，如同其是源的子对象。

**参数**:
- `Is Active` - 控制约束是否评估
- `Weight` - 整体权重 (0-1)

**Constraint Settings**:
- `Position/Rotation At Rest` - 静止位置/旋转
- `Position/Rotation Offset` - 偏移量
- `Lock` - 锁定设置
- `Freeze Position/Rotation Axes` - 冻结轴

**注意**: 每个源的偏移是独立设置的。

### 2.4 VRCPositionConstraint（位置约束）

改变目标 Transform 的位置以匹配源的位置。

**参数**:
- `Is Active` - 控制约束是否评估
- `Weight` - 整体权重 (0-1)

**Constraint Settings**:
- `Position At Rest` - 静止位置
- `Position Offset` - 位置偏移
- `Lock` - 锁定设置
- `Freeze Position Axes` - 冻结轴

### 2.5 VRCRotationConstraint（旋转约束）

改变目标 Transform 的旋转以匹配源的旋转。

**参数**:
- `Is Active` - 控制约束是否评估
- `Weight` - 整体权重 (0-1)

**Constraint Settings**:
- `Rotation At Rest` - 静止旋转
- `Rotation Offset` - 旋转偏移
- `Lock` - 锁定设置
- `Freeze Rotation Axes` - 冻结轴

### 2.6 VRCScaleConstraint（缩放约束）

改变目标 Transform 的缩放以匹配源的缩放。

**参数**:
- `Is Active` - 控制约束是否评估
- `Weight` - 整体权重 (0-1)

**Constraint Settings**:
- `Scale At Rest` - 静止缩放
- `Scale Offset` - 缩放偏移
- `Lock` - 锁定设置
- `Freeze Scale Axes` - 冻结轴

---

## 3. 高级设置

### Target Transform

更改约束目标的 Transform。默认为空，约束应用于附加到的 Transform。

**限制**: 不能通过动画更改此设置。

### Solve In Local Space

- **关闭**: 世界空间求解（匹配源的**世界**位置/旋转/缩放）
- **开启**: 局部空间求解（匹配源的**局部**位置/旋转/缩放）

**用途**: 假肢体、跟随骨骼链等。

### Freeze To World

启用后，约束忽略所有源，锁定目标 Transform 在世界空间。

**用途**: 在世界中放下物体，物体不会跟随 Avatar 移动。

**限制**:
- 必须冻结所有轴才能完全停止
- 与禁用约束组件不同：
  - 禁用约束：Transform 停止在局部空间移动，但仍跟随父对象
  - Freeze To World：主动移动 Transform 防止在世界空间移动

### Rebake Offsets When Unfrozen

启用后，解冻时重新计算相对于源的偏移（而非保持原始偏移）。

> 该值本身切换时**没有即时效果**——它只决定 `Freeze To World` 从 true 变 false 那一刻的行为。

### Smoothing Factor（SDK 3.10.6 新增）

| 项 | 说明 |
|----|------|
| **默认值** | 0（无平滑，瞬时贴合目标） |
| **取值** | 大于 0 即启用平滑；常用区间 **0 ~ 1**，需要"极慢跟随"可以大于 1 |
| **数学定义** | 每帧按 `result = lerp(current, target, 1 - exp(-deltaTime / smoothingFactor))` 逼近，即 **smoothingFactor 秒约走完 63%** |
| **编辑器行为** | 在 SDK 工程里**只有进入 Play 模式才生效**（编辑期不模拟平滑） |
| **典型用途** | Aim Constraint 指向目标时带滞后转身，避免瞬移抽帧 |
| **性能** | 平滑本身不增加约束计数/深度，但会改变每帧的求解输入 |

来源：[FACT] [VRChat 官方 Constraints 文档](https://creators.vrchat.com/common-components/constraints/) · [Constraints API](https://creators.vrchat.com/common-components/constraints/constraints-api/) · [Release 3.10.6](https://creators.vrchat.com/releases/release-3-10-6/)

---

## 4. 性能

### Avatar 性能分类

| 分类 | 说明 |
|------|------|
| **Constraint Count** | 约束总数（含禁用），VRChat + Unity |
| **Constraint Depth** | 最长约束链的深度 |

### Constraint Depth 说明

约束链按特定顺序逐个评估，从链底部到顶部。

**示例**: 3个旋转约束链 = 深度为3

```
约束1 (臀部) → 约束2 (膝盖) → 约束3 (脚)
     深度 1        深度 2        深度 3
```

**优化建议**:
- 减少最长链的长度
- 关注最长链，非总链数
- 多条深度为3的链 = 总体深度仍为3（如果无更长链）

### Unity Constraints 影响

- Unity Constraints 可能导致深度被**高估**
- 建议使用 SDK Control Panel 的 Auto Fix 转换所有 Unity Constraints

---

## 5. 使用技巧

### 故障排除

1. 检查 `Is Active` 是否启用
2. 检查组件本身是否启用
3. 检查 `Lock` 是否启用

### 限制

1. **Target Transform 不能被动画或 Udon 更改**（性能缓存）
2. **不要动画更改约束引用的 Transform**（性能问题）
   - 动画 Transform 的位置/旋转/缩放是可以的
3. **Avatar 上不能动画单个源的变换引用**
   - 替代方案：设置多个源，动画它们的权重

### 按钮功能

| 按钮 | 功能 |
|------|------|
| **Activate** | 保存当前偏移，激活并锁定约束 |
| **Zero out** | 重置偏移为默认值，激活并锁定约束 |

---

## 6. Constraints API

### 命名空间（VRCConstraintSource / VRCConstraintSourceKeyableList）

```csharp
using VRC.SDK3.Dynamics.Constraint.Components;   // VRCAimConstraint / VRCLookAtConstraint / VRCParentConstraint / VRCPositionConstraint / VRCRotationConstraint / VRCScaleConstraint
using VRC.Dynamics;                            // VRCConstraintSource / VRCConstraintSourceKeyableList / VRCConstraintBase.WorldUpType
```

> 与 §1 那个「命名空间」小节的区分：§1 只列了约束组件本身的命名空间；这里额外标出 **源类型与 `WorldUpType` 枚举** 所在的 `VRC.Dynamics`——写工具代码时最常混的就是这两个命名空间。

### 通用属性和方法

#### ApplyConfigurationChanges()

修改约束后**必须调用此方法**，否则更改可能不生效。

```csharp
constraint.PositionOffset = new Vector3(1, 0, 0);
constraint.ApplyConfigurationChanges();
```

#### IsActive

- `bool`: 约束是否正在评估

#### GlobalWeight

- `float`: 全局权重

#### Locked

- `bool`: 约束是否锁定（播放模式始终锁定）

#### Sources（⚠️ 3.10.6 起不再推荐直接操作）

- `VRCConstraintSourceKeyableList`: 源列表

> 🔴 **SDK 3.10.6 变更（Refresh）**：官方新增了一组 `AddSource` / `RemoveSource` / `GetSource` / `SetSource` / `GetSources` / `SetSources` / `ClearSources` / `SourceCount` 方法，并**明确建议在 Udon 中用这些方法替代直接改 `Sources` 列表**——直接改列表被证实有可靠性问题（列表是 struct 类型，Udon 应用修改时可能不生效）。`Sources` 列表**仍然对 Udon 白名单开放**只是为了向后兼容。
>
> 例外：`Sources` 列表在**动画剪辑**和非 Udon 的 C# 工具里使用是安全的。

**注意**: Avatar 上只能动画前16个源元素（引擎限制，与上面的 API 变更无关）。

```csharp
// 修改源权重
for (int i = 0; i < constraint.Sources.Count; i++)
{
    VRCConstraintSource source = constraint.Sources[i];
    source.Weight = UnityEngine.Random.value;
    constraint.Sources[i] = source;
}

// 添加源
VRCConstraintSource source = new VRCConstraintSource(transform, 1.0f);
constraint.Sources.Add(source);

// 移除源
constraint.Sources.Remove(source);
constraint.Sources.RemoveAt(i);
```

#### TargetTransform

- `Transform`: 受约束影响的 Transform

#### SolveInLocalSpace

- `bool`: 是否在局部空间求解

#### FreezeToWorld

- `bool`: 是否冻结在世界空间

#### RebakeOffsetsWhenUnfrozen

- `bool`: 解冻时是否重新计算偏移

#### ActivateConstraint()

激活并锁定约束，保持当前偏移。

#### ZeroConstraint()

激活并锁定约束，重置偏移。

### 源管理方法（SDK 3.10.6 新增，推荐替代直接改 `Sources`）

| 成员 | 签名 / 返回 | 说明 |
|------|-------------|------|
| `SourceCount` | `int`（只读） | 当前源数量；遍历前用它做边界检查 |
| `AddSource(VRCConstraintSource source)` | `void` | 追加到列表**末尾** |
| `RemoveSource(int index)` | `void` | 按下标移除 |
| `RemoveSource(VRCConstraintSource source)` | `bool` | 按源移除；返回值表示是否找到并移除 |
| `GetSource(int index)` | `VRCConstraintSource` | 越界时**打错误日志并返回空源**（不抛异常）——务必先做边界检查 |
| `SetSource(int index, VRCConstraintSource source)` | `void` | 覆盖指定下标的源；越界行为同上 |
| `GetSources(VRCConstraintSource[] sources)` | `int` | 把全部源拷进你给的数组，返回**实际拷入数量**；数组不够大时只拷能放下的部分 |
| `SetSources(VRCConstraintSource[] sources)` | `void` | 用给定数组**完全替换**源列表；传 null 或空数组等价于清空 |
| `ClearSources()` | `void` | 清空源列表 |

```csharp
// 官方推荐写法：随机化所有源权重（替代旧的 Sources[i] 读改写）
for (int i = 0; i < constraint.SourceCount; i++)
{
    VRCConstraintSource source = constraint.GetSource(i);
    source.Weight = UnityEngine.Random.value;
    constraint.SetSource(i, source);
}

// 批量读取
VRCConstraintSource[] buf = new VRCConstraintSource[constraint.SourceCount];
int n = constraint.GetSources(buf);
```

> ⚠️ **不要把这套 API 写进需要兼容 3.10.5 及更早 SDK 的工程**——3.10.5 及之前的 SDK 没有这些方法，会编译失败。需要跨版本时用 Assembly Version Defines（3.10.5 起支持，写法如 `#if SDK_310_OR_NEWER`）。

> 💡 与官方一致的调用纪律：一次改多个属性时，**最后只调用一次** `ApplyConfigurationChanges()`，减少性能开销。

来源：[FACT] [Constraints API 官方页](https://creators.vrchat.com/common-components/constraints/constraints-api/) · [Release 3.10.6](https://creators.vrchat.com/releases/release-3-10-6/)

---

### At-Rest 和 Offset 值
每个约束类型有不同的 At-Rest 和 Offset 值：

| 约束类型 | At-Rest | Offset |
|----------|---------|--------|
| Position | Vector3 | Vector3 |
| Rotation | Vector3（Euler 角）⚠️ | Vector3（Euler 角）⚠️ |
| Scale | Vector3 | Vector3 |
| Parent | Vector3 + Quaternion | Vector3 + Quaternion |
| Aim | Vector3（Euler 角）⚠️ | Vector3（Euler 角）⚠️ |
| Look At | Vector3（Euler 角）⚠️ | Vector3（Euler 角）⚠️ |

> ⚠️ **类型仍待核实（2026-11-10 复核后未变）**: 官方 Constraints API 页只写 "a set of Euler angles"，**未给出 C# 类型签名**；本页下文的 `Quaternion RotationAtRest` 写法来自社区整理，与 "Euler angles" 的措辞存在张力。接入真实项目前请在 SDK Inspector 或反射确认，不要直接照抄下文的 `Quaternion` 声明。
>
> 另：**Parent Constraint 的 offset 不是全局值，而是"每个源一份"**——需要通过 `Sources` 里每个 `VRCConstraintSource` 的 `ParentPositionOffset` / `ParentRotationOffset` 修改。下文把 Parent 的 offset 写在约束本体上，属历史写法，已标注。

### 约束类型特定 API

#### PositionConstraint

```csharp
Vector3 PositionAtRest { get; set; }
Vector3 PositionOffset { get; set; }
bool GetPositionAxisIsFrozen(int axis);
void SetPositionAxisIsFrozen(int axis, bool value);
```

#### RotationConstraint

```csharp
Quaternion RotationAtRest { get; set; }
Quaternion RotationOffset { get; set; }
bool GetRotationAxisIsFrozen(int axis);
void SetRotationAxisIsFrozen(int axis, bool value);
```

#### ScaleConstraint

```csharp
Vector3 ScaleAtRest { get; set; }
Vector3 ScaleOffset { get; set; }
bool GetScaleAxisIsFrozen(int axis);
void SetScaleAxisIsFrozen(int axis, bool value);
```

#### ParentConstraint

```csharp
Vector3 PositionAtRest { get; set; }
Quaternion RotationAtRest { get; set; }
Vector3 PositionOffset { get; set; }
Quaternion RotationOffset { get; set; }
bool GetPositionAxisIsFrozen(int axis);
void SetPositionAxisIsFrozen(int axis, bool value);
bool GetRotationAxisIsFrozen(int axis);
void SetRotationAxisIsFrozen(int axis, bool value);
```

#### AimConstraint

```csharp
Quaternion RotationAtRest { get; set; }
Quaternion RotationOffset { get; set; }
bool GetRotationAxisIsFrozen(int axis);
void SetRotationAxisIsFrozen(int axis, bool value);
```

#### LookAtConstraint

```csharp
Quaternion RotationAtRest { get; set; }
Quaternion RotationOffset { get; set; }
bool GetRotationAxisIsFrozen(int axis);
void SetRotationAxisIsFrozen(int axis, bool value);
```

---

### 新增 AIM / Look At 专用属性（官方 API 页）

本节记录原页缺失的 Aim / Look At 对齐属性，便于按官方签名编写工具代码。

#### Aim Constraint 对齐属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `AimAxis` | `Vector3` | 应当**指向源**的轴，等价于定义"哪个方向算前方" |
| `UpAxis` | `Vector3` | 约束视为"上"的轴，会向 `WorldUp` 指定的上方向对齐 |
| `WorldUpTransform` | `Transform` | 仅在特定 `WorldUp` 取值下参与计算 |
| `WorldUpVector` | `Vector3` | 仅在特定 `WorldUp` 取值下参与计算 |
| `WorldUp` | `VRCConstraintBase.WorldUpType`（枚举，命名空间 `VRC.Dynamics`） | 见下表 |

| `WorldUp` 取值 | 含义 |
|------|------|
| `WorldUpType.SceneUp` | 以场景 +Y 为上（`Vector3.up`） |
| `WorldUpType.ObjectUp` | 从目标 Transform 指向 `WorldUpTransform` 的向量为上 |
| `WorldUpType.ObjectRotationUp` | 以 `WorldUpVector` 在 `WorldUpTransform` 局部空间中的方向为上 |
| `WorldUpType.Vector` | 以世界空间中的 `WorldUpVector` 为上 |
| `WorldUpType.None` | 不定义上方向 |

#### Look At Constraint 对齐属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `Roll` | `float` | 绕约束自身 Z 轴的旋转角度（度）；**仅当 `UseUpTransform == false` 时生效** |
| `WorldUpTransform` | `Transform` | 朝其滚转的目标；**仅当 `UseUpTransform == true` 时生效** |
| `UseUpTransform` | `bool` | `false` 时用 `Roll` 决定倾角；`true` 时尝试让 +Y 指向 `WorldUpTransform` |

> 注：Look At 本质上是 Aim 的简化版（固定 +Z 朝向源）。

来源：[FACT] [Constraints API 官方页](https://creators.vrchat.com/common-components/constraints/constraints-api/)

---

## 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| 1.0 | 2026-06-05 | 初始创建 |
| 1.3 | 2026-11-10 | 补 SDK 3.10.6：Smoothing Factor 高级设置与 API、源管理方法组（AddSource/RemoveSource/GetSource/SetSource/GetSources/SetSources/ClearSources/SourceCount）、3.10.6 起"Udon 中不再推荐直接改 Sources"、Aim/LookAt 对齐属性补全、Parent offset 属于源而非约束本体的更正；来源页换为官方 Constraints / Constraints API / Release 3.10.6；§6 重复的「### 命名空间」标题改为「命名空间（VRCConstraintSource / VRCConstraintSourceKeyableList）」并标注与 §1 的分工 |

## 来源

- creators.vrchat.com/common-components/constraints/
- creators.vrchat.com/common-components/constraints/vrc-*-constraint
- creators.vrchat.com/common-components/constraints/constraints-api
---

## 相关页面

[[animator-system.md]] · [[modular-avatar.md]] · [[teaching-methodology.md]] · [[tex-trans-tool.md]] · [[accessories.md]]
