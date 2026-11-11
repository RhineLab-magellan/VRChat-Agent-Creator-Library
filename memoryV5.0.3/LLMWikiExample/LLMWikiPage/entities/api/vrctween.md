---
title: "VRCTween — 官方补间 API（SDK 3.10.4+）"
category: api
knowledge_level: core
status: active
source: "VRChat 官方 release note 3.10.4 + creators.vrchat.com VRCTween 四页文档 + vrchat/packages release 3.10.4"
source_type: official
version: 1.0
last_review: 2026-11-10
confidence: High
tags:
  - api
  - vrctween
  - animation
  - udonsharp
  - dotween
aliases:
  - "VRCTween"
  - "VRCTweenHandle"
  - "补间 API"
  - "Tween API"
related:
  - index.md
  - dynamics.md
  - audio.md
  - animator.md
  - ../world/udon/vrctween/index.md
  - ../world/udon/vrctween/tween-types.md
  - ../world/udon/vrctween/control.md
  - ../world/udon/vrctween/custom-tween.md
type: entity
created: 2026-11-10
sources: "VRChat 官方 release note 3.10.4 + creators.vrchat.com VRCTween 四页文档 + vrchat/packages release 3.10.4"
updated: 2026-11-10
---
# VRCTween — 官方补间 API（SDK 3.10.4+）

> SDK 引入版本: **3.10.4**（2026-06-17 发布，release note 原文 "Added VRCTween, which exposes select functionality from the powerful DOTween library to Udon"）
> 最后核证: 2026-11-10（对照官方 release note 3.10.4 + 官方 VRCTween 四页文档，文档页 Last updated: 2026-05-28）
> 完整本地化教程见 [[entities/world/udon/vrctween/index]]（7 个子页），本页是 **API 面速查**，不重复教程细节。

---

## 1. 定位（一句话）

VRCTween 是 SDK 内置的 **DOTween 封装**：把"每帧手写 lerp/slerp + 计时"换成"一行创建 + 链式配置 + 一个 handle 控制"。

| 维度 | 事实 | 来源 |
|---|---|---|
| 引入版本 | 3.10.4（Beta.2 修 World SDK 集成，Beta.3 加 DelayedSetActive / TweenPitch） | release note 3.10.4 |
| 覆盖能力 | **"select functionality"**——DOTween 的**部分**能力，不是全量 | release note 3.10.4 措辞 |
| Udon Graph | ✅ 支持（节点搜索 "VRCTween …"） | 官方 VRCTween 主页 |
| 网络同步 | ❌ **不自动同步**，逐客户端本地播放 | 官方主页 Limitations |
| 异常行为 | ❌ **不抛异常**，无效输入静默返回无效 handle / 静默 no-op | 官方主页 Input Validation |

---

## 2. UdonSharp 使用前提

```csharp
using UdonSharp;
using UnityEngine;
using VRC.SDK3.Components;   // ← VRCTween / VRCTweenHandle 所在命名空间

public class TweenDemo : UdonSharpBehaviour
{
    public GameObject cube;

    void Start()
    {
        // 创建即返回 handle；扩展方法形式（也有 VRCTween 静态形式）
        VRCTweenHandle h = cube.TweenPosition(new Vector3(0, 5, 0), 2f, VRCTweenEase.OutQuad);
    }
}
```

> 两种等价写法：**目标类型的扩展方法**（`gameObject.TweenPosition(...)`）或 **`VRCTween` 静态方法**；两者都返回 `VRCTweenHandle`。

---

## 3. 内置补间类型（创建方法）

| 类别 | 方法 | 作用对象 |
|---|---|---|
| Transform | `TweenPosition` / `TweenLocalPosition` | GameObject 或 Transform |
| Transform | `TweenRotation` / `TweenLocalRotation`（欧拉角） | GameObject 或 Transform |
| Transform | `TweenScale`（只操作 `localScale`；Unity 世界缩放只读，故**无** TweenGlobalScale） | GameObject 或 Transform |
| Path | `TweenPath`（世界空间）/ `TweenLocalPath`（局部空间） | GameObject 或 Transform |
| UI | `TweenColor` / `TweenFade` | `Graphic`（Image / Text / RawImage …） |
| UI | `TweenFade` | `CanvasGroup` |
| UI | `TweenValue` | `Slider` |
| UI | `TweenAnchorPos` / `TweenSizeDelta` | `RectTransform` |
| Sprite | `TweenColor` / `TweenFade` | `SpriteRenderer` |
| Renderer | `TweenColor` / `TweenFloat`（走 **MaterialPropertyBlock**，不生成材质实例） | `Renderer` |
| Light | `TweenIntensity` / `TweenColor` | `Light` |
| Audio | `TweenVolume` / `TweenPitch` | `AudioSource` |

