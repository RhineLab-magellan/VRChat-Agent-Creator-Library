---
title: "OffMeshLink — 世界导航离网链接组件"
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
  - OffMeshLink
  - 离网链接
related:
  - ../whitelisted-world-components.md
  - ../udon/ai-navigation.md
  - ../examples/ai-navigation.md
  - navmeshagent.md
  - navmeshobstacle.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-OffMeshLink.html | https://issuetracker.unity3d.com/issues/offmeshlink-does-not-calculate-the-path-correctly-after-carving-a-navmesh-at-runtime | https://creators.vrchat.com/worlds/whitelisted-world-components/"
updated: 2026-08-17
---
# OffMeshLink — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无本组件独立帧时间数据，详见性能小节）

---

## 核心功能（官方，[FACT]）

OffMeshLink 连接两个**不相连的 NavMesh**（例如跳沟、翻墙、过桥、开门后通行）。链接有两个锚点（Start / End Transform）；代理需要跨网格时走向最近的链接端点并穿越，实现**脱离网格几何的跳跃/落点移动**。

- 默认自动穿越；也可用脚本检测代理到达链接时**拒绝穿越**（如：需钥匙才能开的门）。
- 可通过烘焙时配置「Off Mesh Link Generation」自动生成，或手动放置 OffMeshLink 组件。

源：https://docs.unity3d.com/2022.3/Documentation/Manual/class-OffMeshLink.html

## 关键属性/行为

| 属性 | 说明（[FACT]） |
|---|---|
| **Start Transform / End Transform** | 标记链接两端的锚点 Transform。 |
| **Cost Override** | 覆盖该链接的路径代价，可让代理偏好/回避某条链接。 |
| **Bidirectional** | 是否双向可穿越（默认双向）。 |
| **Activated** | 链接激活开关，可脚本控制（配合"门是否已开"逻辑）。 |

- 端点需**贴近 NavMesh 表面**链接才生效。
- 烘焙时自动生成的链接受「Bake 设定：Drop Height / Jump Distance」约束。

## VRChat 特定限制/注意

**官方地位**：OffMeshLink 是 World 白名单组件 —— Allowlisted World Components「Unity Components (77) / Navigation」段，World 可用；VRChat **无 OffMeshLink 专属文档页**，归入 AI Navigation 示例 + Allowlist 覆盖。见 [[../whitelisted-world-components.md|whitelisted-world-components]]。

> ⚠️ **[UNKNOWN/时效]** VRChat 对 Unity 2022 NavMesh 的兼容未官方定论：社区报告（Ask Forum 2024-02）需用 legacy「Window > AI > Navigation (Obsolete)」烘焙；官方 AI Navigation 示例使用 NavMeshSurface 运行期重建。**建议目标工程实测后复核**。参见 [[../udon/ai-navigation.md|AI Navigation]]。

**[FACT] 官方示例中的静态配置**（[[../examples/ai-navigation.md|AI Navigation Example]]）：官方示例将 NavMeshLink / OffMeshLink 作为**静态配置**使用（AI Navigation 文档表：链接「静态配置」），未演示经 Udon 动态增删链接。

**[UNKNOWN] Udon 可访问属性**：`Activated` / `Cost Override` 等 Unity 公共字段在脚本上可读写，但对 Udon 的具体暴露程度不在本次调研范围，需以 Udon Type Exposure 清单为准；配合 Udon 判定（如开门事件）时建议优先走 `Activated` 开关而非移动锚点。

**[INFERENCE] Piggyback（背人/骑乘）类世界常用**：背人/接力玩法常需 NPC 跨越跳台/天桥/开关门，OffMeshLink 是让寻路 NPC 脱离常规 NavMesh 表面、走自定义跳跃/攀爬路径的常用手段（锚点人工摆放、配合门开关的 `Activated` 切换）。

**[INFERENCE] 性能注意**：OffMeshLink 本身以静态配置为主、运行期开销很低；真正的成本在**命中链接后的路径重算与穿越动画**。若与动态 carving 障碍同场（洞反复挖/填），链接路径可能被反复重算——见下方 Unity 已知问题。

**[COMMUNITY/UNITY BUG] 与 Carving 交互**：[Unity Issue Tracker](https://issuetracker.unity3d.com/issues/offmeshlink-does-not-calculate-the-path-correctly-after-carving-a-navmesh-at-runtime) 记录运行时 carving 后 OffMeshLink 路径计算错误是 Unity 已知问题（2017–2020 复现）——VRChat 中动态切换 carvable 障碍 + OffMeshLink 场景**需实测**。

## 性能数据（unityvrchat 或 [UNKNOWN]）

`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）` **无 OffMeshLink 独立帧时间数据** → **[UNKNOWN]**。

## 社区佐证

- Unity Issue Tracker「OffMeshLink does not calculate the path correctly after carving a NavMesh at runtime」：非 VRChat 专属但适用（[COMMUNITY/UNITY BUG]）；已复现于 2017–2020 编辑器版本，动态 carving 场景需谨慎。

## 相关页面

[[navmeshagent.md|NavMeshAgent]] · [[navmeshobstacle.md|NavMeshObstacle]] · [[../udon/ai-navigation.md|AI Navigation]] · [[../examples/ai-navigation.md|AI Navigation Example]] · [[../whitelisted-world-components.md|Allowlisted World Components]]

## 源清单

- Unity Manual Off-mesh Links: https://docs.unity3d.com/2022.3/Documentation/Manual/class-OffMeshLink.html
- Unity Issue Tracker（carving 后 off-mesh link）: https://issuetracker.unity3d.com/issues/offmeshlink-does-not-calculate-the-path-correctly-after-carving-a-navmesh-at-runtime
- VRChat Allowlist: https://creators.vrchat.com/worlds/whitelisted-world-components/
- 本地: `[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无数据）