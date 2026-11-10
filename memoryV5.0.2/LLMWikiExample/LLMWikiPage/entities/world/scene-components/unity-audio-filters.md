---
title: "Unity 音频滤镜合并专页（5 个杂项滤镜）"
category: world
subcategory: scene-components
knowledge_level: applied
status: active
source: "Unity Manual 2022.3（5 个滤镜页）+ VRChat 官方白名单"
source_type: official
version: 1.0
last_review: 2026-08-19
confidence: high
tags:
  - world
  - unity
  - audio
  - audio-filter
aliases:
  - AudioChorusFilter
  - AudioDistortionFilter
  - AudioEchoFilter
  - AudioHighPassFilter
  - AudioReverbFilter
  - 音频滤镜合并页
related:
  - ../whitelisted-world-components.md
  - audiolowpassfilter.md
  - audiosource.md
  - audioreverbzone.md
  - ../../api/audio.md
  - ../udon/players/player-audio.md
type: entity
created: 2026-08-19
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-AudioChorusFilter.html | https://docs.unity3d.com/2022.3/Documentation/Manual/class-AudioDistortionFilter.html | https://docs.unity3d.com/2022.3/Documentation/Manual/class-AudioEchoFilter.html | https://docs.unity3d.com/2022.3/Documentation/Manual/class-AudioHighPassFilter.html | https://docs.unity3d.com/2022.3/Documentation/Manual/class-AudioReverbFilter.html | https://creators.vrchat.com/worlds/whitelisted-world-components/"
updated: 2026-08-19
---
# Unity 音频滤镜合并专页（AudioChorusFilter / AudioDistortionFilter / AudioEchoFilter / AudioHighPassFilter / AudioReverbFilter）

> **白名单地位**: 全部 5 个滤镜均在官方 [Allowlisted World Components](../whitelisted-world-components.md)「Unity Components (77) / Audio Filters」段内，**World 可用**。
> **本页为合并页**：AudioChorusFilter / AudioDistortionFilter / AudioEchoFilter / AudioHighPassFilter / AudioReverbFilter 各自无独立专页，统一收录于此。低通滤镜 **AudioLowPassFilter 已有专页**，见 [[audiolowpassfilter.md|AudioLowPassFilter]]，互链对照。
> **VRChat 专属限制**: 5 个滤镜 VRChat 侧均无专属专页 → **[FACT] 白名单可用** + **[UNKNOWN] 无 VRChat 侧限定**。

---

## 总览

上述 5 个滤镜均为 **Audio Filter**（挂在 AudioSource 或 AudioListener 上的逐源/逐监听音频 DSP 效果），语义来自 Unity Manual 2022.3 各页。低通相关（AudioLowPassFilter 的遮挡/距离衰减用法）不在本页展开，见 [[audiolowpassfilter.md|AudioLowPassFilter]]。

## 各滤镜核心参数对比（[FACT] Unity Manual 2022.3）

| 滤镜 | 用途 / 效果 | 核心参数 | 默认/范围要点 |
|---|---|---|---|
| **AudioChorusFilter** | 用正弦低频频振（LFO）调制原声，产生多个来源齐唱（合唱）效果；是 Flanger 的变体 | Dry Mix、Wet Mix 1/2/3、**Delay**、**Rate**、**Depth**、Feed Back | Delay 0.1–100ms（默认 40）；Rate 0–20Hz（默认 0.8）；Depth 默认 0.03 |
| **AudioDistortionFilter** | 失真效果，模拟低质量电台传输声 | **Distortion** | 0.0–1.0，默认 0.5 |
| **AudioEchoFilter** | 按 Delay 重复声音并按 Decay 衰减（回声/峡谷效果；也可模拟声光延迟） | **Delay**、**Decay Ratio**、**Wet Mix**、**Dry Mix** | Delay 10–5000ms（默认 500）；Decay Ratio 0–1（默认 0.5） |
| **AudioHighPassFilter** | 高通：让高频通过、切断低于 Cutoff 的频率 | **Cutoff Frequency**、**Highpass Resonance Q** | Cutoff 10–22000Hz（默认 5000）；Q 1–10（默认 1） |
| **AudioReverbFilter** | 对 AudioClip 做自定义混响（用 Reverb Preset，User 才可改参数） | **Reverb Preset**、Dry Level、Room/HF/LF、Decay Time、Decay HFRatio、Reflections/Reverb Level 等 | 大量 mB/秒单位参数，均预设默认值；仅 User 预设可编辑 |

