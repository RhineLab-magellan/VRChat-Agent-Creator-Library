---
title: "VideoPlayer — 世界视频组件"
category: world
subcategory: scene-components
knowledge_level: applied
status: active
source: "Unity Manual 2022.3 + VRChat 官方文档"
source_type: official
version: 1.0
last_review: 2026-08-17
confidence: high
tags:
  - world
  - unity
  - video
  - quest
aliases:
  - VideoPlayer
  - 视频播放器
  - UnityVideoPlayer
related:
  - ../whitelisted-world-components.md
  - ../udon/video-players/index.md
  - ../../api/ui.md
  - audiosource.md
  - canvas.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-VideoPlayer.html | https://docs.unity3d.com/2022.3/Documentation/ScriptReference/Video.VideoPlayer.html | https://creators.vrchat.com/worlds/udon/video-players/ | https://creators.vrchat.com/worlds/udon/video-players/www-whitelist/ | https://creators.vrchat.com/worlds/whitelisted-world-components/"
updated: 2026-08-17
---
# VideoPlayer — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（未包含 VideoPlayer 数据，详见性能小节）
> VRChat 侧两套播放器（UnityVideo / AVPro）及 URL 白名单体系见 [[../udon/video-players/index.md|Video Players]]。

---

## 核心功能（官方，[FACT]）

VideoPlayer 在场景中播放视频，输出目标与播放控制均由 Unity 官方支持：

- Unity Manual（class-VideoPlayer）：播放视频到摄像机平面、RenderTexture、UI（RawImage）或材质（Material Override）；支持 `url`/`clip`、`isLooping`、`playbackSpeed`、`frame`、`renderMode`（CameraFarPlane / CameraNearPlane / RenderTexture / MaterialOverride / APIOnly）、`skipOnDrop` 等，[FACT] https://docs.unity3d.com/2022.3/Documentation/Manual/class-VideoPlayer.html
- Unity Scripting API：`Play/Pause/Stop`、`frame`、`frameCount`、`Prepare`、事件 `loopPointReached`、`prepareCompleted` 等，[FACT] https://docs.unity3d.com/2022.3/Documentation/ScriptReference/Video.VideoPlayer.html

## 关键属性/行为

| 属性/行为 | 说明（[FACT]） |
|---|---|
| **不支持直播流** | Unity VideoPlayer 不支持 YouTube Live / Twitch 等直播流（对比 AVPro）。 |
| **Editor 内可播放** | Unity Video 在 Play Mode 可直接播放 `.mp4`/`.webm` 直链；YouTube/Vimeo 等托管服务**只在客户端**工作。 |
| **音频模型** | UnityVideo 每个 speaker 对应一个**音轨**而非声道；大多数内容用单个立体声混音 AudioSource。 |

## VRChat 特定限制/注意

**[FACT] 白名单地位**：VideoPlayer 在官方 Allowlisted World Components「Unity Components (77)」内，World 可用，见 [[../whitelisted-world-components.md|Allowlisted World Components]]。

**[FACT] 两套视频播放器**（VRChat 官方提供 `VRCUnityVideoPlayer` 与 `VRCAVProVideoPlayer` 两个 prefab，见 [[../udon/video-players/index.md|Video Players]]）：
- **Unity Video Player**：VOD/直链简单场景，无直播。
- **AVPro**（MediaPlayer 等，白名单 AVPro 组 7 个）：支持 YouTube Live/Twitch 等直播流；但 **Editor 内不播放**，需 Build & Test。AVPro speaker 号称支持 8 声道，实际只支持 6 声道（5.1）。

**[FACT] 视频 URL 白名单**：播放器只能访问白名单内服务（YouTube/Vimeo/Twitch/SoundCloud 等）或用户启用「Allow Untrusted URLs」；`VRCUrl` 是唯一访问外部 URL 的方式（详见官方 www-whitelist：https://creators.vrchat.com/worlds/udon/video-players/www-whitelist/）。

**[FACT] Quest（Android）限制**：强制 HTTPS；默认不支持 YouTube/Twitch 直播；YTDL 已可用但稳定性问题；推荐 `.mp4`/`.webm` 直链。H.265 不支持、AV1 部分支持。

**[FACT] Master「Allow Untrusted URLs」**：SDK 示例播放器不处理该情况；需 Udon 将同步所有权转移给请求者。

**[COMMUNITY] AVPro 音频滤镜失效**：[ProTV](https://protv.dev/guides/audio) — AVPro speaker 上 lowpass/reverb 等音频滤镜不生效（见 [[audiosource.md|AudioSource]] 性能小节）。

## 性能数据（unityvrchat 或 [UNKNOWN]）

本地 `[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`：**未包含** VideoPlayer 性能数据 → **[UNKNOWN]**。

（视频性能主要受解码/网络/平台影响，官方无帧时间基准。相关邻近事实：AudioSource 上音频滤镜对 AVPro 播放器不生效会间接影响播放音频链路的可用性，非本体帧时间数据。）

## 社区佐证

- [VRChat Wiki: Video players](https://wiki.vrchat.com/wiki/Video_players)（[COMMUNITY]）：两套播放器 + 编解码史表，建议社区维护。
- [ProTV Audio](https://protv.dev/guides/audio)（[COMMUNITY]）：UnityVideo vs AVPro 音频处理与 speaker 配置。

## 相关页面

[[../udon/video-players/index.md|Video Players]] · [[../whitelisted-world-components.md|Allowlisted World Components]] · [[../../api/ui.md|UI API]] · [[audiosource.md|AudioSource]] · [[canvas.md|Canvas]]

## 源清单

- Unity Manual Video Player: https://docs.unity3d.com/2022.3/Documentation/Manual/class-VideoPlayer.html
- Unity Scripting API VideoPlayer: https://docs.unity3d.com/2022.3/Documentation/ScriptReference/Video.VideoPlayer.html
- VRChat Video Players: https://creators.vrchat.com/worlds/udon/video-players/
- VRChat www-whitelist: https://creators.vrchat.com/worlds/udon/video-players/www-whitelist/
- VRChat Allowlist: https://creators.vrchat.com/worlds/whitelisted-world-components/
- 本地: `[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无数据）
- 本地: [[../udon/video-players/index.md|Video Players 指数页]]（含 ecosystem-overview / www-whitelist）