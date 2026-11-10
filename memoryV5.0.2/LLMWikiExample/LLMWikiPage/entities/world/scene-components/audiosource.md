---
title: "AudioSource — 世界音频组件"
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
  - audio
  - spatial
  - pickup
aliases:
  - AudioSource
  - 音频源
  - 音频组件
related:
  - ../whitelisted-world-components.md
  - ../../api/audio.md
  - ../udon/players/player-audio.md
  - videoplayer.md
  - canvas.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/AudioSource-reference.html | https://docs.unity3d.com/2022.3/Documentation/ScriptReference/AudioSource.html | https://creators.vrchat.com/worlds/components/vrc_spatialaudiosource/ | https://creators.vrchat.com/worlds/whitelisted-world-components/"
updated: 2026-08-17
---
# AudioSource — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（音源 Audio Source 章节，含 64 源 / Avatar 3 源定量；世界侧数量阶梯无独立基准，详见性能小节）
> 本文聚焦 AudioSource **组件本身**（属性/用法）与 VRChat 空间化实践；Udon API 暴露（Play/Stop/volume 等）见 [[../../api/audio.md|Audio API]]。

---

## 核心功能（官方，[FACT]）

AudioSource 在 3D 场景中播放 `AudioClip`，需要场景中有 `AudioListener` 才能被听到；2D/3D 由 `Spatial Blend` 混合控制。核心能力与 API 均来自 Unity 官方：

- Unity Manual（AudioSource-reference）：在 3D 场景播放 AudioClip；2D/3D 通过 `Spatial Blend` 混合，[FACT] https://docs.unity3d.com/2022.3/Documentation/Manual/AudioSource-reference.html
- Unity Scripting API：`volume`、`pitch`、`loop`、`clip`、`Play/Pause/Stop`、`PlayOneShot`、`spatialBlend`（0=全 2D，1=全 3D）、`spatialize`、`spread`、`maxDistance`、`minDistance`、`rolloffMode`（Logarithmic/Linear/Custom），[FACT] https://docs.unity3d.com/2022.3/Documentation/ScriptReference/AudioSource.html

## 关键属性/行为

| 属性 | 说明（[FACT]） |
|---|---|
| **Spatial Blend** | 0.0 全 2D（不随距离/方向衰减），1.0 全 3D（downmix 到 mono 并按距离/方向衰减）；可用曲线在 0–1 间渐变。 |
| **3D Sound Settings** | Doppler Level、Spread、Min Distance、Max Distance、Volume Rolloff（Logarithmic/Linear/Custom 曲线）。 |
| **空间化插件（spatialize）** | 允许接入自定义 Spatializer（如 Oculus Spatializer / VRChat 自带空间化）。 |
| **晚期混响** | Listener 处于 AudioReverbZone 内时，Reverb 会自动施加到源上。 |

## VRChat 特定限制/注意

**[FACT] 白名单地位**：AudioSource 在官方 Allowlisted World Components「Unity Components (77)」内，World 可用，见 [[../whitelisted-world-components.md|Allowlisted World Components]]。

**[FACT] VRC_SpatialAudioSource 是音频标准**：官方强烈建议给每个有声的 AudioSource 加 `VRC_SpatialAudioSource` 组件做空间化——**Avatar 上不加会自动加默认值**，可能产生未预期的行为（官方：https://creators.vrchat.com/worlds/components/vrc_spatialaudiosource/）。

**[FACT] VRC_SpatialAudioSource 关键属性**：
- **Gain**：世界 0–24dB（默认 +10dB）；Avatar 限 ≤10dB。
- **Far**：默认 40m；Avatar 限 ≤40m。
- **Near**：建议 0；**Volumetric Radius**：建议 0 且 <Far。
- **Use AudioSource Volume Curve**、**Enable Spatialization**；默认 falloff 近似 inverse-square。

**[FACT] 与 Player 音频是不同系统**：玩家语音/Avatar 音频由 Player Audio API（`SetVoiceGain` 等）控制，不走 Unity AudioSource，见 [[../udon/players/player-audio.md|Player Audio]]。

**[INFERENCE] AudioLink 属第三方**：社区音频可视化方案（常见于世界音频），非白名单原生组件，不在本页范围。

**[FACT] 性能相关**：unityvrchat 基准显示 AudioSource 本身难以测出帧时间开销（即使 64 个源也测不到；VRChat 每 Avatar 限 3 个源）——详见性能小节。

## 性能数据（unityvrchat 或 [UNKNOWN]）

本地 `[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（音源 Audio Source 章节）：

- **[FACT]** 无法让 AudioSource 本身产生可测帧时间开销；Unity 音频源上限 **64**，VRChat 每个 Avatar 限 **3 个**。
- 世界侧 AudioSource 为 **[UNKNOWN]**：基准未单独测世界侧数量阶梯；VRChat 对世界音频源数量同样有隐式上限。
- 反例数据（邻近事实，非本体）：[AVPro/ProTV](https://protv.dev/guides/audio) 表明 **AudioSource 上的音频滤镜（lowpass/reverb）对 AVPro 播放器不生效**（渲染顺序问题），属 AVPro 特有坑，见 [[videoplayer.md|VideoPlayer]]。

## 社区佐证

- [ProTV Audio Configuration](https://protv.dev/guides/audio)（[COMMUNITY]/第三方文档）：AVPro speaker 相关的 AudioSource 滤镜失效问题。

## 相关页面

[[../../api/audio.md|Audio API]] · [[../udon/players/player-audio.md|Player Audio]] · [[../whitelisted-world-components.md|Allowlisted World Components]] · [[videoplayer.md|VideoPlayer]] · [[canvas.md|Canvas]]

## 源清单

- Unity Manual Audio Source: https://docs.unity3d.com/2022.3/Documentation/Manual/AudioSource-reference.html
- Unity Scripting API AudioSource: https://docs.unity3d.com/2022.3/Documentation/ScriptReference/AudioSource.html
- VRChat VRC Spatial Audio Source: https://creators.vrchat.com/worlds/components/vrc_spatialaudiosource/
- VRChat Allowlist: https://creators.vrchat.com/worlds/whitelisted-world-components/
- 本地: `[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（音源章节）
- 本地: [[../../api/audio.md|Audio API]]
- 本地: [[../udon/players/player-audio.md|Player Audio]]