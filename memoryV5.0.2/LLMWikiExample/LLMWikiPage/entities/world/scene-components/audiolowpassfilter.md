---
title: "AudioLowPassFilter — 世界音频组件"
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
  - audio-filter
  - occlusion
aliases:
  - AudioLowPassFilter
  - 低通滤波器
  - 音频低通滤波
related:
  - ../whitelisted-world-components.md
  - ../../api/audio.md
  - audiosource.md
  - audioreverbzone.md
  - ../udon/video-players/index.md
  - ../udon/players/player-audio.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-AudioLowPassFilter.html | https://docs.unity3d.com/ScriptReference/AudioLowPassFilter.html | https://creators.vrchat.com/worlds/whitelisted-world-components/"
updated: 2026-08-17
---
# AudioLowPassFilter — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无 AudioLowPassFilter 独立帧时间数据，详见性能小节）
> 本文聚焦 AudioLowPassFilter 组件本身（低通滤波/距离衰减模拟）；AVPro 视频播放器滤镜失效例外见「VRChat 特定限制」，语音低通独立 API 对照见 [[../udon/players/player-audio.md|Player Audio]]。

---

## 核心功能（官方，[FACT]）

AudioLowPassFilter 是低通滤波器——**通过低频、衰减高于 `Cutoff Frequency` 的高频**。可作用于单个 AudioSource 或所有到达 AudioListener 的声音。核心语义来自 Unity 官方：

- Unity Manual（Audio Low Pass Filter）：典型用途是**遮挡音效**（门后的声音高频被滤除）、配合视觉雾效、模拟距离感；官方示例「开门时改变 Cutoff Frequency 模拟门后声音」，[FACT] https://docs.unity3d.com/2022.3/Documentation/Manual/class-AudioLowPassFilter.html
- Unity Scripting API：`cutoffFrequency`（10–22000Hz，默认 5000）、`lowpassResonanceQ`（1–10，默认 1）、`customCutoffCurve`（按距离改变截止频率），[FACT] https://docs.unity3d.com/ScriptReference/AudioLowPassFilter.html

## 关键属性/行为

| 属性 | 说明（[FACT]） |
|---|---|
| **cutoffFrequency** | 截止频率（10–22000Hz，默认 5000）；越低衰减越多高频，-3dB 点为截止频率。**设置该值会清空自定义曲线**。 |
| **lowpassResonanceQ** | 共振 Q 值（1–10，默认 1），影响截止点附近的增益峰。 |
| **customCutoffCurve / Rolloff 曲线** | 可定义 Cutoff Frequency 随 AudioSource↔AudioListener 距离变化——距离越远截止频率越低，模拟衰减与遮挡。 |
| **挂载位置** | 挂在 AudioSource 或 AudioListener 上皆可（同 GameObject 需有对应组件）。 |

## VRChat 特定限制/注意

**[FACT] 白名单地位**：AudioLowPassFilter 在官方 Allowlisted World Components「Unity Components (77) / Audio Filters」内，World 可用；同属 Audio Filters 段的还有 AudioChorusFilter / AudioDistortionFilter / AudioEchoFilter / AudioHighPassFilter / AudioReverbFilter / **AudioReverbZone / AudioSource**（共 8 项），见 [[../whitelisted-world-components.md|Allowlisted World Components]]。

**[FACT] 无 VRChat 官方专页**：无专属文档；由 Allowlist + 音频体系覆盖。World 内可直接当 Unity 组件使用，常见于世界遮挡音效设计。

**[FACT] 遮挡音效用法（官方用途）**：Unity 官方明确为「模拟门/墙后高频衰减」；VRChat World 内可直接用（挂在 AudioSource 或 AudioListener 上），如门缝/屏障遮挡感——开门、开屏障时提高 Cutoff Frequency 还原清晰声音。

**[FACT] Player 语音 Lowpass 是独立 API**：VRChat 玩家语音的低通是 `SetVoiceLowpass`（见 [[../udon/players/player-audio.md|Player Audio]]），与 Unity AudioLowPassFilter 无关——语音遮挡需求不要挂此组件，走 Player Audio API。

**[COMMUNITY] AVPro 视频播放器滤镜失效坑**：AVPro speaker 上的音频滤镜（含 low-pass）不生效（渲染顺序问题）——视频播放器音频若需低通/遮挡效果，不能依赖挂在此类滤镜上（见 [[audiosource.md|AudioSource]] 反例、[[../udon/video-players/index.md|Video Players]] 相关章节，[COMMUNITY] 源：ProTV Audio 文档）。

## 性能数据（unityvrchat 或 [UNKNOWN]）

本地 `[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）` **未包含** AudioLowPassFilter 性能数据 → **[UNKNOWN]**。

相关上下文（[FACT]，非本体数据）：unityvrchat 基准显示 AudioSource 本身难以测出帧时间开销（即使 64 个源也测不到；每 Avatar 限 3 源，见 [[audiosource.md|AudioSource]] 性能小节）；音频 DSP 滤镜（lowpass）为逐源处理、成本随激活源数量线性增长，属定性判断，无官方/基准定量 —— 不得据此估算具体开销。

## 社区佐证

- [ProTV Audio](https://protv.dev/guides/audio)（[COMMUNITY]/第三方文档）：AVPro 上 lowpass 等滤镜不生效。

## 相关页面

[[audiosource.md|AudioSource]] · [[audioreverbzone.md|AudioReverbZone]] · [[../udon/video-players/index.md|Video Players]] · [[../udon/players/player-audio.md|Player Audio]] · [[../whitelisted-world-components.md|Allowlisted World Components]] · [[../../api/audio.md|Audio API]]

## 源清单

- Unity Manual Audio Low Pass Filter: https://docs.unity3d.com/2022.3/Documentation/Manual/class-AudioLowPassFilter.html
- Unity Scripting API AudioLowPassFilter: https://docs.unity3d.com/ScriptReference/AudioLowPassFilter.html
- VRChat Allowlist: https://creators.vrchat.com/worlds/whitelisted-world-components/
- 本地: `[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无数据）
- 本地: [[../udon/players/player-audio.md|Player Audio]]（语音 Lowpass 对照）