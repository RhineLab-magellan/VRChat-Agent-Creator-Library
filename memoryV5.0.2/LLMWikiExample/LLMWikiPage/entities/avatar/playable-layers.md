---
title: "VRChat Playable Layers 知识库"
category: avatar
knowledge_level: applied
status: active
source: "creators.vrchat.com/avatars/playable-layers/"
source_type: community
version: 1.1
last_review: 2026-07-04
confidence: High
tags:
  - avatar
  - animator
  - playable-layer
  - vrc-school
aliases:
  - "Playable Layers"
  - 播放层
  - "Avatar 播放层"
  - "FX Layer"
related:
  - animator-system.md
  - avatar-modding-guide.md
  - avatar-parameter-staged-sync.md
  - lac-avatar-compressor.md
  - ma-component-cards.md
type: entity
created: 2026-07-04
sources: "creators.vrchat.com/avatars/playable-layers/"
updated: 2026-07-04
---
# VRChat Playable Layers 知识库


---

## 目录

1. [概述](#1-概述)
2. [Humanoid vs Generic](#2-humanoid-vs-generic)
3. [层级顺序](#3-层级顺序)
4. [各层详解](#4-各层详解)
5. [附加姿势](#5-附加姿势)
6. [Avatar Mask 规则](#6-avatar-mask-规则)
7. [VRCFury 集成](#7-vrcfury-集成)
8. [Avatar Rigs 详细配置](#8-avatar-rigs-详细配置)
9. [Culling Mode 行为](#9-culling-mode-行为)

---

## 1. 概述

Playable Layers 允许将 Avatar 功能分离到独立的动画层中，如跑步、跳跃、点赞、微笑、摇尾巴等。

### 需要的基础知识

- Unity Animator Controller
- Avatar Mask
- Blend Trees

---

## 1.1 Avatar 3.0 基础概念（VRC School 补充）⭐

> 来源：VRC School + VRChat 官方文档
> 验证状态：2026-07-04

### 核心定义

**Avatar 3.0** = 5 个 Playable Layers + Expression Menu + Expression Parameters 构成的**参数驱动系统**

### Avatar 3.0 vs 2.0 核心区别

| 维度 | Avatar 2.0 | Avatar 3.0 |
|------|-----------|-----------|
| 控制系统 | Gesture Override | Animator Parameters + Gesture |
| 动画层 | 单层 | 5 个 Playable Layers |
| 菜单控制 | 有限 | 完整 Expression Menu |
| 灵活性 | 低 | 高（可自定所有层）|
| 复杂度 | 低 | 高（需理解状态机）|
| Gesture Toggle 行为 | 控制 Gesture Override | 控制 Gesture Animator Parameters |

### Generic Avatar 的 3 个 Playable Layers

- Base
- Action
- FX

> 其他层（Additive、Gesture）是 Humanoid 专用

### 学习路径建议

- **初学者**：先看 VRChat Avatar 3.0 入门
- **深入者**：再学 Unity 动画系统（Animation）
- **高级者**：研究 Playable Layers + Expression Menu + 状态机设计

### 关键概念

| 概念 | 含义 |
|------|------|
| **Playable Layer** | 独立的动画层 |
| **Avatar Mask** | 控制层可影响的部位 |
| **Blend Tree** | 参数驱动的混合动画 |
| **Write Defaults** | 动画默认值持久化策略 |
| **Direct Blend Tree** | 优化多 Toggle 合并的混合树 |

### 与 Unity 动画系统的关系

- Avatar 3.0 基于 Unity 动画系统
- VRChat 不使用 Animator 的 Controller 字段
- 改为使用 VRC Avatar Descriptor 配置 5 个 Playable Layers
- 子 GameObject 的 Animator 仍使用自己的 Controller

### Generic Avatar 限制

- ⚠️ 不能使用 Expression Parameters 和 State Behaviours 等 AV3 功能
- ⚠️ 除非遵循特定的 FBX 导入要求
- 如不需要 AV3 功能 → 可在 root Animator 中直接使用动画 Controller
---

## 2. Humanoid vs Generic

### Humanoid Avatar（人形Avatar）

5个 Playable Layers:
| 层级 | 说明 |
|------|------|
| **Base** | 基础运动层 |
| **Additive** | 叠加层 |
| **Gesture** | 手势层 |
| **Action** | 动作层 |
| **FX** | 特效层 |

### Generic Avatar（通用Avatar）

3个 Playable Layers:
| 层级 | 说明 |
|------|------|
| **Base** | 基础运动层 |
| **Action** | 动作层 |
| **FX** | 特效层 |

---

## 3. 层级顺序

层级按顺序应用：

```
Base → Additive → Gesture → Action → FX
```

**优先级**: FX 最高，Base 最低

**示例**: 如果 Additive 动画骨骼（权重1.0），然后 Action 也动画同一骨骼（权重1.0），**Action 优先**。

---

## 4. 各层详解

### 4.1 Base Layer

**用途**: 始终播放的功能，响应移动（locomotion），如行走、跑步、跳跃、跌落、蹲下等。

**要求**:
- ✅ 只影响 Transform
- ✅ 使用 Avatar Mask 确保只影响适当的 Transform

**注意**: 在此层添加内容需要重新定义 locomotion 动画状态，非常复杂。

### 4.2 Additive Layer

**用途**: 在 Base 层已有的基础上"添加"动画，如呼吸动画。

**重要**:
- ⚠️ **仅用于人形骨骼**
- ❌ 非人形骨骼（尾巴、耳朵等）应使用 **Gesture** 层

**特点**:
- 始终设置为 "Additive" 混合模式
- 如果骨骼在 locomotion 期间移动，Additive 动画会叠加在上面

**注意**: 第一个 Additive 层的 Avatar Mask 会被忽略（内部屏蔽用途）。

### 4.3 Gesture Layer

**用途**: 需要在播放底层动画的同时作用于特定身体部位的动画。

**功能**:
- 手势触发（Hand OR Expression Menu）
- 闲置动画（尾巴摇摆、翅膀拍动、耳朵移动等）
- 非人形骨骼动画

**要求**:
- ✅ 只影响 Transform
- ✅ 使用 Avatar Mask 确保只影响目标部位

### 4.4 Action Layer

**用途**: 完全覆盖其他层的骨骼动画，类似 AV2 Emotes。

**重要**:
- ⚠️ **默认混合到零**
- ⚠️ 在 Action 层做任何事之前，必须使用 **Playable Layer Control** 状态行为将层混合上来
- ⚠️ 完成时必须混合回零

### 4.5 FX Layer

**用途**: 处理所有**非 Transform** 的动画。

**特点**:
- ✅ 全部复制到镜像克隆（与其他层不同）
- ✅ 可以启用/禁用 GameObject
- ✅ 可以修改组件、材质、Shader 属性
- ✅ 可以动画粒子系统

**包含**:
- 启用/禁用 GameObject
- 组件修改
- 材质切换
- Shader 动画
- 粒子系统动画
- Blend Shape 动画

**注意**:
- ⚠️ 不建议在 FX 层动画 Transform
- 第一层 FX 默认 mask 会禁用所有人形肌肉但启用所有 GameObject 动画

---

## 5. 附加姿势

### 5.1 T-Pose

自定义 T-Pose 用于：
- 确定 Avatar 测量（尤其是视点位置）
- 手腕对齐/扭曲
- 翼展（影响 IPD）

**注意**:
- T-Pose 中的关节弯曲会影响 Avatar 比例
- 肘部弯曲可能影响多个功能

### 5.2 IK Pose

IK Pose 用于确定主要关节弯曲方向。

**膝盖弯曲规则**:
- 脚向外旋转 → 膝盖向内弯曲
- 脚向内旋转 → 膝盖向外弯曲

### 5.3 Sitting Pose

Sitting Pose 用于：
- 坐姿时视点校准
- 坐姿动画和闲置动画

**注意**: 自定义可能需要大量调整，建议使用过渡状态。

---

## 6. Avatar Mask 规则

### 6.1 冲突规则

如果 Gesture 中有非肌肉动画（如 Transform），这些 Transform **必须**在 FX mask 中禁用。

### 6.2 示例

**场景**:
- Avatar 有尾巴（非人形骨骼链）
- Gesture 动画层有尾巴的特殊 mask
- 另一个 Gesture 动画层有"所有部位"mask

**解决方案**:
1. 在第一层 FX 创建自定义 mask
2. 禁用所有肌肉（人形图全部红色）
3. 禁用尾巴中的所有骨骼
4. 启用任何 FX 动画的 Transform（如用于 blend shape 或材质的蒙皮网格）

### 6.3 常见问题

**问题**: GameObject 同时有 Gesture 中的动画 Transform 和 FX 中的效果组件

**解决方案**:
- 创建子 GameObject 放置静态网格或效果
- 不动画子对象的 Transform，只动画父对象

---

## 7. VRCFury 集成

### VRCFury Components（Avatar 自动化）

| 组件 | 用途 |
|------|------|
| Armature Link | 服装/道具附加到骨骼 |
| Full Controller | 合并完整 animator/expression menu/parameters |
| Gestures | 基于手势创建动画 |
| Global Collider | 全局 PhysBone collider 和 Contact sender |
| Toggle | 菜单切换、按钮、滑块 |
| Other | 高级功能 |

### VRCFury 中的 Constraint 使用

**Cross-Eye Fix**:
- 使用 rotation constraints 消除眼睛滚动
- 自动修复 VRChat 引入的眼睛骨骼滚动问题

### 重要提示

⚠️ **不要在多个 Playable Layers 中使用相同的 controller**

这可能导致某些设置正常工作，但随着 Avatar 功能扩展会造成重大问题。

---

## 8. Avatar Animator = 参数驱动系统

### 核心定义

```
Avatar Animator = VRChat 对 Unity Animator 的特殊封装
驱动方式 = VRChat Parameters 驱动状态机
```

### 工作流

```
Expression Menu
        ↓
Expression Parameter（参数）
        ↓
Playable Layer（Animator Controller）
        ↓
State Machine
        ↓
Animation Clip
        ↓
Avatar
```

**关键特点**：
- 控制来源：VRChat Parameters（手势、表情、菜单项）
- 无需 Udon 脚本，参数自动驱动
- VRChat 自动同步部分参数
- 核心用途：表情、菜单、手势、Toggle

### 内置驱动参数

```csharp
VelocityMagnitude  → 速度
GestureLeft       → 左手势
GestureRight      → 右手势
IsLocal           → 本地/远程
// 以及用户定义的 Expression Parameters
```

### Playable Layers 中的驱动方式

| Layer | 驱动源 | 说明 |
|-------|--------|------|
| Base | 移动输入 | Locomotion 状态机 |
| Additive | 持续 | 呼吸等叠加动画 |
| Gesture | 手势触发 | 手势 + Expression Menu |
| Action | 动作触发 | Emotes 完全覆盖 |
| FX | 菜单参数 | Toggle、Button、Slider |

### Avatar vs World 核心区别

| 方面 | Avatar Animator | World Animator |
|------|-----------------|----------------|
| 运行主体 | 玩家 Avatar | 场景物体 |
| 控制来源 | VRChat Parameters | Udon / Script |
| 脚本控制 | ❌ 不支持 | ✅ 支持 |
| 网络同步 | VRChat 自动同步 | 开发者自行同步 |
| 开发模式 | **参数驱动** | **逻辑驱动** |

## SDK 示例文件

位置: `Packages/com.vrchat.avatars/Samples/AV3 Demo Assets/Animation/Controllers`

| Playable Layer | 文件名 |
|----------------|--------|
| Base | `vrc_AvatarV3LocomotionLayer` |
| Additive | `vrc_AvatarV3IdleLayer` |
| FX | `vrc_AvatarV3FaceLayer` |
| Action | `vrc_AvatarV3ActionLayer` |
| Gesture | `vrc_AvatarV3HandsLayer` |

---

## 8. Avatar Rigs 详细配置（VRC School 补充）⭐

> 来源：VRC School + VRChat 官方文档
> 重要性：Humanoid 是 VRChat Avatar 的前提
> 验证状态：2026-07-04

### 8.1 四种骨架类型

| 类型 | 说明 | 用途 | VRChat 必须？ |
|------|------|------|-------------|
| **Humanoid（人形）** | 最流行，VRChat 默认动画依赖此类型。包含人形骨骼列表，映射到 Transform | **VRChat Avatar 必须** | ✅ |
| **Generic（通用）** | 类似于 Humanoid，但不含所有人形骨骼/肌肉 | 仅用于创建 Animator 层遮罩 | ⚠️ 部分支持 |
| **Legacy（旧版）** | 不创建 Avatar 骨架，导入骨骼/动画/混合形状 | 旧 Unity 动画系统（**已弃用**）| ❌ |
| **None（无）** | 不创建 Avatar 骨架，不含骨骼/动画/混合形状 | 性能最佳选择（用于非动画网格）| ❌ |

### 8.2 Humanoid 关键配置

#### Avatar Definition
- **Copy From Other Avatar**：拖动其他虚拟形象复制骨骼定义
- **Create From This Model**：根据导入是否成功显示 ×/✓，提供 Configure 按钮

#### Skin Weights
- **Standard (4 bones)**：每个顶点最多受 4 个骨骼组影响
- **Custom**：可编辑每顶点最大骨骼数量 + 最小骨骼权重

#### Optimize Game Objects
- 启用后：所有未在 `Extra Transforms To Expose` 中启用的骨骼从 FBX 新实例中移除
- 改用内部表示以**提高性能**
- 配置菜单可选择性暴露骨骼

### 8.3 Humanoid 配置界面工作流

点击 **Configure** 后：
- 当前场景关闭（完成后重开）
- 显示人形骨架配置界面
- 顶部有 **Mapping** 和 **Muscles & Settings** 两个部分

#### Mapping（映射）
- 4 个子标题：Body / Head / Left Hand / Right Hand
- 底部下拉菜单：
  - **Mapping**: Clear / Automap / Save / Load
  - **Pose**: Reset / Sample Bind-Pose / **Enforce T Pose**

> ⚠️ 点击 Apply 或 Done 时虚拟形象需处于 **T 姿势**

#### Muscles & Settings（肌肉与设置）
1. **第一部分**：拖动滑块预览完整肌肉组运动（Reset All 重置）
2. **第二部分**：预览和编辑单个肌肉范围
   - 范围表示骨骼可旋转角度
   - 例：范围 -180 到 180，肌肉值 0.75 → 右旋转 90 度
3. **第三部分**：整体设置
   - **Twist settings**：扭转肌肉对邻近骨骼影响
   - **Stretch settings**：重定向到其他模型时骨骼拉伸
   - **Feet spacing**：脚间默认距离
   - **Translation DoF**：允许人形骨架旋转 + 移动骨骼
   - **Muscles → Reset**：重置第二选项卡的肌肉范围

按钮：**Revert**（撤销退出）/ **Apply**（应用停留）/ **Done**（应用退出）

### 8.4 Generic 骨架

- Avatar Definition / Skin Weights / Optimize Game Objects 下拉菜单功能同 Humanoid
- **Root Node**：设置动画播放的根节点

### 8.5 Legacy 骨架

- 同样有 Optimize Game Objects
- 额外的下拉菜单：选择如何导入动画
- **仅适用于旧的 Unity 动画系统**（已弃用，不应再使用）

### 8.6 Avatar 类型与 Playable Layers 对应关系

| Avatar 类型 | Playable Layers | 备注 |
|------------|----------------|------|
| **Humanoid** | Base + Additive + Gesture + Action + FX | 完整 5 层 |
| **Generic** | Base + Action + FX | 3 层，无 Additive/Gesture |

---

## 9. Culling Mode 行为（VRChat 特殊处理）⭐

> 来源：VRC School + VRChat 官方文档
> 重要性：理解 Culling Mode 是性能优化的关键
> 验证状态：2026-07-04

### 9.1 Animator Culling Mode 三种模式

| 模式 | 行为 |
|------|------|
| **Always Animate** | 即使离屏时也保持整个 Avatar 动画化 |
| **Cull Update Transforms** | 渲染器不可见时禁用变换值变化（包括启用/禁用 GO、组件、修改 Blendshape）。**例外**：AAP 和触发状态机行为 |
| **Cull Completely** | 渲染器不可见时冻结整个 Animator（不处理逻辑、不检查过渡、不动画）|

### 9.2 VRChat 默认 Culling Mode

> ⚠️ **VRChat 中**:
> - **本地 Avatar 的 Animator → 设置为 Always Animate**
> - **远程 Avatar 的 Animator → 设置为 Cull Update Transforms**
> - **子 GameObject 上的 Animator 不受此影响**

### 9.3 性能含义

- 远程 Avatar 离屏时大幅减少计算（性能优化）
- 本地 Avatar 始终保持响应（玩家体验）
- 子 GameObject 动画不受 VRChat 默认设置影响

### 9.4 AAP 例外

- 动画化动画参数（AAPs）即使在 Cull Update Transforms 模式下也会工作
- 触发状态机行为也会被处理

### 9.5 创作者建议

- 创作者无需修改默认设置（VRChat 自动处理）
- 子 GameObject 动画（如武器、配件）应使用 Always Animate 以保持交互性

---

## 10. 关键警告与实操补充（2026-07-04 来自 yexcadocs 实操经验）

> 来源:yexcadocs §State Behaviors / VRC Avatar Descriptor / Animator Layers(P1/P2)
> 本节为社区实操经验补充,非 VRChat 官方文档。

### 10.1 ⚠️ Animator Locomotion Control 必须配对使用(P1)

**核心规则**:
- 在进入"需禁用 Locomotion"的状态(如舞蹈)前添加 `Disable`
- 在退出状态时(如 BlendOut)添加 `Enable`
- **缺失任一 → 模型乱动或无法移动**

**典型场景**:自身跳 MMD 舞蹈
```
Entry → WaitForActionOrAFK → Prepare Standing → 舞蹈状态 → BlendOut Stand
                                          ↑ 状态添加 Locomotion Control = Disable
                                                            ↑ 状态添加 Locomotion Control = Enable
```

**错误后果**:
- 仅 Disable 无 Enable → 跳舞结束后"再也无法移动或转动视角"
- 仅 Enable 无 Disable → "游戏中转动或移动时模型也会动"

详见 `self-mmd.md` 完整工作流。

### 10.2 📌 Avatar Descriptor Colliders 双重身份(P1)

**重要事实**:
Avatar Descriptor 的 Colliders 不仅是**物理碰撞体**,也是 **Contacts Sender**(默认参数为 "Head"/"Body"/"Hand L"/"Hand R" 等)。

**含义**:
- 手动设置 Contacts 时,**无需重复添加这些部位的 Sender**
- 默认 Colliders 已提供 Sender 功能

详见 `contact.md §1 概述补充`。

### 10.3 📌 各层典型应用模式(P1)

| 层 | 典型应用 | 常用资源 |
|---|---------|---------|
| **Base** | 站姿 / 蹲姿 / 趴姿 | `proxy_stand_still` 等可替换 |
| **Action** | 舞蹈 + AFK | AFK 三状态:Afk Init / AFK / BlendOut |
| **FX** | 物品开关 + 参数驱动 | Toggle / Slider / Button 驱动 |

### 10.4 📌 Interruption Source 与 Trigger 注意事项(P2)

| 设置 | 行为 |
|------|------|
| **Interruption Source = Current State** | 当前状态可被新过渡打断 |
| **Interruption Source = Next State** | 下一状态可被打断 |
| **Interruption Source = Current Then Next** | 当前 → 下一状态顺序可被打断 |
| **Interruption Source = Next Then Current** | 下一 → 当前状态顺序可被打断 |

> ⚠️ **Trigger 在 VRChat 中**:Unity 状态机中 Trigger 可正常工作,VRChat 行为需人工验证(标记为 [待证])
>
> 建议:关键路径使用 Bool/Int 替代 Trigger 触发条件。

---

## 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| 1.0 | 2026-06-05 | 初始创建 |
| 1.1 | 2026-07-04 | 补充 VRC School 内容(§1.1 Avatar 3.0 基础 + §8 Avatar Rigs + §9 Culling Mode)|
| 1.2 | 2026-07-04 | 新增 §10 关键警告与实操补充(Animator Locomotion Control 配对 + Colliders 双重身份 + 各层典型应用 + Interruption Source 注意事项) |

## 来源

- creators.vrchat.com/avatars/playable-layers/
- vrcfury.com/components/

### 补充来源（2026-07-04 纳入 VRC School + VRChat 官方）

- VRC School: https://vrc.school/
- VRC School GitHub: https://github.com/VRLabs/VRCSchool
- VRChat 官方 Avatars: https://creators.vrchat.com/avatars/
- VRChat 官方 Avatar Performance: https://creators.vrchat.com/avatars/avatar-performance-ranking-system/
- VRC School Avatar Masks: https://vrc.school/docs/Avatars/Avatar-Masks/

---

## 第三方工具中的 Write Defaults 行为（2026-07-04 新增）

### VirtualLens2 行为

> VirtualLens2 会在设置过程中尝试根据现有的 FX 层的动画控制器自动调整设置。然而，在以下情况下，它可能无法确定设置，需要手动调整：
> - 您在安装后更改了设置。
> - 在安装过程中存在"写入默认值"设置的混合。

**当使用非破坏性设置时**:
- 将"**MA 菜单安装器**"组件添加到具有 VirtualLens 设置组件的对象上
- 可以根据其设置添加菜单项
- 通过正确配置添加的组件的"**安装到**"参数，可以更改菜单项的安装位置

### VRCLens 行为

文档未深入说明 Write Defaults，建议参考官方 Canny 状态。

### 通用建议

- 整个 Avatar 保持 **Write Defaults 一致**（全 On 或全 Off）
- **混合模式警告**: SDK 会检测到 "Mixed Write Defaults" 并发出警告
- 参考 [L1 官方文档](https://creators.vrchat.com/avatars/#write-defaults-on-states) 获取完整指南

### 引用

- [L1] VRChat Creator Docs - Write Defaults on States: <https://creators.vrchat.com/avatars/#write-defaults-on-states>
- 详细技术分析见 [[entities/avatar/camera-tools]] §2.4 / §3
---

## 相关页面

[[animator-system.md]] · [[avatar-modding-guide.md]] · [[avatar-parameter-staged-sync.md]] · [[lac-avatar-compressor.md]] · [[ma-component-cards.md]]
