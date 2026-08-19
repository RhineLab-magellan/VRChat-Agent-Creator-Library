---
title: "PlayableDirector — 世界 Timeline 播放组件"
category: world
subcategory: scene-components
knowledge_level: applied
status: active
source: "Unity Manual (Timeline/PlayableDirector) + VRChat 官方白名单"
source_type: official
version: 1.0
last_review: 2026-08-19
confidence: medium
tags:
  - world
  - unity
  - animation
aliases:
  - PlayableDirector
  - Timeline 播放器
related:
  - ../whitelisted-world-components.md
  - ../../api/animator.md
  - ../../api/audio.md
type: entity
created: 2026-08-19
sources: "https://docs.unity3d.com/2019.1/Documentation/Manual/class-PlayableDirector.html | https://creators.vrchat.com/worlds/whitelisted-world-components/"
updated: 2026-08-19
---
# PlayableDirector — VRChat 原生 Unity Timeline 播放组件

> **白名单地位**: Unity `PlayableDirector` 在官方 [Allowlisted World Components](../whitelisted-world-components.md)「Unity Components (77)」内**单独一段**（PlayableDirector），**World 可用**。
> **VRChat 专属限制**: VRChat 无 PlayableDirector/Timeline 专属专页 → VRChat 内 Timeline 支持边界见「VRChat 特定注意」的 [UNKNOWN] 标注。

---

## 核心功能（官方，[FACT]）

`PlayableDirector`（`UnityEngine.PlayableDirector`）保存 **Timeline 资产**与 **场景 GameObject** 的绑定（Timeline instance），并控制 Timeline instance 何时播放、如何更新时钟、播放结束后的行为。官方语义来自 Unity Manual（Timeline 章节，白名单链接到的 2019.1 版本）：[FACT] https://docs.unity3d.com/2019.1/Documentation/Manual/class-PlayableDirector.html

- **Playable**：手动把 Timeline Asset 关联到场景 GameObject，形成 Timeline instance。
- **Update Method（时钟源，[FACT]）**：**DSP**（音频采样精确调度）、**Game Time**（受 time scaling 影响）、**Unscaled Game Time**（不受缩放）、**Manual**（脚本手动设置时钟）。
- **Play on Awake / Wrap Mode / Initial Time / Current Time**：是否开局播放；结尾行为 **Hold / Loop / None**（None=播放后重置动画属性为播放前值）；起始时间偏移与当前时间查看。
- **Bindings**：把场景 GameObject 绑定到 Timeline 资产的轨道。

## 与动画体系的关系

- 核心驱动 Timeline 轨道（动画/音频/激活/控制等轨道）播放。动画轨道受 **Animator** 系统约束，可链 [[../../api/animator.md|Animator API]] 与 [[../../avatar/animator-system.md|Animator 体系]]。
- Timeline 中可包含 **Animation Track（Animator）+ Audio Track（配合 [[../../api/audio.md|Audio API]]/AudioSource）** 的多轨组合；World 侧常用来做机关、演出、过场序列。

## VRChat 特定注意

- **[FACT] 白名单地位**：PlayableDirector 在官方白名单单独列出，World 可用。
- **[FACT] 无 VRChat 官方专页**：官方文档没有为 PlayableDirector/Timeline 提供 World 侧专属说明。
- **[UNKNOWN]**：VRChat World 内 **Timeline 支持的边界**——是否完整支持全部轨道类型（尤其动画轨道对 World Animator/动画系统的驱动）、多客户端同步行为、玩家在 Timeline 上的播放状态——**官方未定论**，本页不臆测，须实测复核。
- **性能定性（[INFERENCE]）**：Director 播放驱动轨道逐帧求值，涉及 Animator 与音频时开销随轨道数与激活时长增长；但无官方/基准定量，不得据此估算具体开销。
- **时钟源注意（[FACT]）**：VRChat World 的 time scaling 行为未专门对 PlayableDirector 说明；若涉及精确音画同步，时钟源选择（DSP/Unscaled）属 Unity 默认语义。

## 相关页面

[[../../api/animator.md|Animator API]] · [[../../api/audio.md|Audio API]] · [[../../avatar/animator-system.md|Animator 体系]] · [[../whitelisted-world-components.md|Allowlisted World Components]] · [[index.md|Scene Components 总览]]

## 源清单

- Unity Manual Playable Director component: https://docs.unity3d.com/2019.1/Documentation/Manual/class-PlayableDirector.html
- VRChat Allowlist: https://creators.vrchat.com/worlds/whitelisted-world-components/