> `TweenPitch` 是 3.10.4-beta.3 补的：官方原文 "Added TweenPitch for AudioSources, enabling you to animate the pitch of audio playback"。
> Renderer 类可传自定义 shader 属性名（如 `TweenColor("_EmissionColor", ...)`），默认 `_Color`。

---

## 4. 虚拟补间（Virtual Tweens）

虚拟补间把插值结果**每帧写进 UdonBehaviour 上的一个字段**，用途是那些内置类型覆盖不到的量（FOV、Animator 参数、粒子参数……）。

| 方法 | 签名要点 |
|---|---|
| `VRCTween.TweenFloat` | `(float from, float to, float duration, UdonBehaviour target, string fieldName, string callbackName, ease)` |
| `VRCTween.TweenInt` | 同上，int 值域 |
| `VRCTween.TweenColor` | Color 值域 |
| `VRCTween.TweenVector3` | Vector3 值域（缓动作用于**整个向量**，不逐轴） |
| `VRCTween.DelayedCall` | `(UdonBehaviour target, string eventName, float delay)` — **可 Kill 的延迟事件**，替代 `SendCustomEventDelayedSeconds` |
| `VRCTween.DelayedSetActive` | `(GameObject target, bool active, float delay)` — 3.10.4-beta.3 新增；目标先被销毁则静默跳过 |

硬约束（官方原文）：

- 承载值的字段必须 **public**，否则虚拟补间写不进去；
- 建议加 `[System.NonSerialized]`，避免这个瞬态值被 Unity 序列化进场景；
- 同一脚本可同时跑多个虚拟补间，用不同字段名区分。

```csharp
[System.NonSerialized] public float fovValue;

VRCTweenHandle _fov = VRCTween.TweenFloat(60f, 90f, 2f, this,
    nameof(fovValue), nameof(OnFovUpdate), VRCTweenEase.OutQuad);

public void OnFovUpdate() { myCamera.fieldOfView = fovValue; }
```

> `DelayedCall` 与 `DelayedSetActive` 是**唯一被 release note 点名**的"可取消延迟"优势：返回 handle，`Kill()` 即取消。

---

## 5. 控制方法（实例方法，作用在 handle 上）

| 方法 | 行为 |
|---|---|
| `Play()` | 恢复被暂停的 tween（**不是** `Resume()`） |
| `Pause()` | 暂停（返回 handle 可继续链式） |
| `Kill()` | 停止并移除 |
| `Complete()` | 立即跳到终点值 |
| `Restart()` | 从头重播 |
| `Flip()` | 翻转方向，但**不自动播放**（暂停态仍暂停） |
| `PlayForwards()` / `PlayBackwards()` | 设方向**并播放** |
| `Goto(float to, bool andPlay)` | 跳到指定秒；`andPlay=true` 继续播放，否则停在该处 |
| `VRCTween.KillAll()` | 杀全部活动 tween（静态） |
| `gameObject.KillAllTweens()` | 杀某 GameObject 上的全部 tween |

### 查询属性

| 属性 | 类型 | 含义 |
|---|---|---|
| `IsValid` | bool | handle 是否指向真实 tween（默认 handle = false） |
| `IsPlaying` | bool | 正在播放 |
| `IsActive` | bool | 存在且未被 Kill |
| `Elapsed` | float | 已过秒数 |
| `Duration` | float | 总时长 |
| `IsBackwards` | bool | 正在反向播放 |

---

## 6. 配置方法（链式，返回 handle）

