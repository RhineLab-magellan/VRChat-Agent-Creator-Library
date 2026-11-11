---
title: "VRChat Advanced APIs"
category: api
knowledge_level: core
status: active
source: "VRChat 官方文档 (creators.vrchat.com/worlds/udon/) + release note 3.10.5 + 官方 VRCQualitySettings 页"
source_type: official
version: 1.1
last_review: 2026-11-10
confidence: High
tags:
  - api
  - shader
  - light
  - avatar
aliases:
  - "VRChat Advanced APIs"
  - vrchat-advanced-apis.md
related:
  - animator.md
  - api-checker.md
  - audio.md
  - data-containers.md
  - dynamics.md
type: entity
created: 2026-06-20
sources: "VRChat 官方文档 (creators.vrchat.com/worlds/udon/) + release note 3.10.5 + 官方 VRCQualitySettings 页"
updated: 2026-11-10
---
# API: VRChat Advanced APIs

> SDK Version: VRChat SDK 3.x（VRCQualitySettings 增量核证至 **3.10.5+**）
> Last Verified: 2026-11-10

---

## 1. VRCGraphics（图形 API）

### 暴露的函数

| 函数 | 说明 | 热路径 |
|---|---|---|
| `VRCGraphics.Blit(src, dst, mat)` | 使用着色器将源纹理复制到目标 RenderTexture | ❌ |
| `VRCGraphics.DrawMeshInstanced()` | GPU instancing 多次绘制相同网格 | ❌ |
| `VRCShader.PropertyToID(name)` | 获取着色器属性名的 ID（初始化时调用一次，可复用） | ✅ (缓存后) |
| `VRCShader.SetGlobal(propId, value)` | 使用 ID 设置全局着色器值，**在所有着色器可用，包括 Avatar** | ⚠️ |

### 重要说明
- `VRCShader.SetGlobal` 设置的值在 Avatar 着色器中也可见
- `Blit` 不能传 null 目标
- `PropertyToID` 应缓存，避免重复调用

---

## 2. VRCQualitySettings（画质设置）

> 官方页：https://creators.vrchat.com/worlds/udon/vrc-graphics/vrc-quality-settings （Last updated 2026-06-10）
> release note 3.10.5 原文："You can now enable/disable Realtime Reflection Probes on all platforms. Use VRCQualitySettings.RealtimeReflectionProbes from Udon." / "You can now enable 'Distance Shadowmask' mode on PC … Use VRCQualitySettings.ShadowmaskMode from Udon."

### 可访问属性（官方枚举顺序，2026-11-10 复核）

| 属性 | 类型 | 读 | 写 |
|---|---|---|---|
| `AntiAliasing` | int | ✅ | ❌ |
| `PixelLightCount` | int | ✅ | ❌ |
| `LODBias` | float | ✅ | ❌ |
| `MaximumLODLevel` | int | ✅ | ❌ |
| `ShadowResolution` | ShadowResolution | ✅ | ❌ |
| `ShadowDistance` | float（钳制 0.1–10000） | ✅ | 走 `SetShadowDistance()` |
| `ShadowCascades` | int | ✅ | ❌ |
| `vSyncCount` | int | ✅ | ❌ |
| `RealtimeReflectionProbes` | **bool** | ✅ | ✅（3.10.5+） |
| `ShadowmaskMode` | **UnityEngine.ShadowmaskMode** | ✅ | ✅（3.10.5+，PC 上即 "Distance Shadowmask"） |

> ⚠️ **官方页存在自相矛盾的措辞（值得记住的坑）**：官方 VRCQualitySettings 页导言写 "A thin **read-only** layer over properties of UnityEngine.QualitySettings"，但同一页把 `RealtimeReflectionProbes` 与 `ShadowmaskMode` 列在 "Currently exposed" 里，且 release note 3.10.5 明确说这两个**可以从 Udon 启用/禁用**。
> 结论（按 release note 为准）：**大部分属性只读，这两个是 3.10.5 新增的可写项**。其余属性若需要修改，仍只能通过 `SetShadowDistance()` 这类专用方法或 inspector 侧的 World Quality Settings 脚本。
> 另有提示："Some of these may be affected by the user's graphics settings."——读到的值可能被玩家画质档位影响，不要用它做逻辑断言。

