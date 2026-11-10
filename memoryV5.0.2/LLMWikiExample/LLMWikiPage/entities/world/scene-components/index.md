---
title: "Scene Components 总览"
category: world
subcategory: scene-components
knowledge_level: applied
status: active
source: 本地知识库整理
source_type: community
version: 1.1
last_review: 2026-08-17
confidence: Medium
tags:
  - misc
  - index
  - navigation
aliases:
  - "Scene Components 总览"
related:
  - ../whitelisted-world-components.md
  - textmeshpro.md
  - vrc-avatarpedestal.md
  - vrc-cameradolly.md
  - vrc-mirrorreflection.md
  - vrc-objectsync.md
  - vrc-portalmarker.md
  - vrc-scenedescriptor.md
  - vrc-station.md
  - vrc-enablepersistence.md
  - ../../api/pickups.md
  - ../../api/audio.md
  - ../../api/ui.md
  - ../../api/events-reference.md
  - ../../api/persistence.md
type: entity
created: 2026-06-20
sources: "https://creators.vrchat.com/worlds/components/ | https://creators.vrchat.com/worlds/whitelisted-world-components/"
updated: 2026-08-17
---
# Scene Components 总览

> VRChat Worlds 专用组件索引
>
> 来源: https://creators.vrchat.com/worlds/components/
> 本任务版本: 覆盖 9 个核心组件 (达到单任务 URL 上限)

---

## 概述

每个想导入 VRChat 的 Unity 场景都需要 `VRC_SceneDescriptor` 组件。VRChat Worlds SDK 提供了多种专用组件,允许用户与世界进行交互(拾取物体、在镜面中看到自己、坐下、使用 UI 等)。

完整组件白名单请参考 [Allowlisted World Components](../whitelisted-world-components.md)。

> **FACT**: 每个 World 必须有 **1 个且仅有 1 个** `VRC_SceneDescriptor`,缺失会导致 World 完全无法加载。

---

## 组件索引

| 组件 | 类名 | 文档 | 用途 |
|------|------|------|------|
| **TextMesh Pro** | `TMP_Text` (TextMeshProUGUI / TextMeshPro) | [textmeshpro.md](textmeshpro.md) | 高质量 2D/3D 文本 |
| **VRC Avatar Pedestal** | `VRC_AvatarPedestal` | [[entities/world/scene-components/vrc-avatarpedestal|vrc-avatarpedestal]] | 展示和切换 Avatar |
| **VRC Camera Dolly** | `VRC_CameraDollyAnimation` / `Path` / `Point` | [[entities/world/scene-components/vrc-cameradolly|vrc-cameradolly]] | 相机轨道动画 |
| **VRC Mirror Reflection** | `VRC_MirrorReflection` | [[entities/world/scene-components/vrc-mirrorreflection|vrc-mirrorreflection]] | 实时反射镜面 |
| **VRC Object Sync** | `VRCObjectSync` | [[entities/world/scene-components/vrc-objectsync|vrc-objectsync]] | 物理对象位置/旋转同步(核心) |
| **VRC Portal Marker** | `VRC_PortalMarker` | [[entities/world/scene-components/vrc-portalmarker|vrc-portalmarker]] | 传送门 |
| **VRC Scene Descriptor** | `VRCSceneDescriptor` | [[entities/world/scene-components/vrc-scenedescriptor|vrc-scenedescriptor]] | 场景描述符(World 必含) |
| **VRC Station** | `VRCStation` | [[entities/world/scene-components/vrc-station|vrc-station]] | 玩家固定座位 |
| **VRC Enable Persistence** | `VRCEnablePersistence` | [[entities/world/scene-components/vrc-enablepersistence|vrc-enablepersistence]] | 启用持久化 |

### Unity 原生组件（白名单内，2026-08-17 批次纳入）

> 依据官方 [Allowlisted World Components](../whitelisted-world-components.md)（Unity Components 段）逐组件调研；置信度与性能数据以各页 [FACT]/[INFERENCE]/[UNKNOWN] 标注为准。

#### 物理

| 组件 | 文档 | 用途 | 置信度 |
|------|------|------|--------|
| **Rigidbody** | [rigidbody.md](rigidbody.md) | 刚体物理（重力/碰撞/力）；VRC Pickup 依赖 | high |
| **BoxCollider** | [boxcollider.md](boxcollider.md) | 盒形碰撞体 | high |
| **CapsuleCollider** | [capsulecollider.md](capsulecollider.md) | 胶囊碰撞体 | high |
| **SphereCollider** | [spherecollider.md](spherecollider.md) | 球形碰撞体 | high |
| **MeshCollider** | [meshcollider.md](meshcollider.md) | 网格碰撞体（Convex 等） | medium |
| **CharacterJoint** | [characterjoint.md](characterjoint.md) | 角色关节（布娃娃基础） | medium |
| **ConfigurableJoint** | [configurablejoint.md](configurablejoint.md) | 可配置关节（万能关节） | medium |
| **HingeJoint** | [hinge-joint.md](hinge-joint.md) | 铰链关节（门/转轴/马达） | high |
| **FixedJoint** | [[entities/world/scene-components/fixed-joint|fixed-joint]] | 固定关节（刚性锁定/脆断） | high |
| **SpringJoint** | [[entities/world/scene-components/spring-joint|spring-joint]] | 弹簧关节（绳索/荡秋千） | high |
| **ConstantForce** | [[entities/world/scene-components/constant-force|constant-force]] | 恒定力/扭矩（推进器/漂浮） | high |
| **Cloth** | [[entities/world/scene-components/cloth|cloth]] | 布料模拟（需 SkinnedMeshRenderer） | medium |

