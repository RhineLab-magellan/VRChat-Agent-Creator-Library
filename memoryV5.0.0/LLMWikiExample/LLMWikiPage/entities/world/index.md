---
title: World Domain Index
category: world
knowledge_level: applied
status: active
source: 本地知识库整理
source_type: community
version: 1.0
last_review: 2026-08-13
confidence: High
tags:
  - world
  - index
  - navigation
  - obsidian
aliases:
  - World Index
  - World Domain
  - World 知识入口
  - 世界域索引
related:
  - ../../index]]
  - ../../[[udon/index.md]]
  - ../../[[examples/index.md]]
  - ../../[[examples/README.md]]
  - ../../[[scene-components/index.md]]
  - ../../[[clientsim/index.md]]
  - ../../[[bakery/index.md]]
  - ../../[[shader/index.md]]
  - ../../[[patterns/index.md]]
  - "[[performance-guide.md]]"
  - "[[occlusion-culling-guide.md]]"
  - "[[reflection-probes.md]]"
  - "[[vrc-light-volumes.md]]"
  - "[[community-labs.md]]"
  - "[[creator-economy.md]]"
  - "[[vrc-camera-settings.md]]"
  - "[[vrc-quality-settings.md]]"
type: entity
created: 2026-07-14
sources: 本地知识库整理
updated: 2026-08-13
---
# World Domain Index

> World 域中层入口。用于把 [[index]] 与 World 下的 Udon、Networking、Persistence、Scene Components、Examples、性能、光照、Shader 等主题连接起来。

## 快速入口

| 主题 | 入口 | 说明 |
|---|---|---|
| Udon 总览 | [[udon/index.md]] | Udon、UdonSharp、Graph、事件、调试、运行时 |
| Networking | [[udon/networking/index.md]] | Variables、Events、Ownership、带宽、Late Joiner |
| Persistence | [[udon/persistence/index.md]] | PlayerData、PlayerObject、序列化、限制与实战模式 |
| Scene Components | [[scene-components/index.md]] | World 核心组件、Portal、ObjectSync、Station、Persistence + Unity 原生组件 24 个(Rigidbody/Collider/粒子/光照/音频/NavMesh/UI/视频) |
| Examples | [[examples/index.md]] | SDK 示例、Example Central、Persistence Examples |
| ClientSim | [[clientsim/index.md]] | 本地模拟、PlayerData/PlayerObject 调试、系统架构 |
| Bakery | [[bakery/index.md]] | 光照烘焙工具与流程 |
| World Shader | [[shader/index.md]] | Graphlit 等 World Shader 知识 |
| World Patterns | [[patterns/index.md]] | World 局部设计模式 |

## 核心单页

| 文档 | 主题 |
|---|---|
| [[performance-guide.md]] | 预算、材质、光照、测试方法 |
| [[occlusion-culling-guide.md]] | Occlusion Culling 性能优化 |
| [[reflection-probes.md]] | Reflection Probe 使用与限制 |
| [[vrc-light-volumes.md]] | 体素光照、Point Light Volumes |
| [[bakery/light-baking-guide.md#glim-lightmapper\|Glim Lightmapper]] | 独立 GPU 光照渲染器；Bakery 开源替代；VRChat 已确定可用 |
| [[community-labs.md]] | World 发布与审核路径 |
| [[creator-economy.md]] | 创作者经济 SDK 与产品集成 |
| [[vrc-camera-settings.md]] | 屏幕/手持相机、VR 双眼相关设置 |
| [[vrc-quality-settings.md]] | 阴影距离覆盖、质量设置限制 |
| [[items.md]] | Items / Props 在 World 中的边界 |
| [[layers.md]] | VRChat World Layer 分配与物理安全 |
| [[sdk-prefabs.md]] | SDK 内置 Prefab 速查 |
| [[supported-assets.md]] | 受支持脚本资产与组件 |
| [[whitelisted-world-components.md]] | World 组件白名单 |
| [[luraswitch2.md]] | 通用开关/滑块套件参考 |
| [[vvmw.md]] | 视频播放器参考 |
| [[audio-steam.md]] | ONSP 替换与音频系统迁移 |
| [[companions.md]] | Companion / Item 分裂与 World Creator 视角 |

## Udon 子域

| 子域 | 入口 |
|---|---|
| UdonSharp | [[udon/udonsharp/index.md]] |
| Graph | [[udon/graph/index.md]] |
| Players | [[udon/players/index.md]] |
| VRCGraphics | [[udon/vrc-graphics/index.md]] |
| VRCTween | [[udon/vrctween/index.md]] |
| MIDI | [[udon/midi/index.md]] |
| Video Players | [[udon/video-players/index.md]] |
| Data Containers | [[udon/data-containers/index.md]] |
| Networking Tools | [[udon/networking/tools/index.md]] |
| Persistence Patterns | [[udon/persistence/patterns/index.md]] |

## 维护说明

- 本文件是 World 域中层导航，不替代各子域 `index.md` 的详细说明。
- 新增 World 知识页后，应优先挂入对应子域 index；若属于高频入口或跨子域主题，再同步补充到本页。
- 本页使用路径型 Wikilink，避免 `［［index］］` 等同名文件歧义。
