---
title: "Avatar 参数分步同步架构"
category: avatar
knowledge_level: applied
status: active
source: "VRCD 文档库 — LIII Works（酒保K）"
source_type: official
version: 1.2
last_review: 2026-07-04
confidence: High
tags:
  - avatar
  - playable-layer
  - sync
  - parameter-driver
  - puppet
  - cross-platform
  - vrcd-21-supplement
aliases:
  - "Avatar 参数分步同步架构"
  - avatar-parameter-staged-sync.md
  - "分步同步"
related:
  - modular-avatar.md
  - playable-layers.md
  - accessories.md
  - animator-system.md
  - avatar-dynamic-bone-limits.md
type: entity
created: 2026-07-04
sources: "VRCD 文档库 — LIII Works（酒保K）"
updated: 2026-07-04
---
# Avatar 参数分步同步架构


---

## §0 入门级心智模型：两个盒子 + 信息传递（VRCD 21 补充）⭐

> **来源**: [Avatar 文档（vrcd.org.cn）](https://docs.vrcd.org.cn/books/avatar) §第三部分「同一个 Parameter 为什么要写在两个地方」 + VRChat 官方 Animator Parameters 文档
> **本地化日期**: 2026-07-04
> **重要程度**: 🟡 教学性补充,非工程硬性要求

### 0.1 为什么需要同步？（教学性解释）

> 想象你的游戏在一个盒子中,别人的游戏也在他自己的盒子中,两个盒子有一模一样的场景、物件、安排...等等。你们有在一起的感觉是因为你们看到了对方对这个世界造成的影响。如果要让对方看到你造成的影响,你**必须要**把信息传递过去。

**关键隐喻**:
- 本地 Avatar = 你盒子里的实例
- 远端玩家 Avatar = 别人盒子里的实例
- 信息传递 = Parameter 同步
- **没有同步 = 对方永远看不到你做的任何事**

### 0.2 同步示例（VRCD 21 原文）

```
玩家 A 的 Avatar 上一个名为 cube_IsOn 的 parameter 从 true 变成 false
↓
cube_IsOn 的变化使得 Animator 进入了一个方块被关闭的状态
↓
播放了一个关闭方块的动画,使得方块被关闭了
↓
这个变化的信息被发送给了玩家 B
↓
玩家 B 看到你的方块被关闭了
```

### 0.3 "双处同步"心智模型（教学性简化）

**为什么同一个 Parameter 要写在两个地方？**

| 位置 | 用途 |
|------|------|
| **Animator 控制器** | 用于 Animator 的过渡条件 |
| **Expression Parameters** | 用于**与远端玩家同步**,控制 Animator 参数,以及用于 Expression Menu |

**教学性简化规则**:
- 🔴 想让 Animator 用的 Parameter → **必须**在 Animator 参数表里
- 🔴 想同步给远端玩家的 Parameter → **必须**在 Expression Parameters 中**同名**且**勾选 Sync**
- 🔴 同一个 Parameter 在两处的名称必须**完全一致**

### 0.4 Cross-Platform Sync：基于 Position 而非 Name（VRChat 官方）

> [FACT] VRChat 官方原文: "When using an avatar that has both Quest and PC versions uploaded, parameters are synced by their **position in the parameters list and their parameter type, not by the names of the parameters**."

**关键含义**:
- Quest 和 PC 双版本 Avatar → 同步基于**位置**和**类型**,不是名称
- 同一参数在 PC 和 Quest 端必须在参数列表的**相同位置**,且**类型相同**
- 这是 PC/Quest 双版本开发的硬性约束

### 0.5 教学场景中的应用

**对创作者教学的建议**:
- 用"两个盒子" 比喻让初学者理解同步本质
- 用"信息必须传递过去" 强调没有同步 = 远端看不到
- 用简化版"双处同步" 规则作为入门心法
- 工程性细节（256 bits 上限、Sync 类型）留到进阶内容

### 0.6 引用

- [L1] https://creators.vrchat.com/avatars/animator-parameters/ - "Cross-Platform Parameter Sync" 章节
- [L2] https://docs.vrcd.org.cn/books/avatar - VRCD 21「Avatar 与你」§Parameter
- [L3] https://wiki.vrchat.com/wiki/Expressions - Late-joiner 推荐模式

---
## 概述

**分步同步**是一种用于大幅减少 Synced Parameter Bit 消耗的主动系统，以容纳超过 256 bit 的同步参数。

### 核心思想
将参数同步从**同时**拆分为**多次**，在一段时间内**分批完成**，用时间换空间。

### 背包比喻
- 同时同步：一次性背上所有物品
- 分步同步：去上学装课本、去登山装食物、去旅行装地图
- 每次只用一个同步容器，分批传递所有参数

---

## 核心约束

### VRChat 参数同步限制
| 类型 | 占用 | 说明 |
|---|---|---|
| **Bool** | 1 bit | - |
| **Int/Float** | 8 bit | - |
| **总计上限** | 256 bit | 每个 Avatar |
| **内置参数** | 不计入 | VRChat 内置参数不占 bit 配额 |

> **💡 生效条件**：仅当参数被**勾选 Sync** 时才消耗同步 bit。未勾选 Sync 的参数变化**仅本地生效**（不通过网络传输，也不计入 256 bit 上限）。

> **来源**：[FACT] creators.vrchat.com/avatars/animator-parameters/ "Only synced values are counted towards bits being used" + vrc.school/docs/Other/Network-Sync/

### 注册要求（双重注册）

> **🔴 关键流程**：参数同步需要**双重注册**才能生效。

1. **第一重 — Playable Layer 动画控制器**：先在 Animator Controller 中添加参数（决定参数的行为/动画）
2. **第二重 — Expression Parameters List**：在 `VRCExpressionParameters` 对象中添加**同名**参数，勾选 Sync 标记

> **未注册的影响**：
> - 仅存在于 Playable Layer 控制器 → 参数变化**不通过网络同步**，可被 Parameter Driver 驱动（仅本地）
> - 仅存在于 Expression Parameters → 同步列表中存在但**无法被 Animator 使用**
> - 完整双重注册 → 正常同步

> **来源**：[FACT] creators.vrchat.com/avatars/animator-parameters/ + vrc.school/docs/Other/Network-Sync/

### 同步间隔与延迟
| 参数 | 值 | 说明 |
|---|---|---|
| **Sync Interval** | 0.2s | 每 0.2 秒发起一次同步 |
| **Latency** | 0.1 ~ 1.0s | 抵达远端时间 |

---

## 同步类型（VRChat 官方）

| 类型 | 用途 | 更新频率 | 延迟 |
|---|---|---|---|
| **Speech** | Viseme（唇形同步） | 本地计算 | 无显著延迟 |
| **Playable** | 一般参数 | 每 0.1-1s | ~0.2s |
| **IK** | 频繁变化值（见下方说明） | 每 0.1s | 低 |

### IK 同步使用限制

> **🔴 关键限制**：IK 同步类型的使用场景是受限的。

- **触发条件 1**：由 **Radial Puppet** 或 **Axis Puppet** 控制的 Float 参数
- **触发条件 2**：VRChat 内部指定参数（与玩家 IK 状态相关）
- **切换机制**：启用 Puppet 控制时，VRChat 自动从 Playable 切换为 IK 同步；关闭 Puppet 后回到 Playable 同步
- **远端插值**：本地为远程用户**插值 float 值**（10 updates/s 连续更新）

> **意义**：分步同步使用 Playable 同步类型，**不会**因 Puppet 控制自动升级到 IK。

> **来源**：[FACT] creators.vrchat.com/avatars/animator-parameters/ "when you control a parameter with a Puppet control, VRChat switches from Playable to IK sync, improving the update rate smooth interpolation. When you close the Puppet control, it returns to Playable sync"

---

## 核心公式

### Bit 节省公式
```
节省 Bit = (N - ceil(N / Steps)) × BitPerParam
```
- N = 参数数量
- Steps = 同步步骤数
- BitPerParam = 每参数 bit 数（Bool=1, Int/Float=8）

**示例**: 10 个 Int，4 步同步
- 原始: `10 × 8 = 80 bit`
- 分步后: `ceil(10/4) × 8 = 24 bit`
- 节省: `56 bit`

### 延迟公式
```
Delay = Steps × 0.2s
```
**示例**: 4 步 = 0.8s 延迟

### 容器数量公式
```
Containers = ceil(ParamCount / Steps)
```
**示例**: 10 个 Int，4 步 = 3 个容器

---

## 架构组件

### 组件概述
```
┌─────────────────────────────────────────────────────────────┐
│                    分步同步架构                              │
├─────────────────────────────────────────────────────────────┤
│  远端区分 → 判断本地/远端，决定使用接收器或发射器            │
│                                                             │
│  发射器 ──→ 同步容器 + 时序标记 ──→ 接收器                  │
│    │              │                     │                  │
│    ↓              ↓                     ↓                  │
│  本地写入    每0.2s同步间隔      远端根据时序读取             │
└─────────────────────────────────────────────────────────────┘
```

### 远端区分
- **功能**: 区分 Avatar 由玩家穿着（本地）还是被观察（远端）
- **实现**: 使用 `IsLocal` 判断

### 发射器
- **功能**: 将本地参数值写入同步容器，设定同步时序
- **实现**: Parameter Driver (Set)
- **注意**: 每步骤间需间隔 0.2s

### 接收器
- **功能**: 根据时序标记，将同步容器内容写入对应参数
- **实现**: Parameter Driver (Copy)
- **注意**: 不需要关心 0.2s 间隔，不断尝试接收即可
- **时序灵活性** [INFERENCE]: 时序标记可以使用任何形式（Int/Bool/其他），不必与发射器保持同一形式。设计建议：超过 7 步时使用 Bool 效率更高。

### 同步容器
- **功能**: 承载要同步的参数数值
- **类型**: Bool / Int / Float
- **优化**: Int 和 Float 可互转，Float 精度 1/127，建议发送端转 Int，接收端逆转换
- **计算等效** [INFERENCE]: 计算 Bit 消耗时，**Int 与 Float 视为同一类型**（均 8 bit/参数），可独立计算每种类型再求和。Bool（1 bit）需单独计算。

### 时序标记
- **功能**: 指示当前同步步骤
- **实现**: Bool 数组或 Int
- **位数计算**: `log2(Steps)`，超过7步用 Bool 更高效

#### 时序标记位数对照
| 步骤数 | 所需位数 | 备注 |
|---|---|---|
| 2步 | 1 bit | 2个状态 |
| 3-4步 | 2 bit | 4个状态 |
| 5-8步 | 3 bit | 8个状态 |
| 9+步 | 4+ bit | 用 Bool 效率更高 |

---

## 设计权衡

### 时间换空间
| 因素 | 说明 |
|---|---|
| **步骤越多** | 延迟越大，但 Bit 消耗越少 |
| **步骤越少** | 延迟越小，但容器数量增加 |

### 优化建议
- 根据功能延迟容忍度选择步骤数
- 高频交互功能 → 少步骤（2-3步）
- 低频功能 → 多步骤（4+步）
- 超过7步用 Bool 作为时序标记更高效

---

## Parameter Driver 参考

### 基础行为

> **执行顺序**：单个 Avatar Parameter Driver 的**多个操作按从上到下顺序依次完成**。
>
> - 行为在**进入 State 时**触发（**仅一次**）
> - 多个操作依次执行，操作之间无延迟
> - 操作的 Source/Destination 参数为字符串名称匹配
>
> **同步条件**：仅在 `VRCExpressionParameters` 对象中**定义**的参数才会通过网络同步。
>
> - 未在 Expression Parameters 中定义 → 行为仍可执行（仅本地生效）
> - 已定义但未勾选 Sync → 仅本地生效
> - 已定义并勾选 Sync → 正常同步（受 Int[0,255] / Float[-1,1] 范围限制）

> **来源**：[FACT] creators.vrchat.com/avatars/state-behaviors/ "A single Avatar Parameter can perform multiple operations, and they are done in order from top to bottom. These operations are completed once upon entry to the State"

### 操作类型
| 操作 | 功能 | 约束 |
|---|---|---|
| **Set** | 设置指定值 | 无特殊限制 |
| **Add** | 添加值 | 建议仅本地运行 + 同步 |
| **Random** | 设为随机值 | 建议仅本地运行 + 同步 |
| **Copy** | 复制参数值 | 源：不能是 VRChat 内置参数；目标：必须在 Expression Parameters 中 |

### 数值范围限制
| 类型 | 范围 | 说明 |
|---|---|---|
| **Int** | [0, 255] | 同步限制 |
| **Float** | [-1, 1] | 同步限制 |
| **Float 精度** | 1/127 | 8 bit quantization |

### Local Only 选项
- 启用后仅在本地操作，不进行网络同步
- 可替代 `IsLocal` 判断的繁琐步骤

### Debug String（调试字符串）

> **功能**：每个 Avatar Parameter Driver 操作都可以附加一个**调试字符串**字段。
>
> - 当该行为运行时，字符串会被**输出到日志**（VRChat 日志或 Unity Console）
> - 用于调试和问题排查
> - 字段是**每个操作**级别的（不是 Driver 级别）
> - **所有 State Behavior 都有该字段**（不仅 Parameter Driver）

> **来源**：[FACT] creators.vrchat.com/avatars/state-behaviors/ Avatar Parameter Driver 表格 "Debug String | When this StateBehavior runs, this string will be printed to the output log. Useful for debugging." + VRC School "every State Behaviour has a `Debug String` field" + VRC SDK `VRC_AvatarParameterDriver.debugString` 字段（VRCSDKBase.cs:3474）

### 重要约束
- VRChat 内置参数（如 `GestureLeftWeight`）**不能**作为 Copy 的源参数
- 目标参数必须在 Expression Parameters 中才能同步
- 即使目标不在同步表中，Parameter Driver 仍可驱动（仅本地生效）

---

## 最佳实践

1. **分析功能延迟容忍度** — 确定合适的同步步骤数
2. **分组设计** — 将功能按更新频率分组，高频少量同步，低频多量同步
3. **时序标记优化** — 超过7步时使用 Bool 而非 Int
4. **精度管理** — Float 发送前转 Int，接收后逆转换保留最大精度
5. **测试验证** — 验证远端同步效果和延迟可接受性

---

## 相关资源

- [VRChat Animator Parameters 文档](https://creators.vrchat.com/avatars/animator-parameters/)
- [VRChat Avatar Parameter Driver](https://creators.vrchat.com/avatars/state-behaviors/#avatar-parameter-driver)
- [VRC School — Network Sync](https://vrc.school/docs/Other/Network-Sync/)
- [VRC School — State Behaviours](https://vrc.school/docs/Avatars/State-Behaviours/)
- [VRCD 文档库 — Avatar 参数分步同步架构（白皮书）](https://docs.vrcd.org.cn/books/avatar-dhs)

---

## ⚠️ 硬性限制补充（2026-07-04 重要补充）

> 🔴 **来源**: VRChat 官方 Creator Docs + 官方 Canny feedback

### 已知硬性限制

| 项目 | 限制 | 验证来源 |
|------|------|----------|
| **Synced 同步参数配额** | **256 bits** per avatar | L1 官方文档 |
| **总自定义 Expression Parameters** | 8192 个（synced + unsynced 之和）| L1 官方文档 |
| **已知 bug**: 总参数 > 256 时加载失败 | ⚠️ SDK 允许添加但实际加载失败 | L2 反馈页 tracked |

### 单位消耗表（与官方一致）

| 参数类型 | 范围 | 同步消耗 |
|----------|------|----------|
| `int` | 0-255 | 8 bits |
| `float` | -1.0 ~ 1.0 | 8 bits |
| `bool` | True/False | 1 bit |

### 实际影响（来自反馈页）

> "The limit for the ExpressionParameters loading correctly does indeed appear to be 256 plus some local parameters is ok, until you have more than 256 total."
> — StormRel, VRChat open beta 1434 反馈

**关键发现**:
- 实际硬性限制比官方"256 bits synced"更严
- 总参数（含 unsynced）超过 256 时，参数**加载失败且不可见**
- 错误日志: `Error - [Behaviour] Unable to initialize stage parameters, defined parameters use more memory then allowed.`

### 第三方工具验证

- **VirtualLens2**: 需要至少 8 bit 余量（1 个 Int 或 Float 空间）
- **VRCLens**: 同样需要 8 bit 余量

**对创作者的建议**:
- 始终保持至少 8-16 bit 缓冲（避免添加新组件时撞墙）
- 工具集成前先检查 `VRCExpressionParameters` 已用配额
- 关注官方修复进展（反馈页状态为 tracked）

### 引用

- [L1] <https://creators.vrchat.com/avatars/animator-parameters/>
- [L2] <https://feedback.vrchat.com/avatar-30/p/1332-bug-vrcexpressionparameters-fail-to-load-correctly-with-more-than-256-param>
---

## 相关页面

[[modular-avatar.md]] · [[playable-layers.md]] · [[accessories.md]] · [[animator-system.md]] · [[avatar-dynamic-bone-limits.md]]