### World Quality Settings 脚本（3.10.5+）

- `VRCWorld` prefab 自带 **"World Quality Settings"** 脚本，可在 Inspector 里配 **Shadow Distance / Shadowmask Lighting Mode / Realtime Probes**。
- ⚠️ **升级 SDK 不等于现有场景自动获得该组件**——升级后要检查场景里 VRCWorld 上的序列化值（缺失就手动挂）。
- **启动时总是写 Shadowmask Mode**（即该字段不是"可选覆盖"，每次世界加载都会应用）。
- 探针 / 阴影距离有**单独的 override 开关**；关掉 override 就回落到玩家自己的画质设置。
- 所有属性在 world 加载时重置。

### 阴影设置

```csharp
// 按质量等级设置
VRCQualitySettings.SetShadowDistance(float low, float medium, float high, float mobile);
// 所有等级相同
VRCQualitySettings.SetShadowDistance(float allSame);
// 恢复用户配置
VRCQualitySettings.ResetShadowDistance();
```

> ⚠️ 设置阴影距离后用户的 "Shadow Quality" 设置会被覆盖，用户会在菜单中看到警告

### 级联阴影

```csharp
VRCQualitySettings.shadowCascade2Split = float;
VRCQualitySettings.shadowCascade4Split = Vector3;
```

### 事件

`OnVRCQualitySettingsChanged` — 用户更改**暴露属性**对应的图形设置时触发（官方页 "When a user changes graphics settings that affect an exposed property"）。

---

## 2b. 其他 3.10.5 / 3.10.6 的 Udon 侧 API 变化（速查）

| 版本 | 变化 | 备注 |
|---|---|---|
| 3.10.5 | `VRCPickup` 新增 **Outline Renderers** 字段 | authoring，无 Udon accessor |
| 3.10.5 | `VRCRaycast` 新增 **Root Transform** 属性：ray 可从别的 Transform 起步 | Avatar 侧组件 |
| 3.10.5 | `VRCRaycast` 被禁用时参数按"未命中"处理（此前保持禁用前的值） | 行为变更，依赖旧行为的要改 |
| 3.10.5 | Constraint 的 Activate / Zero 可 Undo，且会标记场景为未保存 | Editor 工作流 |
| 3.10.5 | `Assets/` 与 `Packages/` 都能再建 U# 脚本；项目外创建改为**明确的报错提示** | 见 [[concepts/rules/udonsharp-language-limits]] |
| 3.10.5 | 未知的 `VRC_ENABLE_*` scripting define 会被 **strip** | 拼错的 define 会被静默移除 |
| 3.10.6 | `Interact()` 的 UdonBehaviour 新增 **Outline Renderers** | authoring，无 Udon accessor |
| 3.10.6 | **VRCVelocityTracker**（world 组件）：无需自建 update loop 即可从 Udon 读速度 | 见 world 侧组件页 |
| 3.10.6 | **VRCBillboard**：`Register / Unregister`，支持 named arguments（`allowTilt:` / `smoothingFactor:` / `upFrom:` / `matchViewpointRotation:`） | 本目录 [[vrc-billboard.md]] 仍是预告版，待复核 |
| 3.10.6 | 修复 **sync null VRCUrl + persistence** 组合下的 Udon 异常 | 见 [[persistence.md]] |
| 3.10.6 | U# 编译器：enum 按枚举值比较 vs 按整数值比较不一致 → 已修 | 见 [[concepts/rules/udonsharp-language-limits]] |
| 3.10.6 | U# 编译器：**命名参数跳过可选参数**时错位落参 → 已修 | 同上 |

> 说明：本表只列**官方 release note 明文**且有 Udon 面影响的项。Face Tracking / Expressions（Lerp Duration / Bit Width / Representation）属 Avatar 域，不在本页范围。

---

## 3. 语音/化身音频控制

### 语音设置 (VRCPlayerApi)

