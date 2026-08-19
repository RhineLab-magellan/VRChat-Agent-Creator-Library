---
title: "NavMeshAgent — 世界导航组件"
category: world
subcategory: scene-components
knowledge_level: applied
status: active
source: "Unity Manual 2022.3 + VRChat 官方文档"
source_type: official
version: 1.0
last_review: 2026-08-17
confidence: medium
tags:
  - world
  - unity
  - navmesh
  - performance
  - sync
aliases:
  - NavMeshAgent
  - 导航代理
related:
  - ../whitelisted-world-components.md
  - ../udon/ai-navigation.md
  - ../examples/ai-navigation.md
  - navmeshobstacle.md
  - offmeshlink.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-NavMeshAgent.html | https://docs.unity3d.com/2022.3/Documentation/Manual/nav-MixingComponents.html | https://creators.vrchat.com/worlds/examples/ai-navigation/ | https://creators.vrchat.com/worlds/whitelisted-world-components/ | https://ask.vrchat.com/t/udon-navmeshagent-works-in-unity-but-not-when-built-and-tested/21745 | https://feedback.vrchat.com/bug-reports/p/stuttering-on-player-pickups"
updated: 2026-08-17
---
# NavMeshAgent — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无本组件独立帧时间数据，详见性能小节）

---

## 核心功能（官方，[FACT]）

NavMeshAgent 是 AI 导航代理：在烘焙的 NavMesh 上**寻路**并自动移动 GameObject。设置 `destination` 后，代理自动寻找路径、**避障**、沿路径移动，无需逐帧手写移动逻辑。

- 需要场景中存在烘焙 NavMesh（Unity Navigation 系统 / AI Navigation 包）。
- 代理之间**无需物理 Collider 即可互相避让**（导航系统内部模拟代理）。
- 源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-NavMeshAgent.html

## 关键属性/行为

| 属性 | 说明（[FACT]） |
|---|---|
| **radius / height** | 代理寻路与避障用的圆柱体尺寸，决定能否通过狭窄通道。 |
| **speed / angularSpeed** | 移动线速度与转身角速度。 |
| **acceleration** | 加减速能力，影响起步与刹停。 |
| **stoppingDistance** | 到达目标前停止的距离（用于攻击/交互范围判定）。 |
| **avoidancePriority** | 避障优先级，数值越低越优先（多代理拥挤时）。 |
| **baseOffset** | 代理底部相对 Transform 位置的偏移（让代理"站"在地面上）。 |
| **autoBraking** | 到点自动刹车（关闭则滑行到目标）。 |
| **obstacleAvoidanceType** | 避障质量档位（No/ Low / Medium / Good / High），影响避障精度与 CPU 开销。 |

**行为约束（[FACT]，Unity Manual: Mixing Components）**：
- NavMeshAgent 与**非 kinematic Rigidbody** 同时启用会竞争移动 → 未定义行为，**不要**混用。
- **不要**在同一 GameObject 上同时启用 NavMeshAgent 与 [[navmeshobstacle.md|NavMeshObstacle]]（代理会试图避让自己；再开 Carving 会更糟）。
- `NavMeshAgent.velocity` 可用于预测其他对象移动，让代理提前避让。

## VRChat 特定限制/注意

**官方地位**：NavMeshAgent 是 World 白名单组件 —— Allowlisted World Components「Unity Components (77) / Navigation」段，与 Grid / NavMeshObstacle / OffMeshLink 同组，World 可用。见官方白名单清单 [[../whitelisted-world-components.md|whitelisted-world-components]]。

> ⚠️ **[UNKNOWN/时效]** VRChat 对 Unity 2022 NavMesh 的兼容未官方定论：社区报告（Ask Forum 2024-02）需用 legacy「Window > AI > Navigation (Obsolete)」烘焙；官方 AI Navigation 示例使用 NavMeshSurface 运行期重建。**建议目标工程实测后复核**。参见 [[../udon/ai-navigation.md|AI Navigation]]。