#### 粒子/光照

| 组件 | 文档 | 用途 | 置信度 |
|------|------|------|--------|
| **ParticleSystem** | [particlesystem.md](particlesystem.md) | 粒子系统（World 侧限制/Quest 行为） | high |
| **ParticleSystemForceField** | [particlesystemforcefield.md](particlesystemforcefield.md) | 粒子力场（build 904 起 World 白名单） | high |
| **Light** | [light.md](light.md) | 光源（烘焙优先/粒子挂光陷阱） | high |
| **LightProbeGroup** | [lightprobegroup.md](lightprobegroup.md) | 光照探针组 | high |

#### 渲染

| 组件 | 文档 | 用途 | 置信度 |
|------|------|------|--------|
| **LineRenderer** | [linerenderer.md](linerenderer.md) | 线段渲染（笔刷/轨迹） | high |
| **TrailRenderer** | [trailrenderer.md](trailrenderer.md) | 拖尾渲染 | high |
| **MeshRenderer** | [meshrenderer.md](meshrenderer.md) | 网格渲染器 | high |
| **SkinnedMeshRenderer** | [skinnedmeshrenderer.md](skinnedmeshrenderer.md) | 蒙皮网格渲染器（BlendShape 等） | high |
| **ReflectionProbe** | [[entities/world/scene-components/reflection-probe|reflection-probe]] | 反射探针（镜面反射/Box Projection） | high |
| **渲染/2D 杂项合并页** | [[entities/world/scene-components/unity-rendering-aux|unity-rendering-aux]] | LPPV/LODGroup/Projector/SpriteRenderer/SpriteMask/BillboardRenderer | medium |
| **环境/文本杂项合并页** | [[entities/world/scene-components/unity-environment-components|unity-environment-components]] | Terrain/TerrainCollider/Tilemap/TilemapRenderer/Skybox/TextMesh | medium |

#### 音频

| 组件 | 文档 | 用途 | 置信度 |
|------|------|------|--------|
| **AudioSource** | [audiosource.md](audiosource.md) | 音频源（VRC_SpatialAudioSource 配置） | high |
| **AudioReverbZone** | [audioreverbzone.md](audioreverbzone.md) | 区域混响 | high |
| **AudioLowPassFilter** | [audiolowpassfilter.md](audiolowpassfilter.md) | 低通滤波（屏障遮挡感；AVPro 失效坑） | high |
| **Audio Filters 合并页** | [[entities/world/scene-components/unity-audio-filters|unity-audio-filters]] | Chorus/Distortion/Echo/HighPass/Reverb 五滤镜 | high |

#### 导航（⚠️ 时效风险）

| 组件 | 文档 | 用途 | 置信度 |
|------|------|------|--------|
| **NavMeshAgent** | [navmeshagent.md](navmeshagent.md) | 寻路代理（2022 兼容坑见页内 ⚠️） | medium |
| **NavMeshObstacle** | [navmeshobstacle.md](navmeshobstacle.md) | 动态导航障碍 | medium |
| **OffMeshLink** | [offmeshlink.md](offmeshlink.md) | 脱离网格连接（跳跃/落点） | medium |

#### 视频/UI

| 组件 | 文档 | 用途 | 置信度 |
|------|------|------|--------|
| **VideoPlayer** | [videoplayer.md](videoplayer.md) | 视频播放（VOD/URL 白名单） | high |
| **Canvas** | [canvas.md](canvas.md) | 世界 UI 画布（VRC_UIShape 交互条件） | high |
| **CanvasGroup** | [canvasgroup.md](canvasgroup.md) | 画布组（透明/交互开关） | high |
| **PlayableDirector** | [playable-director.md](playable-director.md) | Timeline 播放器（机关/过场/Animation+Audio 轨） | medium |

> ⚠️ NavMesh 三组件含 `[UNKNOWN/时效]` 警告（Unity 2022 NavMesh 兼容性未官方定论，须实测复核），见各页「VRChat 特定限制/注意」节。

### 本任务未覆盖的组件(后续任务处理)