> 对照｜低通 **AudioLowPassFilter**（另有专页，见 [[audiolowpassfilter.md|AudioLowPassFilter]]）：Cutoff 10–22000Hz（默认 5000）、Lowpass Resonance Q 1–10，用于遮挡/距离衰减。

## 关联组件

- **[[audiosource.md|AudioSource]]**：滤镜直接作用于单个 AudioSource，或作用于到达 AudioListener 的所有声音（挂 AudioListener 上）。
- **[[audioreverbzone.md|AudioReverbZone]]**：AudioReverbZone 是**基于空间区域**的混响（位置感知）；AudioReverbFilter 是**逐源/逐监听**的混响 DSP。两者均可做混响，但机制不同——本合并页的 Filter 是组件级效果，Zone 是空间驱动。
- **[[../../api/audio.md|Audio API]]**：World 侧音频体系（VRC_SpatialAudioSource / 音量控制等）的对照入口。
- **[[../udon/players/player-audio.md|Player Audio]]**：玩家语音（含 SetVoiceLowpass）走独立 API，与这些滤镜无关。

## VRChat 特定注意

- **[FACT] 白名单地位**：5 个滤镜均在官方白名单「Audio Filters」段内，World 可用。
- **[FACT] 无 VRChat 官方专页**：官方文档无针对这些滤镜的 World 侧专属说明。
- **[UNKNOWN]**：VRChat 侧对这类逐源音频 DSP 滤镜是否有限定（数量/性能/平台差异）——**官方未定论**，本页不臆测。
- **[COMMUNITY] AVPro 滤镜失效坑**：与 [[audiolowpassfilter.md|AudioLowPassFilter]] 专页一致——第三方面板报告的 AVPro speaker 上音频滤镜（含这些）常不生效（渲染顺序问题），视频播放器音频不应依赖本类滤镜（见 [[../udon/video-players/index.md|Video Players]]）。
- **性能定性（[INFERENCE]）**：音频 DSP 滤镜为逐源处理，成本随激活源数量线性增长；但无官方/基准定量，不得据此估算具体开销。

## 相关页面

[[audiolowpassfilter.md|AudioLowPassFilter]] · [[audiosource.md|AudioSource]] · [[audioreverbzone.md|AudioReverbZone]] · [[../../api/audio.md|Audio API]] · [[../udon/players/player-audio.md|Player Audio]] · [[../whitelisted-world-components.md|Allowlisted World Components]] · [[index.md|Scene Components 总览]]

## 源清单

- Unity Manual Audio Chorus Filter: https://docs.unity3d.com/2022.3/Documentation/Manual/class-AudioChorusFilter.html
- Unity Manual Audio Distortion Filter: https://docs.unity3d.com/2022.3/Documentation/Manual/class-AudioDistortionFilter.html
- Unity Manual Audio Echo Filter: https://docs.unity3d.com/2022.3/Documentation/Manual/class-AudioEchoFilter.html
- Unity Manual Audio High Pass Filter: https://docs.unity3d.com/2022.3/Documentation/Manual/class-AudioHighPassFilter.html
- Unity Manual Audio Reverb Filter: https://docs.unity3d.com/2022.3/Documentation/Manual/class-AudioReverbFilter.html
- VRChat Allowlist: https://creators.vrchat.com/worlds/whitelisted-world-components/