**[FACT] 官方 AI Navigation 示例的用法**（[[../examples/ai-navigation.md|AI Navigation Example]]，2024-11）：
- 用 NavMeshAgent + UdonBehaviour 做 NPC 寻路，`NavMeshSurface.BuildNavMesh()` 动态重建网格。
- **NavMeshAgent 只在 Owner 端运行**，位置由 `VRC_ObjectSync`（Continuous）同步给其他玩家；只有 NavMesh Owner 重建网格，避免多客户端重复 CPU 计算。
- 官方直接以 `agent.SetDestination(target.position)` 驱动（[FACT] 官方示例用例）。

**[UNKNOWN] Udon 可访问属性**：`SetDestination()` 与持续寻路模式为官方示例已验证入口（[FACT]）；`destination` / `velocity` 等属性是 Unity API 明确暴露的成员（[FACT] Unity Manual/API），但其对 **Udon 的具体暴露程度**不在本次调研范围，使用时以 Udon Type Exposure 清单为准。

**[INFERENCE] Piggyback（背人/骑乘）类世界常用**：背人或骑乘玩法常让 NPC 自动跟随玩家、排队入座或驮运移动，NavMeshAgent 是这类「NPC 自主跟随」需求的常见实现载体；配合 `stoppingDistance` 可精确控制停靠间距。

**[INFERENCE] 性能注意**：**agent 数量直接影响避障 CPU 开销**（local avoidance 为逐代理仿真，`obstacleAvoidanceType` 越高越贵）；寻路路径查找与 NavMesh 重建同样吃 CPU。经验上应限制并发避障代理数量、降低避障档位，并严格保证 NavMesh 计算只发生在 Owner 端。

**[COMMUNITY] 自定义 Agent 类型**：社区报告自定义 Agent Type 在 VRChat 中可能不工作，建议使用默认 Humanoid 或减小代理 radius/height。

## 性能数据（unityvrchat 或 [UNKNOWN]）

`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）` **无 NavMeshAgent 独立帧时间数据**（该书覆盖 Animator/Constraints/Audio/Contacts/Cloth/PhysBones；NavMesh 未单测）→ **[UNKNOWN]**。

相关邻近事实（[FACT]，上下文非本体）：[[../udon/ai-navigation.md|AI Navigation]] 官方文档明确 **NavMesh 计算是 CPU 密集操作**，多客户端同时重建是"灾难"级风险——间接佐证 agent 数量与重建频率是主要性能变量。

## 社区佐证

- VRChat Ask Forum「NavMeshAgent works in Unity but not built」（2024-02）：Unity 2022 的 NavMesh 系统在 VRChat 构建后报「Failed to create agent because there is no valid NavMesh」，需用 legacy Navigation (Obsolete) 烘焙（[COMMUNITY/FACT]）。
- VRChat Feedback「VRCObjectSync + NavMeshAgent 卡顿」：NavMeshAgent + VRCObjectSync 使多目标位置同步出现卡顿（[COMMUNITY]）。

## 相关页面

[[navmeshobstacle.md|NavMeshObstacle]] · [[offmeshlink.md|OffMeshLink]] · [[../udon/ai-navigation.md|AI Navigation]] · [[../examples/ai-navigation.md|AI Navigation Example]] · [[../whitelisted-world-components.md|Allowlisted World Components]]

## 源清单

- Unity Manual NavMeshAgent: https://docs.unity3d.com/2022.3/Documentation/Manual/class-NavMeshAgent.html
- Unity Manual Mixing Components: https://docs.unity3d.com/2022.3/Documentation/Manual/nav-MixingComponents.html
- VRChat AI Navigation Example: https://creators.vrchat.com/worlds/examples/ai-navigation/
- VRChat Allowlist: https://creators.vrchat.com/worlds/whitelisted-world-components/
- VRChat Ask Forum（2022 NavMesh 兼容）: https://ask.vrchat.com/t/udon-navmeshagent-works-in-unity-but-not-when-built-and-tested/21745
- 本地: `[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无数据）