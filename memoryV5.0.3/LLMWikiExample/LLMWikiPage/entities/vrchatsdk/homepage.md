---
title: VRChat.community API 首页
category: vrchatsdk

knowledge_level: core
status: active

tags:
  - vrchatsdk
  - api
  - http
  - websocket
  - sdk

aliases:
  - VRChatSDK
  - VRChat SDK
  - HTTP API
  - SDK 首页
  - "01_首页"

related:
  - typescript-sdk.md
  - websocket-api.md
  - instances.md
  - ../../concepts/hybrid/osc-protocol.md

source: VRChat.community 文档首页 + VRChat packages releases
source_type: community
version: 1.1
upstream_sdk_version: 3.10.6 stable (2026-10-30 16:24 UTC) / 3.10.5 (2026-09-04) / 3.10.4 (2026-06-17)
upstream_client_version: 2026.2.3p3 Live (2026-07-14, Build 1867) / 2026.3.1 Open Beta (2026-07-22, Build 1878; 2026-07-24, Build 1879) / 2026.4.1（release note 3.10.6 引用的 client 行为变更版本）
last_review: 2026-11-10
confidence: High
type: entity
created: 2026-07-26
sources: VRChat.community 文档首页 + VRChat packages releases + creators.vrchat.com release note 3.10.4/3.10.5/3.10.6
updated: 2026-11-10
---
# VRChat.community API 首页

> **当前 SDK 版本**: **3.10.6 stable** (2026-10-30 16:24 UTC，GitHub release `3.10.6`，非 prerelease)
> 上一 stable 线: **3.10.5** (2026-09-04 17:06 UTC) → **3.10.4** (2026-06-17) → 3.10.3 (2026-04-17)
> **当前客户端版本**: **2026.2.3p3 Live** (2026-07-14, Build 1867) | **2026.3.1 Open Beta** (2026-07-22, Build 1878; 2026-07-24, Build 1879)
> 仓库: https://github.com/vrchat/packages/releases
> ⚠️ **3.10.6 仍基于 Unity 2022.3** —— release note 顶部原文警告 "Do NOT upgrade your projects to Unity 6!"

A community-driven project that provides documentation, tools, and resources for developers working with the VRChat API. This project fills the gap left by VRChat's lack of API documentation, tooling and support.

> ⚠️ **来源性质说明**: 本页描述的是 **VRChat.community 社区 API 文档首页**，不是 VRChat Creator Docs 的官方入口。与 [[entities/vrchatsdk/index]] 目录中的接口整理一起使用时，应明确其为**社区维护 API 文档**。

---

## SDK 3.10.4 → 3.10.6 基线变化（2026-11-10 核证）

> 证据源：https://creators.vrchat.com/releases/release-3-10-4/ 、 https://creators.vrchat.com/releases/release-3-10-5/ 、 https://creators.vrchat.com/releases/release-3-10-6/ ；发布时刻取自 GitHub release API（`vrchat/packages`）。

| 版本 | 发布 | 与本 KB 直接相关的增量 |
|---|---|---|
| 3.10.4 | 2026-06-17 | **VRCTween / VRCTweenHandle**（DOTween 封装）→ [[entities/api/vrctween]]；**Contacts Box shape**；World 侧 **VRCPhysBoneCollider 暴露给 Udon**（改后必调 `ApplyConfigurationChanges()`）；DataList/DataDictionary 自定义 capacity + `EnsureCapacity()`；`UdonSharpBehaviour` 实现 `IUdonEventReceiver`；全 SDK 项目强制 shader chunk 4MB |
| 3.10.5 | 2026-09-04 | `VRCQualitySettings.RealtimeReflectionProbes` / `ShadowmaskMode` 可从 Udon 读写；U# **Assembly Version Defines**；`Assets/` 与 `Packages/` 均可再建 U# 脚本；ClientSim **Play→Edit 不再触发 `OnDisable` 等 Unity 事件**；PhysBone "Reset When Disabled" 也重置 animator 参数与 Udon state；VRCJson 嵌套花括号 / 字符串内括号修复；DataToken int→long 修复；Add Components 菜单合并为 "VRChat" 分类 |
| 3.10.6 | 2026-10-30 | **Constraint source API + SmoothingFactor**（见 [[entities/api/dynamics]]）；U# **enum 比较**与**命名参数跳过可选参数**两处编译器修复；sync null VRCUrl + persistence 异常修复；U# `Interact()` 新增 **Outline Renderers**（**无 Udon accessor**）；新组件 VRCBillboard / VRCVelocityTracker / VRCPlacementZone |

> ⚠️ **不要跨版本推广**：3.10.4 的 World SDK 集成 bug 在 3.10.4-beta.2 才修；3.10.6 的 Advanced Parameters / Material Contacts 在 beta 期**只对 Proximity 类型生效**，beta.3 才放开——引用 beta 文档时必须标注版本。

---

Last updated on

VRChat's API is not officially supported or documented by VRChat. This documentation project is maintained on a best-effort basis by the community and attempts to smooth over API breakage by quickly updating when endpoints change. Use responsibly and be aware that endpoints may still break without notice.

Abuse of the API may result in account termination. For their official stance, refer to VRChat's Creator Guidelines.

## Resources

### Communities

### Discord

Join the VRChat.community server to discuss development, ask questions, and get help from the community.

### VRChat Group

Join the VRChat group for events & hangouts with like-minded creators.

### Documentation

### API Reference

Complete documentation of all VRChat API endpoints with examples and schemas.

### WebSocket API

Real-time event streaming documentation for VRChat's pipeline system.

### FAQ

Frequently asked questions and common issues with solutions.

### SDKs & Libraries

Official SDKs are available for multiple programming languages:

### JavaScript

### Python

### .NET

### Dart

### Java

### Rust

### Contributing

This project is open-source and is always looking for contributors. Whether it's fixing documentation errors, adding new endpoints, or improving SDKs, your help is appreciated!

Learn how to contribute by visiting our contributing guide.

Frequently asked questionsCommon questions and answers about the VRChat API, including authentication, rate limits, unofficial status, and best practices for developers building VRChat integrations.
---

## 相关页面

[[typescript-sdk.md]] · [[websocket-api.md]] · [[instances.md]] · [[../../concepts/hybrid/osc-protocol.md]]