| 方法 | 关键约束 |
|---|---|
| `OnComplete(UdonBehaviour, string eventName)` | 回调方法必须 **public**；不同 tween 用不同事件名 |
| `OnRewind(UdonBehaviour, string eventName)` | 只在**反向播放回到 0** 时触发；`Restart()` **不**触发 |
| `SetLoops(int loops, VRCTweenLoopType)` | `-1` = 无限；`Restart(0)` / `Yoyo(1)` / `Incremental(2)` |
| `SetDelay(float)` | 负数 / NaN / Infinity 静默忽略 |
| `SetUpdate(VRCTweenUpdateType)` | `Update(0)` / `LateUpdate(1)` / `FixedUpdate(2)` / `PostLateUpdate(3)`（3 = 所有 LateUpdate 之后，可压相机/IK 的位移） |
| `From()` | 反向 tween：从目标值出发回到当前值；**仅非虚拟 tween**，且必须创建后立刻调用 |
| `SetSpeedBased()` | duration 语义变"每秒单位数"；必须在创建后立刻调用，播放后无效 |
| `SetEase(VRCTweenEase \| AnimationCurve)` | 可对活动 tween 调用 |
| `SetDuration(float)` | 可对活动 tween 调用；非法值忽略 |
| `ChangeEndValue(newEndValue, bool snapStartValue)` | 可对活动 tween 调用；`snapStartValue=true` 时起点吸附当前值 |

> 🔴 **时机纪律**：`OnComplete` / `SetLoops` / `SetDelay` / `SetUpdate` / `From` / `SetSpeedBased` 必须在**创建后同一帧内**应用，否则可能不生效；只有 `SetDuration` / `SetEase` / `ChangeEndValue` 支持随时改。

### `ChangeEndValue` 的支持边界（官方明列）

| 支持 | 不支持 |
|---|---|
| Transform / UI / Sprite / Audio / 虚拟 **float·int** tween | 虚拟 **Color·Vector3** tween、Renderer（MaterialPropertyBlock）tween、Light tween、Path tween |

> 这条直接决定"复用模式"能用在哪些 tween 上：想每帧重定向，先确认类型在左列。

---

## 7. 输入验证（静默拒绝，永不抛异常）

官方设计取舍：宁可"什么都没发生"，也不要让异常打断整个 UdonBehaviour。

**创建阶段返回无效 handle（跳过创建）**：

| 输入 | 条件 |
|---|---|
| `target` | null |
| `duration` | 负数 / NaN / Infinity（**0 允许**，用于造可复用 tween） |
| 位置 / 缩放 / 路径点 | 含 NaN、Infinity，或分量绝对值 > ~520,000 |
| Path `resolution` | 超出 1–50 → **静默 clamp**（tween 仍会创建） |

> 旋转 tween 只要求"有限值"，大欧拉角是允许的（不受 520k 幅度限制）。

**配置/控制阶段静默 no-op**：`SetDuration` / `SetDelay` 收到非法值；`Goto` 收 NaN/Inf（否则 clamp 到 duration）；`ChangeEndValue` 收到不过幅度检查的值。

> 约定：创建后检查 `handle.IsValid` 再决定是否走后续逻辑。

---

## 8. 与 DOTween 的边界（**不要假设 DOTween 全量可用**）

| 断言 | 证据 |
|---|---|
| VRCTween 是**封装**，暴露的是 DOTween 的 **select functionality** | 3.10.4 release note 原句 |
| 底层是 DOTween 免费版核心，**不含 Pro**（可视化编辑器 / 可视化路径编辑器 / 2D Toolkit / TMP 扩展不可用） | KB 既有页 [[entities/world/udon/vrctween/index]] 的 DOTween 章（2026-06-22 校对记录） |
| DOTween 的 **Sequence / InsertCallback / 直接链式 DOTween API** 不在 VRCTween 的公开面内 | 官方文档只列本页 §3–§6 的方法；无 Sequence 类型 |
| 序列只能**用 `OnComplete` 回调串联** | 官方主页 "Chain Tweens with Callbacks" |
| 池化 / 低开销来自 DOTween 内部池，但**不能**假设可以手动池化 handle | 官方只承诺 "Tweens are automatically cleaned up when they complete" |

> ⚠️ 反面纪律：不要因为"底层是 DOTween"就推断 `DOTween.Sequence()`、`DOTween.To()`、`SetOptions` 之类可用——这些**没有**出现在 VRCTween 官方文档里，属于未验证面。

---

## 9. 已知限制与陷阱

