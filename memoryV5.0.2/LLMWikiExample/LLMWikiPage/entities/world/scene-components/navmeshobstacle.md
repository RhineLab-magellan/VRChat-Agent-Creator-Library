---
title: "NavMeshObstacle — 世界导航障碍组件"
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
aliases:
  - NavMeshObstacle
  - 导航障碍物
related:
  - ../whitelisted-world-components.md
  - ../udon/ai-navigation.md
  - ../examples/ai-navigation.md
  - navmeshagent.md
  - offmeshlink.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-NavMeshObstacle.html | https://docs.unity3d.com/2022.3/Documentation/Manual/nav-MixingComponents.html | https://creators.vrchat.com/worlds/whitelisted-world-components/ | https://creators.vrchat.com/worlds/examples/ai-navigation/"
updated: 2026-08-17
---
# NavMeshObstacle — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无本组件独立帧时间数据，详见性能小节）

---

## 核心功能（官方，[FACT]）

NavMeshObstacle 描述**移动障碍物**，NavMeshAgent 需绕开它。运行时行为取决于 **Carve** 开关：

- **Carve 关闭**：障碍物仅被代理以「碰撞避让」方式避开（代理推挤绕行）。
- **Carve 开启**：障碍物静止时在 NavMesh 上**挖洞（carving）**，代理会计算改道绕行；障碍物移动时退化为 obstruction。
- 若物体带 Rigidbody，Obstacle 的速度自动取自 Rigidbody，供代理做运动预测避让。

源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-NavMeshObstacle.html

## 关键属性/行为

| 属性 | 说明（[FACT]） |
|---|---|
| **Shape** | Box / Capsule 两种障碍形状。 |
| **Center** | 障碍中心相对 Transform 的偏移。 |
| **Size**（Box）/ **Radius + Height**（Capsule） | 障碍体积参数。 |
| **Carve** | 是否在 NavMesh 上挖洞；关闭则仅做代理避让。 |
| **carvingMoveThreshold**（Move Threshold） | 障碍移动超过该阈值才更新洞的位置。 |
| **carvingTimeToStationary**（Time To Stationary） | 保持静止多久后重新挖洞。 |
| **Carve Only Stationary** | 默认开：移动时移除洞，静止超过 `carvingTimeToStationary` 后重新挖洞；关闭则洞随移动实时更新（适合大型慢速障碍，如坦克/步兵）。 |

**行为约束（[FACT]，Unity Manual: Mixing Components）**：
- **与 [[navmeshagent.md|NavMeshAgent]] 同 GameObject 不兼容**——同时启用会让代理试图避让自己；再开 Carving 更糟（不断重映射到洞边缘）。应只用其一；对象「死亡/静止」时可关 Agent 开 Obstacle 让其被动避让。
- 移动障碍与静态 NavMesh 本质互补：静态几何烘焙进网格，动态障碍才用 Obstacle/Carving。

## VRChat 特定限制/注意

**官方地位**：NavMeshObstacle 是 World 白名单组件 —— Allowlisted World Components「Unity Components (77) / Navigation」段，World 可用；VRChat **无 NavMeshObstacle 专属文档页**，归入 AI Navigation 示例 + Allowlist 覆盖。见 [[../whitelisted-world-components.md|whitelisted-world-components]]。

> ⚠️ **[UNKNOWN/时效]** VRChat 对 Unity 2022 NavMesh 的兼容未官方定论：社区报告（Ask Forum 2024-02）需用 legacy「Window > AI > Navigation (Obsolete)」烘焙；官方 AI Navigation 示例使用 NavMeshSurface 运行期重建。**建议目标工程实测后复核**。参见 [[../udon/ai-navigation.md|AI Navigation]]。

**[FACT] 官方示例的动态障碍用法**（[[../examples/ai-navigation.md|AI Navigation Example]]）：蓝色积木（VRCPickup）被玩家移动后调用 `NavMeshSurface.BuildNavMesh()` 重建网格，而非依赖 NavMeshObstacle 的 carving——World 中动态障碍可结合 `carving` 或 `BuildNavMesh` 两种路线实现。

**[UNKNOWN] Udon 可访问属性**：Unity 脚本 API 暴露 `carving` / `carvingMoveThreshold` / `carvingTimeToStationary` / `Carve Only Stationary` 等公共字段；其对 Udon 的具体暴露程度不在本次调研范围，需以 Udon Type Exposure 清单为准。官方示例中未直接经 Udon 操作 NavMeshObstacle（[FACT]）。

**[INFERENCE] Piggyback（背人/骑乘）类世界常用**：背人玩法中 NPC 需在人群/载具旁维持队形，NavMeshObstacle 常用于给寻路中的群体 NPC 标注"临时站位阻挡"（如被占用的座位区、排队栏杆），配合 `Carve Only Stationary` 在 NPC 停下时动态封路。

**[INFERENCE] 性能注意**：**carving 是 CPU 消耗型操作**（挖洞 + 洞边缘 remap 会触发相邻代理重新寻路）；多个 concurrently carving 的障碍会放大开销。优先「静态烘焙 + 少量 Carve 障碍」组合，谨慎使用 `Carve Only Stationary = false` 的实时跟随挖洞；`carvingMoveThreshold` 设过小会导致频繁重建洞。

**[COMMUNITY] 混用抖动**：社区多报道 NavMeshObstacle 与 NavMeshAgent 混用出现抖动问题（StackOverflow & Unity Manual，非 VRChat 专属但同样适用）——再次印证二者应分离使用。

## 性能数据（unityvrchat 或 [UNKNOWN]）

`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）` **无 NavMeshObstacle 独立帧时间数据** → **[UNKNOWN]**。

## 社区佐证

- StackOverflow「How to make nav mesh agents see each other as obstacles」：社区关于 Agent/Obstacle 混用抖动的讨论（[COMMUNITY]，非 VRChat 专属，但同样适用）；更优做法是把其他代理当作障碍处理或调整避障参数。

## 相关页面

[[navmeshagent.md|NavMeshAgent]] · [[offmeshlink.md|OffMeshLink]] · [[../udon/ai-navigation.md|AI Navigation]] · [[../examples/ai-navigation.md|AI Navigation Example]] · [[../whitelisted-world-components.md|Allowlisted World Components]]

## 源清单

- Unity Manual Nav Mesh Obstacle: https://docs.unity3d.com/2022.3/Documentation/Manual/class-NavMeshObstacle.html
- Unity Manual Mixing Components: https://docs.unity3d.com/2022.3/Documentation/Manual/nav-MixingComponents.html
- VRChat Allowlist: https://creators.vrchat.com/worlds/whitelisted-world-components/
- VRChat AI Navigation Example: https://creators.vrchat.com/worlds/examples/ai-navigation/
- 本地: `[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无数据）