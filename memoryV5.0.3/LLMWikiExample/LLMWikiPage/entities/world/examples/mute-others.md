---
title: "Mute Others"
category: world
subcategory: examples
knowledge_level: applied
status: active
source: 本地知识库整理
source_type: community
version: 1.1
last_review: 2026-11-10
confidence: Medium
tags:
  - world
  - udonsharp
  - reference
aliases:
  - "Mute Others"
  - mute-others.md
related:
  - README.md
  - screen-canvas.md
  - detect-controller-collide.md
  - image-loading.md
  - player-join-zones.md
type: entity
created: 2026-06-20
sources: 本地知识库整理
updated: 2026-11-10
---
# Mute Others

> 来源: VRChat 官方文档 (creators.vrchat.com/worlds/examples/mute-others)
> 抓取日期: 2026-06-15
> 原始 URL: https://creators.vrchat.com/worlds/examples/mute-others/
> 文档版本: Last updated Oct 29, 2024
> SDK: 3.0+ (VRCPlayerApi.SetVoiceDistanceFar)

## Example Central Package

> ✅ **需要 Example Central Package**
> 通过 `VRChat SDK → 🏠 Example Central` 导入
> 包含 MuteButton Prefab、PlayerMuteLogic 脚本、Canvas 文本

### Example World
- **World ID**: `wrld_6b2ed4f1-09be-4dee-91b2-fc49e12ecb88`
- **URL**: https://vrchat.com/home/world/wrld_6b2ed4f1-09be-4dee-91b2-fc49e12ecb88

---

## 概述

演示如何**静音/取消静音**同一实例中的其他玩家。
- 一个按钮切换全局静音状态
- 静音时将所有玩家语音距离设为 0(听不见)
- 取消静音时恢复默认距离(25 米)
- **注意**: 文档原话强调使用 `SetVoiceDistanceFar(0)` **不等于真正的"静音"**,而是"声音传播距离为 0"

---

## 关键 Udon API

| API | 用途 |
|-----|------|
| `VRCPlayerApi.SetVoiceDistanceFar(float distance)` | 设置玩家语音远距离衰减(米) |
| `VRCPlayerApi.GetPlayers()` | 获取实例中所有玩家数组 |
| `VRCPlayerApi.SetVoiceGain(float gain)` | (备选)直接音量控制(0-1) |
| `VRCPlayerApi.GetVoiceGain()` | 获取当前音量 |

### ⚠️ 关键事实
> 文档原话: "When this value is set to 0, it means that other player voices will travel 0 meters before they are silent."
> **不是真正"屏蔽数据包"**,而是"让声音衰减到听不见"
> VRChat 客户端菜单的"屏蔽"才是真正的静音(用户级)

---

## MuteOthers.cs 结构

### 公共变量(Inspector 暴露)

| 名称 | 类型 | 默认 | 说明 |
|------|------|------|------|
| `_buttonLabel` | `Text` | - | 按钮文本引用 |
| `_areOtherPlayersMuted` | `bool` | `false` | 当前静音状态(同步变量) |
| `Message Muted` | `string` | "Unmute Players" | 静音时按钮显示文本 |
| `Message Unmuted` | `string` | "Mute Players" | 未静音时按钮显示文本 |
| `Default Voice Distance` | `float` | `25` | 恢复时使用的距离(VRChat 平台默认) |

### 核心流程

```csharp
public void _Trigger()
{
    _areOtherPlayersMuted = !_areOtherPlayersMuted;
    OnMuteUpdated();
}

private void OnMuteUpdated()
{
    VRCPlayerApi[] players = VRCPlayerApi.GetPlayers();
    float distance = _areOtherPlayersMuted ? 0f : _defaultVoiceDistance;

    foreach (VRCPlayerApi player in players)
    {
        if (player.isLocal) continue;  // 不要影响自己
        player.SetVoiceDistanceFar(distance);
    }

    _buttonLabel.text = _areOtherPlayersMuted
        ? _messageMuted
        : _messageUnmuted;
}
```