1. **不同步**：每个玩家独立播放；要一致必须自己用 `[UdonSynced]` 存开始时间戳，在 `OnDeserialization` 里 `Goto(elapsed, true)`。
2. **超短 tween**（< 0.01s）动画不流畅。
3. **handle 是逐场景实例唯一**（"Tween handles are unique per scene instance"）——不要跨实例复用 handle。
4. **反向回到起点不会被自动 Kill**，可从 `OnDestroy` 手动 Kill。
5. **长/无限 tween 必须显式清理**：`gameObject.KillAllTweens()`（`OnDestroy`）+ `VRCTween.KillAll()`（全场景）。
6. **一次性 vs 复用**：交互触发的 5–50 个 tween 直接新建即可（DOTween 池化）；每帧重定向才需要 `ChangeEndValue + SetDuration + SetEase + Restart` 复用。
7. **批量创建要错帧**：官方建议用 `delayBetween` 逐个错开，避免同一帧创建上百个 tween。

---

## 10. 3.10.4 系列内的时间线（避免把 beta 行为当 stable）

| 版本 | 与 VRCTween 相关的点 |
|---|---|
| 3.10.4（2026-06-17） | 首发 VRCTween / VRCTweenHandle；DelayedCall 可 Kill |
| 3.10.4-beta.2 | 修 World SDK 集成（Worlds 项目里首次可用） |
| 3.10.4-beta.3 | 新增 `DelayedSetActive`；新增 `TweenPitch`（AudioSource 音高）；支持创建"暂停 + 零时长"的可复用 tween；多 Avatar 同场景时 Contact/PhysBone 的 Allow Self / Allow Others 按"每个 Avatar 一个玩家"处理 |

> 本页签名一律以 **stable 3.10.4 官方文档页**为准；beta 项已合入但**未在 3.10.6 release note 里再提**，视为稳定面。

---

## 11. 与 KB 其他页的关系

| 主题 | 页面 |
|---|---|
| VRCTween 教程全集（7 子页） | [[entities/world/udon/vrctween/index]]、`basics.md`、`tween-types.md`、`control.md`、`sequence.md`、`custom-tween.md`、`patterns.md` |
| World 动力学（Contact / PhysBone / Constraint） | [[dynamics.md]] |
| 音画（AudioSource 播放控制） | [[audio.md]] |
| Animator 参数驱动 | [[animator.md]] |
| 事件执行顺序（tween 回调的时机） | [[entities/world/udon/event-execution-order]] |
| 语言限制（回调方法必须 public 等原因） | [[concepts/rules/udonsharp-language-limits]] |

---

## 12. 待复核 / 越界报告

- 既有页 `entities/world/udon/vrctween/index.md` 第 48 行写 "SDK 要求: VRChat Worlds SDK **3.7+** (2024 年发布)"，与官方 release note 3.10.4 "Added VRCTween" **冲突且无来源锚点**。该页不在本任务 write scope，**只报告不修改**（建议由 world 侧任务修正为 3.10.4）。
- 官方 VRCTween 四页文档的 `Last updated` 为 2026-05-28，早于 3.10.4 stable（2026-06-17）——文档面未随 beta.3 的两项新增（DelayedSetActive / TweenPitch）更新标题时间戳，但正文**已包含**这两项，故以正文为准。

---

## 13. 来源

| 事实 | 来源 URL |
|---|---|
| VRCTween 引入 / "select functionality" / DelayedCall 可 Kill / beta.2·beta.3 三项 | https://creators.vrchat.com/releases/release-3-10-4/ |
| 命名空间、创建方式、类型清单、Limitations、Input Validation、Performance Tips | https://creators.vrchat.com/worlds/udon/vrctween/ |
| 各类型示例与参数（Path 的 closePath / resolution、Renderer 属性名） | https://creators.vrchat.com/worlds/udon/vrctween/tween-types |
| 控制方法 / 查询属性 / 配置方法 / ChangeEndValue 边界 / Ease 表 | https://creators.vrchat.com/worlds/udon/vrctween/settings |
| 虚拟补间签名、字段必须 public、DelayedCall/DelayedSetActive | https://creators.vrchat.com/worlds/udon/vrctween/virtual-tweens |
| 3.10.4 发布时刻（2026-06-17）与 3.10.6 基线 | https://github.com/vrchat/packages/releases/tag/3.10.4 、 https://github.com/vrchat/packages/releases/tag/3.10.6 |
