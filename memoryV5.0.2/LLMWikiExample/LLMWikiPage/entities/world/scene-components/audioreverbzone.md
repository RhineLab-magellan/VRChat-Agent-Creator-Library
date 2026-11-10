---
title: "AudioReverbZone — 世界音频组件"
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
  - spatial
aliases:
  - AudioReverbZone
  - 混响区域
  - 音频混响区域
related:
  - ../whitelisted-world-components.md
  - ../../api/audio.md
  - audiosource.md
  - audiolowpassfilter.md
  - ../udon/players/player-audio.md
type: entity
created: 2026-08-17
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-AudioReverbZone.html | https://docs.unity3d.com/2022.3/Documentation/ScriptReference/AudioReverbZone.html | https://creators.vrchat.com/worlds/whitelisted-world-components/"
updated: 2026-08-17
---
# AudioReverbZone — VRChat 原生 Unity 组件

> 调研日期：2026-08-17 | 调研主体：VRChat 技术知识调研子代理（只读）| 对齐 Unity 2022.3 LTS
> 本地性能源：`[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无混响区域独立帧时间数据，详见性能小节）
> 本文聚焦 AudioReverbZone 组件本身（区域混响/区域过渡）；空间化体系与 Udon API 暴露见 [[audiosource.md|AudioSource]] 与 [[../../api/audio.md|Audio API]]。

---

## 核心功能（官方，[FACT]）

AudioReverbZone 是**基于位置的混响区域**：AudioListener 移入区域时，混响效果按距离渐变施加——`Max Distance` 外无效果，`Min Distance` 内效果完全施加。核心语义来自 Unity 官方：

- Unity Manual（Reverb Zones）：用于「进入洞穴/房间/水下」的氛围过渡，[FACT] https://docs.unity3d.com/2022.3/Documentation/Manual/class-AudioReverbZone.html
- Unity Scripting API：可混合多个 Reverb Zone 组合效果，[FACT] https://docs.unity3d.com/2022.3/Documentation/ScriptReference/AudioReverbZone.html
- 行为特征：作用于进入区域的 **AudioListener**，因此改变的是「谁在听」而非「源」——常用于脚步声、环境声的氛围变化。

## 关键属性/行为

| 属性 | 说明（[FACT]） |
|---|---|
| **reverbPreset** | 混响预设（AudioReverbPreset，如 Cave/City/Room），快速设定整体混响风格。 |
| **minDistance**（默认 10）/ **maxDistance**（默认 15） | 效果渐变半径：Min 内完全施加，Max 外无效果，其间线性/曲线过渡。 |
| **room / roomHF / roomLF** | 房间整体电平与高/低频段微调。 |
| **decayTime / decayHFRatio** | 混响衰减时长（秒）与高频衰减比例。 |
| **reflections / reflectionsDelay** | 早期反射电平与延迟。 |
| **reverb / reverbDelay** | 混响尾音电平与延迟。 |
| **HFReference / LFReference** | 高/低频参考频率（影响 roomHF/roomLF 的作用频段）。 |
| **diffusion / density** | 混响的扩散度与密度（塑造空间感细腻程度）。 |
| **行为** | 监听者（AudioListener）位置驱动；多区域可混合叠加。 |

## VRChat 特定限制/注意

**[FACT] 白名单地位**：AudioReverbZone 在官方 Allowlisted World Components「Unity Components (77) / Audio Filters」内，World 可用，见 [[../whitelisted-world-components.md|Allowlisted World Components]]。

**[FACT] 无 VRChat 官方专页**：VRChat 官方文档没有 AudioReverbZone 专属页面；统一由 Allowlist 与 `VRC_SpatialAudioSource` 音频体系覆盖。World 内可直接当 Unity 组件使用，通常配合世界环境氛围——空间化仍按 VRC_SpatialAudioSource 体系走（增益/Far/Near 等属性见 [[audiosource.md|AudioSource]]）。

**[COMMUNITY] 与 Avatar 音频的兼容性**：社区经验表明对回归/魔改音频源，空间混响行为不完全可预测；AVPro 播放器上的音频滤镜（含 reverb）不生效（见 [[audiosource.md|AudioSource]] 反例，[COMMUNITY]）。

**[INFERENCE] Quest/平台差异注意**：ReverbZone 依赖 AudioListener 位置，VRChat 的玩家 listener 在不同平台（PC/Quest/镜像）行为可能有差异，属于需实测项，本页无官方定量结论。

**[INFERENCE] 与玩家语音的关系**：玩家语音/Avatar 音频由 VRChat Player Audio 管线控制（`SetVoiceGain`/`SetVoiceLowpass` 等），不经过 Unity AudioSource 渲染——AudioReverbZone 依附 Unity AudioListener，对玩家语音的影响不保证可预期；语音处理走独立 API，见 [[../udon/players/player-audio.md|Player Audio]]。

## 性能数据（unityvrchat 或 [UNKNOWN]）

本地 `[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）` **未包含** AudioReverbZone 性能数据 → **[UNKNOWN]**。

相关邻近事实（[FACT]，上下文非本体）：VRChat 对音频源数量有限制（Unity 上限 64 源 / Avatar 每 Avatar 3 源，本体重测不到独立帧时间，见 [[audiosource.md|AudioSource]] 性能小节）；混响区域对 World 帧时间的影响无官方/基准数据。

## 社区佐证

- 无可靠专门社区基准；反例仅 AVPro 滤镜失效问题（[COMMUNITY]，见 [[audiosource.md|AudioSource]]）。

## 相关页面

[[audiosource.md|AudioSource]] · [[audiolowpassfilter.md|AudioLowPassFilter]] · [[../udon/players/player-audio.md|Player Audio]] · [[../whitelisted-world-components.md|Allowlisted World Components]] · [[../../api/audio.md|Audio API]]

## 源清单

- Unity Manual Reverb Zones: https://docs.unity3d.com/2022.3/Documentation/Manual/class-AudioReverbZone.html
- Unity Scripting API AudioReverbZone: https://docs.unity3d.com/2022.3/Documentation/ScriptReference/AudioReverbZone.html
- VRChat Allowlist: https://creators.vrchat.com/worlds/whitelisted-world-components/
- 本地: `[Unity/VRChat Performance Benchmarks](https://vrclibrary.com/wiki/books/unityvrchat-performance-benchmarks)（翻译源）`（无音频滤镜数据）