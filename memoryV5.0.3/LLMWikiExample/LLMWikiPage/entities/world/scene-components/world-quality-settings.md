---
title: "World Quality Settings"
category: world
subcategory: scene-components
knowledge_level: applied
status: active
source: https://creators.vrchat.com/releases/release-3-10-5/
source_type: official
version: 1.0
sdk_version: "3.10.5"
last_review: 2026-11-10
confidence: Medium
tags:
  - world
  - component
  - lighting
  - shadow
  - quality
aliases:
  - "World Quality Settings"
  - "WorldQualitySettings"
  - world-quality-settings.md
related:
  - ../vrc-quality-settings.md
  - vrc-scenedescriptor.md
  - ../bakery/light-baking-guide.md
  - ../reflection-probes.md
type: entity
created: 2026-11-10
sources: https://creators.vrchat.com/worlds/udon/vrc-graphics/vrc-quality-settings/ | https://creators.vrchat.com/releases/release-3-10-5/
updated: 2026-11-10
---
# World Quality Settings

> **场景组件**（Inspector 编辑、随场景序列化）。
> 主源: [Release 3.10.5](https://creators.vrchat.com/releases/release-3-10-5/)
> 组件三项配置的出处: [官方 · VRCQualitySettings 页](https://creators.vrchat.com/worlds/udon/vrc-graphics/vrc-quality-settings/)（页面 Last updated 2026-06-10）的 **World Quality Settings Script** 小节
> ⚠️ **命名说明**: 3.10.5 release note **没有**直接出现 `WorldQualitySettings` 这个组件名；「组件在新 SDK 中 + 自定义 Inspector」的表述来自团队审计内检，本库按发布描述命名。引用时请以 Inspector 实际显示名为准。
> **本页范围**: 只写 **World Quality Settings 场景组件 + Inspector + 序列化值语义**。
> Udon 侧 `VRCQualitySettings` 的只读属性 / `SetShadowDistance()` / 事件归 **API 域**，见 [[entities/world/vrc-quality-settings]]——两者是**两层不同的东西**，别混。

---

## 概述

`VRCWorld` prefab 自带一个 **World Quality Settings** 脚本，让世界作者在**不写代码**的前提下配置三项 Unity 质量设置：

| Inspector 项 | 对应 Unity 设置 | 说明 |
|---|---|---|
| **Shadow Distance** | `QualitySettings.shadowDistance` | 阴影渲染距离（米）。官方在 Udon 侧会 clamp 到 0.1–10000.0；组件侧同语义 |
| **Shadowmask Lighting Mode** | `QualitySettings.shadowmaskMode` | 阴影遮罩模式（Distance / Shadowmask），混合实时与烘焙阴影 |
| **Realtime Probes** | `QualitySettings.realtimeReflectionProbes` | 是否启用实时反射探针 |

> **FACT**（官方文档原文）: `The VRCWorld prefab includes the 'World Quality Settings' script. It allows you to easily configure the following Unity settings: Shadow Distance / Shadowmask Lighting Mode / Realtime Probes.`

---

## 与 Udon 侧 `VRCQualitySettings` 的分层

| 层 | 位置 | 谁写 | 生效时机 |
|---|---|---|---|
| **World Quality Settings 组件** | 场景 / prefab，Inspector 序列化 | 世界作者（编辑器） | 世界加载时 |
| **`VRCQualitySettings` Udon API** | 运行时脚本 | 世界作者（Udon） | 运行时调用（并随世界加载重置） |

- 组件决定**世界的默认值**；Udon API 决定**运行时怎么改**（例如进入大区域时扩大阴影距离）。
- 两者操作的是同一批底层设置，所以 Udon 侧覆盖会盖掉组件值；`Reset*` 类调用回到「用户配置」，而不是「组件里写的值」——注意这个差别。

> SDK **3.10.5** 起 Udon 侧新增了 `VRCQualitySettings.RealtimeReflectionProbes` 与 `VRCQualitySettings.ShadowmaskMode` 的读写入口（[Release 3.10.5](https://creators.vrchat.com/releases/release-3-10-5/)）。这一半归 API 域维护，本页不展开。

---

## 现场行为（团队审计内检，未在本库复测）

> ⚠️ 以下三条来自团队对 SDK 的内检记录，**未经本库独立实测**；当作「待验证的工作假设」，不要当官方 FACT 引用：

1. **启动时总会写一次 Shadowmask Mode**：即使你在 Inspector 里没有动过该字段，脚本在世界启动阶段也会把 Shadowmask Lighting Mode 写入 `QualitySettings`。
   - 后果：任何「玩家/平台侧」对 shadowmask 的既有状态都会被世界值覆盖。
   - 想保留「不干预」，需要在 Inspector 里确认取值与 Unity 项目设置一致。
2. **探针与阴影距离是分别 override 的**：三项各自独立写入，不存在「关掉一项就整块不生效」的联动。
3. **升级 SDK 包 ≠ 现有场景自动获得该组件**：组件是**场景内序列化**的，老场景在 Inspector 里可能**没有**这一项（或仍是旧序列化值）。
   - 改场景前先在 Inspector 里**看一眼实际序列化的值**，不要按新包默认值推断。
   - 升级后若发现阴影距离/探针行为与预期不符，优先怀疑「场景里根本没有这个组件实例」。

---

## 使用建议

| 目标 | 做法 |
|---|---|
| 大场景需要长阴影距离 | 组件里设 Shadow Distance；跨平台差异交给 Udon 的 `SetShadowDistance(low, medium, high, mobile)` |
| 混合实时/烘焙阴影 | 组件里设 Shadowmask Lighting Mode = Shadowmask（PC）；移动端能力有限，需实测 |
| 需要实时反射的地面/水面 | 组件里开 Realtime Probes；性能开销用反射探针数量与分辨率控制 |
| 不想影响玩家偏好 | 尽量只设必要的项；Udon 侧用完后 `ResetShadowDistance()` 归还用户配置 |

---

## 常见坑

1. **把「组件」和「Udon API」当成一个东西** → 一个在场景里序列化、一个是运行时调用；改错层会导致「打包后设置没生效」。
2. **升级 SDK 后以为组件会自动出现** → 老场景需手动确认/添加。
3. **Shadowmask Mode 被静默写入** → 与 Unity 项目设置不一致时表现差异（尤其烘焙阴影接缝），排查时先查这个组件。
4. **Reflection Probes 全开** → 实时反射探针开销大，移动端尤其敏感；按需开、按场景关。

---

## 相关文档

- [[entities/world/vrc-quality-settings]] — **Udon 侧** `VRCQualitySettings` API（只读属性 / 阴影距离 / 事件）
- [[entities/world/scene-components/vrc-scenedescriptor]] — `VRC_SceneDescriptor`（每个世界必含）
- [[entities/world/reflection-probes]] — 反射探针实践
- [[entities/world/bakery/light-baking-guide]] — 烘焙阴影与 shadowmask 的关系
- [官方文档 · VRCQualitySettings](https://creators.vrchat.com/worlds/udon/vrc-graphics/vrc-quality-settings/)
- [Release 3.10.5](https://creators.vrchat.com/releases/release-3-10-5/)