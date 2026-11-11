---
title: "Avatar Contact System (VRC Contact, 2026.2.1+)"
category: avatar
subcategory: contact
knowledge_level: applied
status: active
source: "VRChat 官方文档 creators.vrchat.com/avatars/contact + 2026.2.1 Release Notes"
source_type: official
version: 1.4
last_review: 2026-11-10
confidence: High
tags:
  - avatar
  - contact
  - vrc-contact
  - dynamics
  - sdk-3-10
aliases:
  - "VRC Contact"
  - "Avatar Contact"
  - "Contact System"
  - 接触系统
  - "VRC Contact Receiver"
  - "VRC Contact Sender"
related:
  - ../api/dynamics.md
  - avatar-dynamic-bone-limits.md
  - optimization-guide.md
  - performance-benchmarks.md
  - ../world/scene-components/index.md
  - ../world/udon/world-debug-views.md
  - ../api/vrc-billboard.md
type: entity
created: 2026-07-01
sources: "VRChat 官方文档 creators.vrchat.com/common-components/contacts/ + Release 3.10.4 / 3.10.6 + 2026.2.1 Release Notes (复核 2026-11-10)"
updated: 2026-11-10
---

# Avatar Contact System (VRC Contact, 2026.2.1+)

> 2026.2.1 Release Notes 引入 Avatar-side 公开 Contact API。
> 本文档汇总 Avatar ↔ World / Avatar ↔ Avatar 的接触检测机制与限制。

---

## 概述

VRC Contact 是 VRChat 内置的接触检测系统,与 PhysBone 同属 Dynamics 框架。2026.2.1 起 Avatar 可通过 `VRCContactReceiver` 组件公开接触行为,World 端通过 Udon 事件 `OnContactEnter` / `OnContactExit` **两个**回调接收通知（**没有 `OnContactStay`**，见 §World 端 Udon 事件的纠错块）。

| 维度 | World Contact (SDK 3.10.0+) | Avatar Contact (2026.2.1+) |
|------|------------------------------|------------------------------|
| 发送方 | World 对象 (`VRCContactSender`) | Avatar 骨骼 (`VRCContactReceiver` 上配对 Sender) |
| 接收方 | World 对象 (`VRCContactReceiver`) | Avatar 公开参数 / 动画 / FX Layer |
| 触发 | Udon `OnContactEnter` 等 | Animator 参数 / PhysBone 状态 |
| 限制 | 1024 房间上限 | 见 §性能基准 |

### 📌 Avatar Descriptor Colliders 默认包含 Contacts Sender(2026-07-04 来自 yexcadocs)

> 来源:yexcadocs §VRC Avatar Descriptor(P1)
> 本节为社区实操经验补充。

**重要事实**:
- Avatar Descriptor 的 Colliders **默认就是 Contacts Sender**
- 默认参数为 `"Head"` / `"Body"` / `"Hand L"` / `"Hand R"` 等
- 手动设置 Contacts 时,**无需重复添加这些部位的 Sender**

**含义**:
- Avatar 上的 VRCContactReceiver 只需配置 `collisionTags` 为 "Head"/"Body" 等
- 自动与 Avatar Descriptor Colliders 配对
- 节省 Sender 数量配额(影响 Performance Rank)

---

## 核心组件

### VRCContactSender(接触发射器)

挂载到 GameObject 上,定义接触源的位置、形状、碰撞标签。

| 字段 | 说明 |
|------|------|
| `shape` | Sphere / Capsule / **Box**（SDK 3.10.4 起宽高深可独立设置）,定义接触体积 |
| `collisionTags` | 字符串标签列表,用于 Sender ↔ Receiver 匹配 |
| `radius` / `size` | 接触体积大小 |
| `allowSelfContact` | 是否允许自身 Avatar 的其他 Sender 触发 |

### VRCContactReceiver(接触接收器)

挂载到 GameObject 上,定义接收接触的目标。

