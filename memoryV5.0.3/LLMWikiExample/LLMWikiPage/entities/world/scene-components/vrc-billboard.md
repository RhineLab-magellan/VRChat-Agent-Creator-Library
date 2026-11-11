---
title: "VRC Billboard"
category: world
subcategory: scene-components
knowledge_level: applied
status: active
source: https://creators.vrchat.com/worlds/udon/vrc-billboard/
source_type: official
version: 1.0
sdk_version: "3.10.6"
last_review: 2026-11-10
confidence: High
tags:
  - world
  - component
  - billboard
  - rendering
  - udonsharp
aliases:
  - "VRC Billboard"
  - "VRCBillboard"
  - vrc-billboard.md
related:
  - ../../api/vrc-billboard.md
  - vrc-cameradolly.md
  - vrc-station.md
  - ../udon/vrc-graphics/index.md
type: entity
created: 2026-11-10
sources: https://creators.vrchat.com/worlds/udon/vrc-billboard/
updated: 2026-11-10
---
# VRC Billboard

> SDK **3.10.6** 新增的世界组件（Billboarding 集中式实现）。
> 来源: [creators.vrchat.com · VRCBillboard](https://creators.vrchat.com/worlds/udon/vrc-billboard/)（页面 Last updated 2026-10-20）
> **本页范围**: 只写**世界端组件行为与踩坑**。Udon 侧的签名/参数权威页见 [[entities/api/vrc-billboard]]，本页不重复 API 签名表。

---

## 概述

Billboarding = 每帧让物体转向观察点。`VRCBillboard` 把 Transform 注册到 VRChat 的**集中式 billboard 系统**，客户端在**一次 pass** 里统一转向所有已注册物体——比自己写 Udon `Update` 循环划算，物体越多差距越大。

> **FACT**（官方文档）:
> - 客户端每帧旋转已注册的 Transform，直到你注销或销毁该 GameObject
> - billboard **只跟随本地玩家视角**（桌面=相机，VR=头部）；要指向别的东西请用 `VRCAimConstraint` / `VRCLookAtConstraint`

---

## 组件字段（Inspector）

| Inspector 字段 | 默认值 | 说明 |
|---|---|---|
| **Facing** | `Back`（-Z） | 用物体的哪一根**自身局部轴**去对准观察点 |
| **Allow Tilt** | `false` | 关闭时瞄准方向被压平 → 只绕 up 轴转（yaw-only），保持竖直；开启后还会俯仰，直接对准观察点 |
| **Smoothing Factor** | `0` | 转向缓动（秒）；≤0 每帧瞬转。数值越大越慢，每 `smoothingFactor` 秒约走完 63%；缓动**与帧率无关** |
| **Up From** | 空（= 世界 up） | 指定「哪个 Transform 的 up」作为上方向，决定 roll 与关闭 Allow Tilt 时的压平平面。坐载具/座椅上用，传座椅或载具 Transform |
| **Match Viewpoint Rotation** | `false` | 关：瞄准观察点**位置**（旋转随自身位置变化）；开：**复制视角自身旋转**（所有物体朝向一致，适合贴屏 UI） |

> Up From / Match Viewpoint Rotation 在 Inspector 里收在 **Overrides** 折叠区（多数 billboard 用不到）。
> 运行时修改这些字段等价于**重新注册**，变更在下一趟 billboarding pass 生效。

---

## Facing 取值

`facing` 选择物体自身的一根局部轴去对准观察点；枚举值与 `Vector3` 常量同名：

| Facing | 对准的轴 | 典型内容 |
|---|---|---|
| **Back**（默认） | 局部 -Z（`Vector3.back`） | Canvas(UI)、TextMeshPro、Quad、Plane、SpriteRenderer |
| **Forward** | 局部 +Z | 导入模型自带正面（建模在 +Z 侧） |
| **Up** | 局部 +Y | 画在物体顶面的内容 |
| **Down** | 局部 -Y | 画在底面的内容 |
| **Left** | 局部 -X | 画在左侧面的内容 |
| **Right** | 局部 +X | 画在右侧面的内容 |

> **判定技巧**：选中物体看 Scene 视图的蓝色 Z 箭头。想让玩家看到的那一面**朝箭头方向** → 用 `Forward`；箭头从那一面**背后穿出** → 用 `Back`。

---

## 世界端行为要点（实战）

### 1. 注册键是 Transform，最后注册者胜出

- 一个 Transform **只有一套参数**；重复注册 = **替换**先前参数。
- 两个脚本同时注册同一 Transform 且参数不同时，**后注册的生效**，且没有警告。
- 因此「谁负责这个物体」要在架构上定清楚，别让多个 Udon 脚本抢着注册同一 Transform。

### 2. 任意脚本都可以 Unregister

- `VRCBillboard.Unregister(transform)` 不区分是谁注册的，**任何脚本都能把别人注册的 billboard 注销掉**。
- 被别的脚本注销后，物体会**一直不 billboard**，直到原负责脚本重新注册。
- 销毁物体时**不需要**手动注销（系统会自动清理）；只有「物体仍然存在但想让它停下」时才需要 `Unregister`。

### 3. billboarding pass 跑在 Udon 的 PostLateUpdate 之后

- 它是每帧**最后一个**写已注册 Transform 旋转的东西。
- 在 `Update` / `LateUpdate` / `PostLateUpdate` 里读该 Transform 的旋转，拿到的是**上一帧**的值，不是玩家马上要看到的值。
- 需要「第二个物体跟着这个 billboard 转」时，**不要**复制它的旋转；给第二个物体**也注册一次** billboard，让它们在同一趟 pass 里各自解析。

### 4. 默认就是 yaw-only

- `allowTilt = false`（默认）时瞄准方向被压平，物体只水平转、始终竖直——文字/招牌/标签的正确选择。
- `allowTilt = true` 才会俯仰追玩家（适合 sprite、图标）。
- 打开 `Allow Tilt` 后压平平面由 `upFrom` 决定；`upFrom` 指向载具时，物体绕**载具的 up** 转而不是世界 up。

### 5. `upFrom` 传自身 = 当作空

> **FACT**: 把 billboard 物体自己传进 `upFrom` 会被视为空（世界 up），因为物体不能当自己的上方向参照。

---

## 两种用法

### A. 零代码：挂组件

1. `Add Component > VRChat > Worlds > VRC Billboard`
2. 组件在 `OnEnable` 自动注册、`OnDisable` 自动注销 → **开关 GameObject 即可开关 billboard**
3. 在 Inspector 设 Facing / Allow Tilt / Smoothing / Up From / Match Viewpoint Rotation

### B. 脚本注册（Udon Graph / UdonSharp）

UdonSharp 一行即可（参数全部可选，默认 = -Z、yaw-only、瞬转、世界 up、瞄准位置）：

```csharp
using VRC.SDK3.Components;

void OnEnable()
{
    VRCBillboard.Register(transform);            // 文字/招牌：竖直、yaw-only
    // VRCBillboard.Register(transform, allowTilt: true);                  // sprite：追脸
    // VRCBillboard.Register(transform, upFrom: Vehicle);                   // 载具上保持竖直
}

void OnDisable()
{
    VRCBillboard.Unregister(transform);
}
```

> **参数很多且默认值在前**，实际写法请用**命名参数**只覆盖需要的那几个；完整签名见 [[entities/api/vrc-billboard]]。

**Udon Graph 注意（官方明示的坑）**：
- 用 `VRCBillboard` 分类下的 **Register** 节点；端口不连 = 用默认值。
- **目标 Transform 不是节点所在物体时，必须同时连 `upFrom`**（连 `Const null` 表示世界 up）。原因是未连接的 Transform 端口会解析成「节点所在物体」，而不是参数默认值；目标=自己时 SDK 能识别自引用，目标≠自己时无法区分。

---

## 调试与验证

| 手段 | 说明 |
|---|---|
| **ClientSim Play Mode** | 不进世界即可验证：物体转向 ClientSim 相机并随移动持续跟随 |
| Scene 视图 | 确认是否真的每帧朝相机（注意 yaw-only 与 allowTilt 的差异） |
| 多脚本场景 | 检查是否有第二个脚本抢注册同一 Transform（后注册胜出，且无警告） |

> 不要在 `Update` 里自己 `transform.LookAt(camera)`——那是每物体一次 Udon 调用，物体一多就成性能黑洞，且与官方 pass 打架。

---

## 性能与限制

| 项 | 说明 |
|---|---|
| 成本模型 | 一次集中 pass 处理所有注册物体，比每物体自己的 Udon 更新循环便宜得多 |
| 适用规模 | 招牌/标签/血条/图标成片出现时优势最大 |
| **不支持指定目标** | 只能跟**本地玩家**视角；要看向别的物体用约束组件 |
| **无网络同步** | billboard 是本地表现；不同玩家看到的朝向不同属正常 |
| Smoothing | 缓动与帧率无关，但数值过大在低帧率下仍会显得「拖」 |

---

## 常见坑

1. **多个脚本抢注册** → 后注册胜出，实际参数可能不是你以为的那套；用「单一负责者」原则。
2. **手动 Unregister 别人的注册** → 物体静默失去 billboard，很难查；排查时先搜全工程的 `Unregister`。
3. **读 billboard 的旋转做二次驱动** → 读到上一帧值且时序不可控；改为对第二物体单独 `Register`。
4. **UI Canvas 用 Forward** → 正面朝反，玩家看到的是背面；Canvas/TextMeshPro 默认走 `Back`。
5. **在载具/座椅上忘记 `upFrom`** → 载具倾斜时内容跟着歪；把座椅/载具 Transform 传进去。

---

## 相关文档

- [[entities/api/vrc-billboard]] — Udon API 权威页（签名、参数、返回值）
- [[entities/world/vrc-quality-settings]] — 世界质量设置（与视觉表现相关）
- [[entities/world/scene-components/vrc-cameradolly]] — 相机相关组件
- [官方文档 · VRCBillboard](https://creators.vrchat.com/worlds/udon/vrc-billboard/)
- [Release 3.10.6](https://creators.vrchat.com/releases/release-3-10-6/)