---

## 关键设计模式

### 1. 本地行为(无需同步)

> ⚠️ **此示例的所有行为都是本地行为**
> 每个玩家各自的"是否静音"独立
> 不需要任何网络同步(`_areOtherPlayersMuted` 只是脚本内的本地变量)
> 适合"个人体验"类功能

### 2. 排除本地玩家
```csharp
if (player.isLocal) continue;  // 关键: 自己的语音不受影响
```

### 3. 通过"距离衰减"模拟静音
- 这是 VRChat 允许 Udon 操控音频的**唯一方式**
- `SetVoiceDistanceFar(0)` 让声音在 0 米处衰减完
- 副作用: 玩家在空间上听起来会"突然消失",而非真实"屏蔽"

---

## 二次开发建议(Challenge 章节)

### 进阶: 只静音部分玩家
> 文档原话: "Can you update this prefab to only mute some players? One approach would be to do this based on where players are..."

**实现方向**:
- 结合 `PlayerJoinZones` 示例(基于 Trigger 区域选择玩家)
- 根据距离、阵营、Tag 过滤
- 维护一个"被静音玩家 ID 列表"代替全局开关

**互补示例**: [[entities/world/examples/player-join-zones]]

---

## Networking 设计要点

| 同步策略 | 状态 |
|----------|------|
| `_areOtherPlayersMuted` | ❌ 不需要同步 |
| `_buttonLabel.text` | ❌ 不需要同步 |
| `SetVoiceDistanceFar` 调用 | ❌ 不需要同步 |

**原因**: 玩家对"自己想听谁"的需求是**完全本地化**的。
- A 想静音 B 不代表 C 也想
- 这是 VRChat "User Safety" 设计的一部分
- 玩家应通过 VRChat 客户端菜单进行"个人级别"的屏蔽

---

## 与知识库互补

- **VRCPlayerApi 完整 API**: [[entities/api/player-api]](待建)
- **Player Audio API**: [[entities/world/udon/players/player-audio]](待建)
- **PlayerJoinZones(区域选择)**: [[entities/world/examples/player-join-zones]] ⭐ 直接互补
- **VRCPlayerApi.SetVoiceGain vs SetVoiceDistanceFar**: 互链 [[entities/api/player-api]]

## ⭐ 工业级升级方案: TLP UdonVoiceUtils

> 如果你的 World 不止"全局静音",而是需要:
> - 多区域组合(房间 + 门 + 触发器)
> - 玩家之间隐私分组(多频道)
> - 遮挡检测(墙后自动降低音量)
> - 优先级仲裁(多个静音规则重叠时谁生效)
> - 跨实例持久化(玩家偏好)
>
> **官方 MuteOthers 是教学版,UVU 是工业级方案**

| 维度 | 官方 MuteOthers | UVU 升级 |
|------|----------------|---------|
| 同步 | 本地(各玩家独立) | 全员同步 + Master 权威 |
| 持久化 | ❌ | ✅ PlayerData 100KB |
| 优先级 | ❌ | ✅ Priority 字段 + 仲裁 |
| 遮挡检测 | ❌ | ✅ `Physics.RaycastNonAlloc` |
| 多区域 | ❌(只能全局) | ✅ 房间/门/触发器叠加 |
| 调试 | 无 | ✅ Gizmos + Custom Editor |

### UVU 版本核对（2026-11-10）

| 版本 | 发布日 | 备注 |
|---|---|---|
| 5.1.4 | 2026-05-02 | 本库原先记录的版本 |
| 6.0.0 | 2026-07-12 | 大版本（跨版本迁移节点） |
| **7.0.0** | **2026-09-02** | **当前最新**；包名 `tlp.udonvoiceutils` |