- `VRC_Pickup` (详见 [../../api/pickups.md](../../api/pickups.md))
- `VRC_SpatialAudioSource` (详见 [../../api/audio.md](../../api/audio.md))
- `VRC_UIShape` (详见 [../../api/ui.md](../../api/ui.md))
- `VRC_PhysBone` / `VRC_PhysBoneCollider` / `VRC_PhysBoneRoot` (详见 [../../avatar/physbones.md](../../avatar/physbones.md))
- `VRC_ContactSender` / `VRC_ContactReceiver` (Avatar 域)
- `VRC_Constraints` (Avatar 域,详见 [../../avatar/vrc-constraints.md](../../avatar/vrc-constraints.md))

#### Unity 原生组件缺口登记（2026-08-19 更新，官方白名单 77 项逐项核查）

> 均已列入 [[../whitelisted-world-components.md|Allowlisted World Components]]「Unity Components (77)」；以下为**已建专页**或**待补/合并**状态。

- ✅ **已建合并页**: Constraints 6 → [[entities/world/scene-components/unity-constraints|unity-constraints]]；音频滤镜 5 → [[entities/world/scene-components/unity-audio-filters|unity-audio-filters]]；渲染/2D 6（LPPV/LODGroup/Projector/SpriteRenderer/SpriteMask/BillboardRenderer）→ [[entities/world/scene-components/unity-rendering-aux|unity-rendering-aux]]；环境 6（Terrain/TerrainCollider/Tilemap/TilemapRenderer/Skybox/TextMesh）→ [[entities/world/scene-components/unity-environment-components|unity-environment-components]]
- ✅ **已建简页**: Camera → [camera.md](camera.md)（受 [[../../world/vrc-camera-settings]] 限制）
- ✅ **已建专页（P1 批，2026-08-19）**: Cloth → [cloth.md](cloth.md)、ReflectionProbe → [reflection-probe.md](reflection-probe.md)、PlayableDirector → [playable-director.md](playable-director.md)、HingeJoint → [hinge-joint.md](hinge-joint.md)、FixedJoint → [fixed-joint.md](fixed-joint.md)、SpringJoint → [spring-joint.md](spring-joint.md)、ConstantForce → [constant-force.md](constant-force.md)
- ✅ **已由 API 域覆盖**: `Animator`（[[../../api/animator]]）
- 📌 **VRCBillboard 是 VRChat 原生 API 类（非 Unity 组件）**: 2026-08-13 官方预告的原生公告板（`VRCBillboard.Register/Unregister`，面向本地玩家朝向），已建专页 [[../../api/vrc-billboard.md]]；**勿与 Unity 原生 `BillboardRenderer`（面向摄像机十字片渲染，已并入 [unity-rendering-aux.md](unity-rendering-aux.md)）混淆**。二者机制与用途不同。
- 🎯 **覆盖合计 42 项 · 仅登记 35 项**（官方 Unity Components 77 = 42 已覆盖 + 35 仅登记）
- 📌 **待补（P2，仅登记 14 项）**: `WheelCollider`/`Tree`/`WindZone`/`OcclusionArea`/`OcclusionPortal`/旧粒子（`EllipsoidParticleEmitter`/`MeshParticleEmitter`）/`FlareLayer`/`Halo`/`LensFlare`/`CanvasRenderer`/`RectTransform`/`Grid`/`SortingGroup`/`Transform`

---

## World PhysBone 与 Contact 限制

> **FACT** (来自官方文档,验证于 2025-03-07): 当前加载的 World 中:
> - **PhysBone 限制**: 1024 个 active 的 PhysBone + PhysBone Collider
> - **Contact 限制**: 1024 个 active 的 Contact Sender + Contact Receiver
>
> 超过上限的组件将被禁用,直到现有组件被禁用或销毁。
> SDK 会显示警告,但只要在运行时不超过 1024 个,警告可忽略。

---

## Inspector 通用提示

- 所有 Scene Component 都通过 Unity Inspector 暴露配置
- UdonSharp 中通过 `GetComponent<T>()` 引用,例如:
  ```csharp
  [SerializeField] private VRCStation station;
  [SerializeField] private VRCObjectSync objectSync;
  [SerializeField] private VRC_PortalMarker portal;
  ```
- 多数组件提供 Udon 事件回调(Networked/本地),可在 `UdonSharpBehaviour` 子类中重写

---

## 跨页引用

- [VRCSceneDescriptor 与 VRCObjectSync 协作](vrc-scenedescriptor.md#object-prefabs)
- [VRCObjectSync 与 Manual Sync 区别](vrc-objectsync.md#与-manual-sync-区别)
- [VRCStation 事件回调](../../api/events-reference.md#station-事件)
- [VRCEnablePersistence 持久化 API](../../api/persistence.md)
- [VRC_Pickup 与 VRCObjectSync](../../api/pickups.md)
- [VRC_SpatialAudioSource 3D 音频](../../api/audio.md)

---

## 相关模式(在 `patterns/` 中)

- 物理对象所有权自动转移 → 见 `patterns/collision-ownership-transfer.md` (引用 vrc-objectsync)
- 座位事件驱动逻辑 → 见 `patterns/station-event-driven-logic.md` (引用 vrc-station)
- 自定义传送门 → 见 `patterns/dynamic-portal-marker.md` (引用 vrc-portalmarker)
