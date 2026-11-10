---
title: "Video Players Ecosystem Overview"
category: world
knowledge_level: applied
status: active
source: video-player.md + 本地知识库整理
source_type: community
version: 1.0
last_review: 2026-07-13
confidence: Medium
tags:
  - world
  - video
  - player
  - ecosystem
aliases:
  - "Video Players Ecosystem Overview"
  - "视频播放器生态总览"
  - "播放器生态"
  - ecosystem-overview.md
related:
  - ../../vvmw.md
  - ../../examples/udon-example-scene/udon-video-sync-player.md
  - index.md
type: entity
created: 2026-07-13
sources: video-player.md + 本地知识库整理
updated: 2026-07-13
---

# Video Players Ecosystem Overview

> 本页用于承载“播放器生态导航 / 总览 / 选型”层知识；不要把它与单一工具深度文档或官方示例机制解析混写。

## 1. 为什么需要生态总览
- `vvmw.md` 适合承载 VizVid / VVMW 的深度专题。
- `udon-video-sync-player.md` 适合承载官方示例播放器的同步机制解析。
- 但创作者在真实选型时，往往先问的是：
  - 我需要哪一类播放器？
  - 我要装官方基础版，还是更完整的第三方方案？
  - 某个工具是“视频播放器”还是“幻灯片 / 图集工具”？

## 2. 粗分类
| 类别 | 代表 | 适用场景 |
|---|---|---|
| **官方基础 / 示例型** | Udon Video Sync Player | 先理解官方示例、基础视频同步结构 |
| **通用多功能型** | VizVid / VVMW, ProTV, YamaPlayer | 面向更完整的世界视频播放需求 |
| **幻灯片 / 图集偏向型** | Sliden | 偏图片、展示、页式内容场景 |

## 3. 使用边界提醒
- 不要把 **生态总览文** 直接塞进单一工具深度文档。
- `Sliden` 这类工具更偏幻灯片 / 图集场景，不应简单等同于通用视频播放器。
- 安装入口、第三方仓库状态、预发行包等信息时效性强，长期 memory 中应只保留高层导航，而不保留细碎分发态细节。

## 4. 后续拆分方向
- `protv.md`
- `sliden.md`
- `yama-player.md`

这些文件若后续建立，应定位为 **轻量工具卡片**，而不是与 `vvmw.md` 同等深度的长档案。

---

## 5. 官方后端组件对照（暴露树 Video 域）

VRChat 官方 SDK 提供两种视频播放器组件（均为 Udon 暴露类型，暴露树 `Video` 命名空间）：

| 组件 | 类名 | 直播流 | 编辑器播放 | 备注 |
|------|------|:--:|:--:|------|
| **AVPro 播放器** | `VRCAVProVideoPlayer` | ✅（YouTube Live/Twitch 等） | ❌ 需 Build & Test | 多平台直播；**PCVR 支持 EAC3 7.1 音频**（8 通道说法不准确） |
| **Unity 播放器** | `VRCUnityVideoPlayer` | ❌ | ✅ 直连 mp4/webm 可播 | YouTube/Vimeo 托管链接仅客户端可播 |

两者共同基类 `BaseVRCVideoPlayer`；`VRCUrl`、`VRCUrlInputField` 配合使用。

### 选型要点（官方 FACT）

- **需要直播** → AVPro；**简单 VOD/稳定** → Unity VideoPlayer。
- **5 秒速率限制**: 同一用户仅每 5 秒可处理一个新视频 URL，且为**全局限制**（跨所有播放器）。多播放器同时播放时，late-joiner 会同时发多次请求而失败，须自行错峰。
- **循环**: Prefab 默认关闭循环（为同步）——要循环需开启 Loop 并移除 UdonBehaviour。
- **不强制用 UdonSyncPlayer**: 无需同步时可直接用视频组件；官方 prefab（`UdonSyncPlayer (AVPro/Unity)`）位于 `Packages/VRChat SDK - Worlds/Samples/UdonExampleScene/Prefabs/VideoPlayers`。
- SDK 官方示例 `UdonSyncPlayer (Unity).prefab` 在某些场景更稳定（[[../../sdk-prefabs]] §6）。

> 来源: 官方 [Video Players](https://creators.vrchat.com/worlds/udon/video-players/)。

---

## 相关页面

[[../../vvmw.md]] · [[../../examples/udon-example-scene/udon-video-sync-player.md]] · [[index.md]]