| 字段 | 说明 |
|------|------|
| `shape` | 同 Sender（含 3.10.4 起的 **Box**，可启用 **Use Face Proximity**） |
| `collisionTags` | 至少 1 个与 Sender 匹配才触发 |
| `onEnter` | 进入时触发的事件 |
| `onExit` | 离开时触发的事件 |

> ⚠️ **没有 `onStay`**：官方 `VRCContactReceiver` 只有进入/退出两个回调，**不存在"每帧持续触发"的事件**。需要"持续接触"语义时，在 `OnContactEnter` 里置本地标志、`OnContactExit` 里清除，再在 `Update` 里读该标志（见下文纠错块）。
| `allowSelfContact` | 同 Sender |

---

## World 端 Udon 事件

> ⚠️ **2026-11-10 复核提示**：官方 3.10.6 [Contacts 文档](https://creators.vrchat.com/common-components/contacts/) 只列出 **两个** 事件 —— `OnContactEnter(ContactEnterInfo contactInfo)` 与 `OnContactExit(ContactExitInfo info)`，info 结构字段为 `contactSender`（`ContactSenderProxy`）、`contactReceiver`（`ContactReceiverProxy`）、`enterVelocity`（`Vector3`）、`contactPoint`（`Vector3`，世界空间）、`matchingTags`（`string[]`）。
>
> ⚠️ **不存在 `OnContactStay`（2026-11-10 纠错）**：VRChat Contacts 在 World 侧**只有进入 / 退出两个回调** —— `OnContactEnter(ContactEnterInfo)` 与 `OnContactExit(ContactExitInfo)`。官方 [Contacts 组件页](https://creators.vrchat.com/common-components/contacts/) 与 [Release 3.10.6](https://creators.vrchat.com/releases/release-3-10-6/) 全文都**没有 Stay 事件**；本页旧版把 `OnContactStay` 写成「每帧持续触发」是**误记**。
>
> **需要「持续接触」语义时的标准实现**（Enter 置标志 → Exit 清标志 → Update 里读标志）：
>
> ```csharp
> using VRC.Dynamics;
> using UdonSharp;
> using UnityEngine;
>
> public class ContactHold : UdonSharpBehaviour
> {
>     private bool _inContact;
>
>     public override void OnContactEnter(ContactEnterInfo info)
>     {
>         _inContact = true;   // ① Enter 置标志
>         // info.contactSender / info.contactReceiver / info.enterVelocity / info.contactPoint / info.matchingTags
>     }
>
>     public override void OnContactExit(ContactExitInfo info)
>     {
>         _inContact = false;  // ② Exit 清标志
>     }
>
>     private void Update()
>     {
>         if (!_inContact) return;   // ③ 自己每帧读标志（= 旧文档里想象的 Stay 语义）
>         // 轻量逻辑：计时、充能、渐变……
>     }
> }
> ```
>
> 不想每帧 `Update` 的话，可改用 `SendCustomEventDelayedSeconds` 做低频轮询（例如 0.1s 一次），成本更低。
>
> 下文代码示例保留旧类型的字段注释作**历史对照**，但已移除 `OnContactStay`；另注意官方 info 类型为 `ContactEnterInfo` / `ContactExitInfo`，下文旧写法 `ContactReceiverInfo` 不再与官方文档对应。

### 三个核心事件（⚠️ 仅前两个存在）

```csharp
using VRC.Dynamics;

public override void OnContactEnter(ContactReceiverInfo info) {
    // info.isAvatar - bool,是否来自 Avatar
    // info.player - VRCPlayerApi(可能为 null)
    // info.contactPoint - Vector3 接触点
    // info.collisionTags - 触发接触的标签数组
    if (info.isAvatar && info.player != null) {
        Debug.Log($"Touched by {info.player.displayName}");
    }
}

public override void OnContactExit(ContactReceiverInfo info) {
    // 离开事件
}

// ❌ 不存在：OnContactStay —— VRChat Contacts 没有 Stay 回调。
//    需要「持续接触」请在 OnContactEnter 置标志、OnContactExit 清标志、Update 里读标志。
```

### ContactReceiverInfo 字段（⚠️ 与官方 `ContactEnterInfo` / `ContactExitInfo` 不符）

| 字段 | 类型 | 说明 |
|------|------|------|
| `isAvatar` | bool | 是否来自 Avatar(World Sender 也可为 true) |
| `isLocal` | bool | 是否本地玩家触发 |
| `player` | VRCPlayerApi | 触发玩家(可能为 null) |
| `contactPoint` | Vector3 | 接触点世界坐标 |
| `collisionTags` | string[] | 触发标签（⚠️ 官方对应字段叫 `matchingTags`，且语义是交集，见下方对照） |

#### ⚠️ 类型名与字段的官方对照（2026-11-10 复核）

> 本节旧表保留**只是为了兼容引用本页小节的旧笔记**（直接删除会打断引用），但**它不是官方签名**，新代码不要照抄。

**官方 3.10.6 的定义**（[Contacts 组件页](https://creators.vrchat.com/common-components/contacts/)「Udon Access in Worlds」段）：

| 事件 | 参数类型 | 命名空间 |
|------|----------|----------|
| `OnContactEnter` | **`ContactEnterInfo`** | `VRC.Dynamics`（组件类型在 `VRC.SDK3.Dynamics.Contact.Components`） |
| `OnContactExit` | **`ContactExitInfo`** | 同上 |

**官方 info 结构字段**：

| 官方字段 | 类型 | 说明 | 与本页旧写法的关系 |
|----------|------|------|------------------|
| `contactSender` | `ContactSenderProxy` | 本次碰撞涉及的 Contact Sender 组件引用 | 旧表无对应项 |
| `contactReceiver` | `ContactReceiverProxy` | 本次碰撞涉及的 Contact Receiver 组件引用 | 旧表无对应项 |
| `enterVelocity` | `Vector3` | 接触瞬间 Sender 相对 Receiver 的**相对速度** | 旧表无对应项（**做撞击力度/音效强弱的关键字段**） |
| `contactPoint` | `Vector3` | 接触点的**世界坐标**（估计值） | ✅ 与旧表 `contactPoint` 一致 |
| `matchingTags` | `string[]` | Sender 与 Receiver **共同拥有**的标签 | ⚠️ 旧表叫 `collisionTags`；官方是**交集**，不是 Receiver 配置的全部标签 |

**旧表里在官方文档中查不到的字段**（不要依赖）：

| 旧字段 | 状态 | 替代做法 |
|--------|------|----------|
| `isAvatar` | ❌ 官方无此字段 | 用 `info.contactSender` 反查所属对象；Avatar 侧也可用 3.10.6 的 `{parameter}_SenderType`（Int，枚举语义见下）判断来源类型 |
| `isLocal` | ❌ 官方无此字段 | 由 sender 反查所属 `VRCPlayerApi` 后比对 `isLocal`；World 侧更常用 `Networking.LocalPlayer` |
| `player` | ❌ 官方 3.10.6 文档未列 | 由 `contactSender` 反查 `VRCPlayerApi`；World 物体接触时取不到，**必须走空值分支** |

> **`{parameter}_SenderType` 的 0-4 编码——官方锚点**：[FACT] 官方 [Contacts 文档 · Advanced Parameters](https://creators.vrchat.com/common-components/contacts/) 原文：
>
> | 值 | 官方原文含义 |
> |----|--------------|
> | **0** | 没有 Contact Sender 正在接触该 Receiver，**或** Sender 挂在 VRChat **无法识别**的对象上 |
> | **1** | Sender 与 Receiver 属于**同一个 Avatar** |
> | **2** | Sender 属于**另一个 Avatar** |
> | **3** | Sender 属于 **World** |
> | **4** | Sender 属于**生成的 item**（spawned item） |
>
> 触发条件：该参数**仅在 Receiver 勾选 Advanced Parameters 后才生成**，且是 `{parameter}` 前缀下的一个 Int 参数；多个 Sender 同时接触时取**proximity 最高**的那个（3.10.6-beta.3 起改为：proximity 都为 1 时取 **point separation 更小**的）。

**建议写法**（保守、可编译）：

```csharp
using VRC.Dynamics;

public override void OnContactEnter(ContactEnterInfo info)
{
    Vector3 point = info.contactPoint;      // 官方字段
    Vector3 vel   = info.enterVelocity;     // 官方字段：撞击相对速度
    string[] tags = info.matchingTags;      // 官方字段：交集标签
    var sender    = info.contactSender;     // 官方字段：Sender proxy

    // 需要玩家身份时：从 sender 侧反查，不要指望 info.player
    // World 物体的 sender 没有所属玩家，必须走空值分支
}
```

> 📌 **为什么不直接删掉旧表**：`ContactReceiverInfo` 这个名字在社区教程与旧笔记里流传很广，本页又被多处引用；直接删除会让「按旧笔记查字段」的人失去对照物。保留旧表 + 并排给官方签名，读者能自己看清差异。

---

## Avatar ↔ World 通信

### 场景:Avatar 进入 World 触发区

```
Avatar(挂载 VRCContactReceiver collisionTags=["WorldTrigger"])
  ↓ 进入
World Object(挂载 VRCContactSender collisionTags=["WorldTrigger"])
  ↓ 触发
World UdonBehaviour.OnContactEnter(info)
  → 业务逻辑:开门/给积分/触发剧情
```

### 场景:World 物体接触 Avatar 身体部位

```
World Object(VRCContactSender collisionTags=["Head","Hand"])
  ↓ 接触
Avatar(VRCContactReceiver collisionTags=["Head","Hand"])
  ↓ 触发
Avatar Animator Parameter("IsTouched", true)
  → FX Layer 播放触碰反应动画
```

---

## 性能基准(来自 `avatar/performance-benchmarks.md`)

| 配置 | 每 1000 个 Contacts 的帧时间 |
|------|------------------------------|
| 关闭状态 | **0.5 ms** |
| 开启 Receivers | **0.75 ms**(+50%) |
| 本地 Avatar 参数 | **+1.5 ms** 每 1000 个 |

**房间最大上限**: 4096 个 Contacts(超出部分停止工作)。

### 优化建议

- **形状选小**: Sphere > Capsule > Cube(碰撞检测成本递增)
- **类型选少**: 同 Receiver 的多 Sender 共享 type,减少配对计算
- **碰撞标签精确**: 避免"接受所有标签"的 Receiver(性能 + 行为双损)
- **本地 Avatar 参数**: 仅在需要本地反应时启用

---

## 房间限制(World 端)

来自 `world/scene-components/index.md`:

> **FACT** (验证于 2025-03-07): 当前加载的 World 中:
> - **PhysBone 限制**: 1024 个 active 的 PhysBone + PhysBone Collider
> - **Contact 限制**: 1024 个 active 的 Contact Sender + Contact Receiver
>
> 超过上限的组件将被禁用,直到现有组件被禁用或销毁。
> SDK 会显示警告,但只要在运行时不超过 1024 个,警告可忽略。

> ⚠️ **1024 限制是世界运行时**,而 4096 是 SDK 编译/统计上限,二者不冲突。

---

## Debug View 7: PhysBone & Contact Overlay

来自 `world/udon/world-debug-views.md`:

> Debug View 7 高亮世界中**附近的 PhysBones 和 Contacts**。
> 用途:调试 PhysBone / Contact 组件行为不符合预期时。

启用方式: 在 World 中按 `F1` → 选 Debug View 7(需 ClientSim / World 已加载)。

---

## SDK 版本要求

| API | 最低 SDK |
|-----|----------|
| World `OnContactEnter` / `OnContactExit` | **3.10.0+** |
| World `VRCContactSender` / `VRCContactReceiver` | **3.10.0+** |
| Avatar 公开 Contact API | **2026.2.1 build+** |
| **Box 形状 Contact（Sender + Receiver）** | **SDK 3.10.4+** |
| **Box `Use Face Proximity`** | **SDK 3.10.4+** |
| **Advanced Parameters（正式发布）** | **SDK 3.10.6+**（3.10.6-beta 系列期间名称/行为有变：beta.3 由 "Advanced Proximity Parameters" 改名为 "Advanced Parameters"，并扩展到所有 Receiver Type） |
| **Material Contacts（Write to Shaders）** | **SDK 3.10.6+**（beta.3 起支持 Shader Parameter Override、修 `Cast Shadows` 关闭时的失效） |
| `Local Only` 下 256 组件上限 | 现行文档口径（未标注引入版本） |

> ⚠️ 旧 SDK(< 3.10.0)编译通过但**运行时静默忽略** Contact 事件。始终检查 `#if UDONSHARP` 或 SDK 版本条件。

---

## 实战模式

### 模式 1:门控触发(Avatar 进区域开门)

```csharp
// 挂在门附近空 GameObject 上的 Udon
public override void OnContactEnter(ContactReceiverInfo info) {
    if (info.isAvatar && info.player != null && info.player.isLocal) {
        // 只对本地玩家开门
        SendCustomNetworkEvent(NetworkEventTarget.All, nameof(OpenDoor));
    }
}
```

### 模式 2:Avatar 端反应(被 World 物体触碰时)

```
Avatar 上 VRCContactReceiver(collisionTags=["Fire"], onEnter=SetBool "IsOnFire"=true)
  → FX Layer: Transition "OnFire" → "Burn" 状态
  → 播放火焰动画 + Particle System
```

### 模式 3:多玩家协同(碰撞点 + 玩家归属)

```csharp
public override void OnContactEnter(ContactReceiverInfo info) {
    if (!info.isAvatar || info.player == null) return;

    // 记录被谁触碰
    _lastTouchedPlayerId = info.player.playerId;
    _lastTouchedPosition = info.contactPoint;

    // 同步给所有人(Manual Sync 模式)
    RequestSerialization();
}
```

---

## 与 PhysBone 的关系

| 维度 | Contact | PhysBone |
|------|---------|----------|
| 用途 | 接触检测(瞬时/持续) | 骨骼模拟(链式摆动) |
| 数据类型 | 触发事件 + 玩家信息 | 骨骼 transform + grab/release 事件 |
| 性能特征 | O(n) 形状检测 | O(链长) 弹簧模拟 |
| 配合使用 | 常见:Avatar 触发的 PhysBone 拉拽 | — |

> **最佳实践**: Avatar 全身 PhysBone + 关键部位 Contact(头/手/脚),配合 FX Layer 反馈。

---

## 已知坑

1. **没有 onStay/OnContactStay** — 不要等「持续接触回调」。用 `OnContactEnter` 置标志 + `OnContactExit` 清标志，在 `Update`（或 `SendCustomEventDelayedSeconds` 低频轮询）里读标志；且标志里不要做重活(网络同步、复杂逻辑)
2. **collisionTags 区分大小写** — `"Head"` ≠ `"head"`
3. **isAvatar 不等于 isLocal** — World 物体碰撞也会 isAvatar=false;需双重判断
4. **player 可能为 null** — Sender 在某些边界情况(null Sender / delayed join)下 player 为 null,需做空检查
5. **脱离范围不立即触发 onExit** — 物理引擎有脱节延迟,关键状态用 polling 而非事件

---

## Receiver Type 数值建议(2026-07-04 来自 yexcadocs)

> 来源:yexcadocs §Contacts(P2)
> 本节为社区实操经验补充。

| Receiver Type | 推荐参数类型 | 说明 |
|--------------|------------|------|
| **Constant** | Bool(True 触发)或 Float/Int(=1) | 持续接触时持续触发 |
| **OnEnter** | Bool(True 触发)或 Float/Int(=1) | 进入时触发一次 |
| **Proximity** | **必须是 Float**(0.0~1.0) | 高精度需 Sender 半径非常小 |

### Proximity 精度优化

- **必须用 Float 类型**(其他类型不支持)
- 高精度场景:需要 Sender 的 `radius` 设置非常小
- 典型应用:距离传感器(精确数值反馈)

---

## Box 形状与 Face Proximity（SDK 3.10.4 起，Avatar 侧）

> 来源：[FACT] 官方 [Contacts 文档](https://creators.vrchat.com/common-components/contacts/) + [Release 3.10.4](https://creators.vrchat.com/releases/release-3-10-4/)；复核 2026-11-10。

### 形状与尺寸

| 项 | 规则 |
|----|------|
| Shape Type | Sphere / Capsule / **Box**（3.10.4 起 Avatar 与 World 两侧都支持 Box） |
| Box 的 `Size` | 宽（X）/ 高（Y）/ 深（Z）**可独立设置** |
| 尺寸上限 | 半径最大 **3 m**；宽/高/深最大 **6 m**；挂在被缩放的物体上时，**上限按缩放后计算** |
| Sphere / Capsule | 只有 `Radius`（Capsule 另有沿 Y 轴含两端半球的 `Height`）生效 |
| Position / Rotation | 相对 Root Transform 的偏移与旋转 |

### 两种 Proximity 计算方式

| 模式 | 计算方式 | 何时用 |
|------|----------|--------|
| **默认（中心距离）** | Sender 表面到 Receiver **中心**的最近点 | 与旧版 Sphere/Capsule 行为一致 |
| **Use Face Proximity**（Box 专用） | Sender 表面到 Receiver **+Z 面**的最近点 | 需要"离贴面远近"而非"离中心远近"，如面部/胸口贴合的触发区 |

> ⚠️ `Use Face Proximity` **只对 Box 形状的 Receiver 提供**；Sphere/Capsule Receiver 无此项。

### 其他与 Box 相关的事实

- **3.10.4 已修复**：World 中与带偏移的 Sphere Sender 碰撞时 contact point 取值错误。
- 3.10.4 起，同一场景内多个 Avatar 时，Contacts/PhysBones 的 `Allow Self` / `Allow Others` 在 Play 模式下**按"每个 Avatar 各自是独立玩家"**判定（多 Avatar 场景调试不再互相串扰）。
- 3.10.6 起，Community 侧还补充了：Sphere Sender 极小半径的检测改进、极薄 Box Receiver 的接触检测/proximity 计算改进、无体积 Receiver 生成 proximity 值的行为改进。

### Collision Tag 与数量上限

| 约束 | 值 |
|------|-----|
| 单个 Contact 可挂标签数 | 最多 **16** 个 |
| 标签匹配 | 大小写敏感；Sender 与 Receiver **至少一对相同**才触发 |
| 单 Avatar 组件上限 | 默认上限内可上传；开启 `Local Only` 后**单个 Avatar 最多 256 个 Contact 组件**，超限无法上传 |
| 命名建议 | 自定义标签用 **PascalCase**，与内置标签（`Head` / `Body` / `Hand L`…）保持一致 |

---

## 高级接触接近度参数（Advanced Parameters，3.10.6 已正式发布）

> ✅ **已于 SDK 3.10.6 正式发布（2026-10-30）** —— 本节的旧 [PLANNED] 状态已作废。正式名称为 **Advanced Parameters**（beta.3 起由 "Advanced Proximity Parameters" 改名，因为它不再只对 Proximity 类型生效）。见下方「3.10.6 落地形态」。

**基础能力（长期有效）**：当接触接收器（Receiver）被至少一个**标签匹配**的接触发送器（Sender）命中时，可为虚拟形象动画控制器生成一个 **proximity（接近度）参数**，表示 Sender 距 Receiver 中心的接近程度；若 Receiver 是**盒体（box）**且启用 `Use Face Proximity`，则改为距 **+Z 面**的距离。

**历史预告（下表的"计划"口径已被 3.10.6 的正式实现取代，仅作对照用）**：

| 预告参数（2026-08-13 口径） | 说明 |
|----------|------|
| **Sender 类型（sender type）** | 标识接触发送器所属的内容类型 |
| **Sender 中点相对局部位置** | 接触发送器中点相对于接触接收器的**本地位置** |
| **相对速度与速率** | 接触发送器相对于接触接收器的速度与速率 |
| **接近度变化率** | 接近度值的改变速率 |
| **最多十六个布尔参数** | 每个对应 Receiver 使用的一个标签，标识 Sender 与 Receiver 共同拥有的标签（每标签一个 bool） |

> **FACT**（官方 3.10.6 文档）：若**多个 Sender 同时接触同一 Receiver**，上述值代表**接近度最高（highest proximity）**的那个 Sender；**3.10.6-beta.3 起**补充规则：若两个 Sender 的 proximity 都为 1，则改用**点间距（point separation）更小**（即靠得更近）的那个。

### 3.10.6 落地形态（正式参数名与语义）

> 来源：[FACT] 官方 [Contacts 文档](https://creators.vrchat.com/common-components/contacts/) · [Release 3.10.6](https://creators.vrchat.com/releases/release-3-10-6/)。参数名前缀 `{parameter}` 即你在 Receiver 上填的 Parameter 字段。

| 正式参数 | 类型 | 语义 |
|----------|------|------|
| `{parameter}_SenderType` | Int | Sender 归属：**0** 无 Sender / 无法识别；**1** 与本 Avatar 同一 Avatar；**2** 另一个 Avatar；**3** World；**4** 生成的 item |
| `{parameter}_SenderLocalPositionX/Y/Z` | Float | Sender **中点**相对 Receiver 的本地坐标（各自沿 Receiver 的 X/Y/Z 轴） |
| `{parameter}_SenderLossyScaleX/Y/Z` | Float | Sender 所属 transform 的 lossy world scale（beta.3 新增） |
| `{parameter}_SenderVelocityX/Y/Z` | Float | Sender 相对 Receiver 的速度分量 |
| `{parameter}_SenderVelocityMagnitude` | Float | 相对速度的**总速率** |
| `{parameter}_PointSeparation` | Float | Sender 与 Receiver **最近内点**之间的世界空间距离（米）；Sphere/Box 从中心量，Capsule 从中心线最近点量；**无接触时为 -1**；< 0.001 视为 0（beta.3 新增） |
| `{parameter}_ProximitySpeed` | Float | proximity 的**变化率**（每秒）。**仅当 Receiver Type = Proximity 时生成**；+0.001 ~ -0.001 之间视为 0 |
| `{parameter}_CollisionTag_{tag}` | Bool | **Receiver 上定义的每个标签各生成一个**，表示该标签是否在本次接触中匹配 |

**关键行为（Refresh 级别，旧帖基本都没写）**：

1. **不再只对 Proximity 生效**：beta.3 起，Advanced Parameters 与 Material Contacts **无论 Receiver Type 是什么都会生效**（此前只有 Proximity 类型才有）。
2. **性能代价是显式的**：官方原文警告 `Advanced parameters can be expensive to calculate. Only enable the ones you need.` —— **逐项勾选**，不要全开。
3. **这些参数不需要进同步参数表**：与各 Receiver Type 的基础参数一样，**不需要**在 Avatar 的 Synced Parameters 里声明。
4. **命名冲突风险**：`{parameter}` 前缀重名会互相覆盖；配合第三方工具改写 Parameter 字段时（官方注明这类工具会连带改掉 Shader 属性名），要一起回归验证。

### Material Contacts（同期正式发布，Avatar 侧）

当 Receiver 勾选 **Write to Shaders** 时，它会**每帧**把数据写入最多 **4 个指定 Renderer** 的 MaterialPropertyBlock：

| 属性 | 内容 |
|------|------|
| `float4x4 _VRChatContact_Transforms_{parameter}[9]` | **最近 8 个** Sender 碰撞形状的世界变换，**第 9 项**是本 Receiver 根 transform（不是它的形状） |
| `float4 _VRChatContact_Data_{parameter}[9]` | 与上表平行的打包数据（playerId + 匹配到的 contact tag 位） |

- 用 SDK 自带 `Packages/com.vrchat.base/ShaderLibrary/VRCContacts.cginc` 里的 `VRC_CONTACT_DECLARE(property)` 宏声明，并提供 `VRC_CONTACT_TARGETCOUNT` / `VRC_CONTACT_TRANSFORM` / `VRC_CONTACT_WORLDPOS` / `VRC_CONTACT_PLAYERID` / `VRC_CONTACT_TAGS` 等访问宏。
- **Shader Parameter Override**：可让 shader 写入口使用与 Animator 参数不同的名字；若主 Parameter 留空而只填 Override，则**只写 shader 属性、完全不影响 Animator 参数**（beta.3 新增）。
- **beta.3 修复**：目标 Renderer 关闭 `Cast Shadows` 时 Material Contacts 失效的问题已修。

> ⚠️ 本页只覆盖 **Avatar 侧**描述；World 侧（`OnContactEnter` / `OnContactExit` 事件签名与 info 字段）归 `entities/world/` 与 `concepts/` 的页面。另：3.10.6 官方文档把事件参数类型写作 `ContactEnterInfo` / `ContactExitInfo`，而原页写的 `ContactReceiverInfo` 与之不一致（见下方勘误）。
>
> **动机**（官方）：响应社区反馈，帮助消除现有 avatar 中为推导这些信息而使用的复杂 workaround，并提升性能。

---

## 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| 1.1 | 2026-08-19 | 补 Avatar Contact 概述与性能基准 |
| 1.3 | 2026-11-10 | 复核 SDK 3.10.4 / 3.10.6：新增 Box 形状与 Use Face Proximity 专节、Advanced Parameters 由 PLANNED 改为已发布并补全 8 类正式参数名、新增 Material Contacts（Write to Shaders）Avatar 侧说明、SDK 版本要求表补 3.10.4/3.10.6 行；**纠正 `OnContactStay` 不存在**（概述、Receiver 字段表、回调代码块、已知坑共 4 处），新增「Enter 置标志 / Exit 清标志 / Update 读标志」的标准实现示例与显式纠错块；**`ContactReceiverInfo` 小节加官方对照**（标题标注不符 + 官方 `ContactEnterInfo`/`ContactExitInfo` 签名表 + 字段逐项对照 + 三个查无实据字段的替代做法 + 建议写法），旧表保留以兼容历史引用；补 `{parameter}_SenderType` 0-4 编码的官方原文锚点与生效条件 |

---

## 相关文档

- **World Dynamics API**:[[entities/api/dynamics]](SDK 3.10.0+ 完整说明)
- **PhysBone 限制**:[[entities/avatar/avatar-dynamic-bone-limits]]
- **Avatar 性能基准**:[[entities/avatar/performance-benchmarks]]
- **Quest 兼容性**:[[concepts/rules/quest-constraints]]
- **lilToon 接触功能(Shader 端)**:[[entities/avatar/shader/liltoon/fur]]
- **原生公告板 API 类（同期预告）**:[[../api/vrc-billboard.md]]

---

## 相关页面

[[../api/dynamics.md]] · [[avatar-dynamic-bone-limits.md]] · [[optimization-guide.md]] · [[performance-benchmarks.md]] · [[../world/scene-components/index.md]] · [[../world/udon/world-debug-views.md]] · [[../api/vrc-billboard.md]]