> **证据说明**：UVU 的 GitHub Releases 里 **6.0.0 与 7.0.0 的 release 正文为空**（本库实测 `body` 长度为 0），没有官方 changelog 文字可引。以下迁移点来自 **v7.0.0 的 `package.json`** 与仓库 README/Wiki 结构，而不是 release notes 的复述。

**v7.0.0 `package.json` 实测要点**：

| 字段 | 值 | 迁移含义 |
|---|---|---|
| `version` | `7.0.0` | — |
| `unity` | `2022.3` | 与 SDK 3.10.6 的 Unity 基线一致，不要升到 Unity 6 |
| `vpmDependencies` | `tlp.udonutils: ^15.0.3`、`tlp.udonavltree: ^2.0.0` | **必须一起满足**，缺一个会编译失败 |
| `legacyFolders` | `Assets\Guribo\UdonVoiceUtils`、`Assets\Guribo\BetterPlayerAudio`、`Assets\TLP\UdonVoiceUtils` | VPM 会按这三个旧路径做**原地迁移** |

#### 迁移步骤（5.1.4 → 7.0.0，跨 6.0.0 / 7.0.0 两个大版本）

1. **只走 VPM**：从 [TLP VPM 列表](https://guribo.github.io/TLP/)（VCC / ALCOM）升级，**不要手动覆盖文件夹**——`legacyFolders` 的迁移逻辑只有 VPM 安装器会执行。
2. **先清手拷残留**：若工程里存在手工拷进来的 `Assets/Guribo/UdonVoiceUtils`、`Assets/Guribo/BetterPlayerAudio`、`Assets/TLP/UdonVoiceUtils`，先删除再装，否则会出现**两份同名脚本**（编译报重复类型）。
3. **依赖一起升**：`tlp.udonutils ≥ 15.0.3`、`tlp.udonavltree ≥ 2.0.0`；这类底层库是 UVU 的运行前提，只升 UVU 会缺 API。
4. **先备份场景再升级**：跨两个大版本，序列化字段有改名/迁移的可能；用分支或场景副本验证，不要直接在正式世界场景上升。
5. **升级后逐个复查引用**：Audio Zone / Occlusion / Voice Directionality 组件若报 Missing Script 或字段回默认值，说明旧序列化没被迁移——按原配置在 Inspector 里重新指认，不要批量替换。
6. **验证顺序**：ClientSim 里先验「多区域 + 遮挡 + 优先级」三件套，再 Build & Test 验同步（UVU 依赖 Master 权威 + 网络同步，Editor 单机测不出问题）。
7. **与其它大版本同批升级的顺序**：基础库（`tlp.udonutils`）→ UVU → 依赖 UVU 的世界插件；与音频无关的（VRCLV v3、VVMW 1.8.3）可并行。

> **本轮不覆盖**：6.0.0 → 7.0.0 之间的**具体 API 增删**无法从官方渠道取得（release 正文为空、仓库无 CHANGELOG.md），因此本页**不列 API diff**；需要逐方法核对时以 v7.0.0 包内 `Runtime/` 源码为准。

**完整资源**:
- **推荐插件索引**: [[concepts/hybrid/udon-world-plugins]] ⭐NEW 2026-06-20
- **深度案例研究**: （来源：UdonVoiceUtils 分析，本地整理） (35 个 .cs + 10 大难题 + 10 个设计模式)
- **VPM 仓库/包清单**: [UdonVoiceUtils · package.json（master 分支）](https://github.com/Guribo/UdonVoiceUtils/blob/master/Packages/tlp.udonvoiceutils/package.json)

## 相关 Udon 文档链接

- [Set Voice Distance Far](https://creators.vrchat.com/worlds/udon/players/player-audio#set-voice-distance-far)
- [VRCPlayerApi 完整文档](https://udonsharp.docs.vrchat.com/vrchat-api/#vrcplayerapi)
- [VRC Audio 系统总览](https://creators.vrchat.com/worlds/udon/players/player-audio)
