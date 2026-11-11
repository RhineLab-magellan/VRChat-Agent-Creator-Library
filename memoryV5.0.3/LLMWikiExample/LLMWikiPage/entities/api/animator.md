---
title: "API: Animator"
category: api
knowledge_level: core
status: active
source: "社区经验 + 项目实测 + Unity官方 2022.3 + unityvrchat 基准"
source_type: community
version: 1.1
last_review: 2026-08-17
confidence: high
tags:
  - api
  - animator
  - udonsharp
aliases:
  - 动画器
  - Animator
related:
  - not-exposed.md
  - official-doc-clarifications.md
  - player-api.md
  - udonsharp-runtime.md
  - ui.md
type: entity
created: 2026-06-04
sources: "社区经验 + 项目实测 | https://docs.unity3d.com/2022.3/Documentation/Manual/class-Animator.html | https://creators.vrchat.com/avatars/playable-layers"
updated: 2026-08-17
---
# API: Animator

---

## Animator 在 Udon 中的暴露

### Animator.SetInteger(string name, int value)
- **暴露**: ✅
- **热路径**: ✅ (轻量参数设置)
- **说明**: 设置 Animator int 参数。推荐用 int 驱动状态而不是 Trigger。

### Animator.SetFloat(string name, float value)
- **暴露**: ✅
- **热路径**: ✅
- **说明**: 设置 Animator float 参数。

### Animator.SetBool(string name, bool value)
- **暴露**: ✅
- **热路径**: ✅
- **说明**: 设置 Animator bool 参数。

### Animator.SetTrigger(string name)
- **暴露**: ⚠️ 需要验证
- **说明**: 不确定在 Udon VM 中是否稳定。推荐用 SetInteger 替代。

### Animator.GetInteger / GetFloat / GetBool
- **暴露**: ⚠️ 需要验证
- **说明**: 读取 Animator 参数值。

### Animator.Play(string stateName)
- **暴露**: ⚠️ 需要验证
- **说明**: 直接播放指定动画。可能不受 Animator Controller 过渡条件约束。

### Animator.speed
- **暴露**: ✅
- **说明**: 动画播放速度倍率。

## 推荐用法

### 用 int 状态参数替代 Trigger
```csharp
// ✅ 推荐: int 参数驱动
_animator.SetInteger("DoorState", 1);  // 1 = Open

// ⚠️ 可能不稳定: Trigger
_animator.SetTrigger("OpenDoor");
```

### 网络同步模式
- Animator 参数由本地 UdonBehaviour 设置
- 状态变化通过 `[UdonSynced]` 变量同步
- 远端在 `FieldChangeCallback` 中设置 Animator 参数
- 不直接在 Network Event 中操作 Animator

## 常见错误
- 频繁设置 Animator 参数（每帧 SetFloat 等）
- 依赖 Trigger 的瞬时性做同步（Trigger 状态不可查询）
- 多个 Behaviour 同时设置同一个 Animator 参数

## 高级技巧（未验证）: BlendTree 卸载计算

> ⚠️ **未验证 Idea** — Animator BlendTree 是否运行在独立线程仍不确定。可能仍在 Unity 主线程执行，因此不能保证真正"卸载"Udon VM。使用风险自负。

### 思路
1. `Animator.SetFloat()` 写入输入参数
2. BlendTree 进行计算
3. 下一帧 `Animator.GetFloat()` 读取结果

### 限制
- 完全未实测验证
- BlendTree 数学能力有限（非通用计算单元）
- 一帧延迟

### 替代的并行卸载技术
- `OnAudioFilterRead`: 音频线程中执行，**确认并行，但有数据竞争风险**
- GPU Callback: 将计算委托给 GPU，**确认可用，延迟高，仅适合批量数据**

### Reference
- vrc.school/docs/Other/Advanced-BlendTrees
- `patterns/unorthodox-patterns.md` Pattern 4（含三种并行卸载技术对比）

---

## 组件/用法侧补充（2026-08-17 批次调研，[FACT]/[INFERENCE] 标注）

> 来源：Unity Manual/Scripting API 2022.3 + `[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（本地性能基准）+ VRChat 官方文档。调研笔记：`2026-08-17-native-unity/animator.md`。

### 组件核心（[FACT]，Unity 2022.3）
- Animator 组件将「动画层 + 状态机 + 混合树」应用到 GameObject。关键字段：`Animator Controller`、`Avatar`（可选；有 humanoid Avatar 时启用 IK 与肌肉求值）、`Apply Root Motion`、`Update Mode`（Normal / Animate Physics / Unscaled Time）、`Culling Mode`。
- API 侧（Play/CrossFade/SetFloat 等）见本页上文「Animator 在 Udon 中的暴露」。
- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-Animator.html

### 性能关键数据（[FACT] 转录自单一本地社区基准，按 §9.4 建议按 medium 口径引用）
- 每增加一个动画层约 **0.01 ms**（近似线性，整体二次曲线）。
- 频繁切换（面部/眼部追踪）比静态高约 **20–30%**。
- AnyState 与普通切换性能接近；「Can Transition to Self」的 AnyState 比不带它高约 **20%**。
- Direct Blend Tree 比基础层切换省约 **3/4**（慢方案）/ **4/5**（两层的快速方案）。
- **状态行为（State Machine Behaviours）存在时整体 +50%**；humanoid 空层/非 humanoid/无 Avatar 每层约 -50%。
- 本地 Avatar 参数约 **每 1000 个 1.5 ms**（不适用于远程 Avatar）。
- Write Defaults：层 WD 开关影响帧时间（WD off 对 layer toggle 约 +50%）。
- 多控制器非线性：1 个 100 层 ≈ 10 个 30 层开销。
- 本地源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（Animator Controllers 章节）——**单源社区基准（vrc.school/unityvrchat 测量），官方未公布同口径数据**，引用时按 medium 置信度对待。

### VRChat 侧注意（[FACT]/[INFERENCE]）
- **Avatar 3.0 五 Playable Layer**：头像经 Avatar Descriptor 的 5 个 Playable Layer 槽（Base/Additive/Gesture/Action/FX）加载 AnimatorController；官方明确「不要再在头像上添加额外 Animator」，合并成一个根 Animator（https://creators.vrchat.com/avatars/playable-layers）。
- **参数同步**：VRChat 最多同步 256 位自定义参数、头像 8192 个表达式参数；官方不建议 Trigger 型参数（准确性/去同步），推荐 Float/Bool（https://creators.vrchat.com/avatars/animator-parameters/）。
- **World 内 Animator**：World 可直接用 Unity Animator 组件 + NavMeshAgent/Udon 驱动；官方 AI Navigation 示例即 NavMeshAgent + Animator 做 NPC 行走动画。白名单内（`entities/world/whitelisted-world-components.md`）。
- **性能警告（[COMMUNITY]，VRChat Unity Animator Lag）**：层数多时 `Animators.IKAndTwistBoneJob` 重复运行（仅 humanoid avatar 时），空层也触发；移除 armature 后消失 → 优化层数/用 Blend Tree 打包是关键。源：https://feedback.vrchat.com/bug-reports/p/vrchat-unity-animator-lag

## 相关页面

[[not-exposed.md]] · [[official-doc-clarifications.md]] · [[player-api.md]] · [[udonsharp-runtime.md]] · [[ui.md]] · [[../world/whitelisted-world-components.md]] · [[../avatar/animator-system.md]]