| 方法 | 参数范围 | 说明 |
|---|---|---|
| `SetVoiceGain(int)` | 0-24 dB | 语音增益，默认 15 |
| `SetVoiceDistanceNear(float)` | 0-1,000,000 米 | 音量开始衰减的近距离阈值，建议 0 |
| `SetVoiceDistanceFar(float)` | 0-1,000,000 米 | 能听到语音的最远距离，默认 25 |
| `SetVoiceVolumetricRadius(float)` | 0-1000 米 | 语音源体积半径，默认 0 |
| `SetVoiceLowpass(bool)` | on/off | 远程语音低通滤波（DJ 场景建议关闭） |

### Avatar 音频设置

| 方法 | 参数 | 说明 |
|---|---|---|
| `SetAvatarAudioGain(int)` | 0-10 dB | Avatar 音频最大增益，默认 10 |
| `SetAvatarAudioFarRadius(float)` | 米 | Avatar 音频最远距离，默认 40 |
| `SetAvatarAudioNearRadius(float)` | 米 | 起始距离，默认 0 |
| `SetAvatarAudioVolumetricRadius(float)` | 米 | 体积半径，默认 0 |
| `SetAvatarAudioForceSpatial(bool)` | | 强制空间化 |
| `SetAvatarAudioCustomCurve(bool)` | | 使用自定义曲线 |

---

## 4. Drones（无人机 API）

| 方法/事件 | 说明 |
|---|---|
| `VRCPlayerApi.GetDrone()` | 获取目标玩家的无人机 API |
| `VRCDroneApi.GetPlayer()` | 获取无人机所属玩家 |
| `OnDroneTriggerEnter/Exit/Stay` | 无人机触发事件 |

---

## 5. MIDI 支持

### MIDI 事件

| 事件 | 参数 | 说明 |
|---|---|---|
| `MidiNoteOn` | channel, noteNumber, velocity | Note On 消息 |
| `MidiNoteOff` | channel, noteNumber, velocity | Note Off 消息 |
| `MidiControlChange` | channel, controlNumber, value | 控制改变（旋钮/滑杆） |

支持实时乐器输入和 MIDI 文件回放。

---

## 6. AI Navigation（AI 导航）

### 能力
- 从场景几何体自动生成 NavMesh
- 支持动态障碍物和 OffMesh Links
- **Runtime 生成**：可在 VRChat 客户端运行时创建和更新导航网格

### VRChat 限制
- **自定义 Agent 类型**：不支持
- `NavMeshSurface.UpdateNavMesh()`：返回不可用的 AsyncOperation
- `NavMeshSurface.CollectObjects`：Enum 数组在 Udon 中不可用

---

## 7. 语言 API

| 方法 | 返回 | 说明 |
|---|---|---|
| `GetCurrentLanguage()` | string (RFC 5646) | 本地用户选择的语言 |
| `GetAvailableLanguages()` | string[] | 玩家可选择的所有语言 |
| `OnLanguageChanged` | string | 本地玩家更新显示语言时触发 |

---

## 8. VRC+ 事件

| 事件 | 参数 | 说明 |
|---|---|---|
| `OnVRCPlusMassGift` | gifter, numGifts (int) | 玩家投放礼物炸弹 |
| `OnVRCCameraSettingsChanged` | VRCCameraSettings | 用户更改图形设置 |
| `OnVRCQualitySettingsChanged` | — | 用户调整影响 Quality Settings 的图形设置 |

---

## 9. VRCObjectSync 完整 API

| 方法/属性 | 说明 |
|---|---|
| `FlagDiscontinuity()` | 传送物体时触发，跳过本帧的平滑处理 |
| `SetGravity(bool)` / `GetGravity()` | 设置/获取重力 |
| `SetKinematic(bool)` / `GetKinematic()` | 设置/获取运动学 |
| `Respawn()` | 重生物体到初始位置和旋转，清除速度 |

---

## 相关页面

[[animator.md]] · [[api-checker.md]] · [[audio.md]] · [[data-containers.md]] · [[dynamics.md]]
