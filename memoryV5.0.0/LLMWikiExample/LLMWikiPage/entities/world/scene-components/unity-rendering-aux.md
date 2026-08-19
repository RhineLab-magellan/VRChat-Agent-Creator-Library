---
title: "Unity 渲染/2D 杂项组件合并专页（LightProbeProxyVolume / LODGroup / Projector / SpriteRenderer / SpriteMask / BillboardRenderer）"
category: world
subcategory: scene-components
knowledge_level: applied
status: active
source: "Unity Manual 2022.3（6 个组件页）+ VRChat 官方白名单（creators.vrchat.com 实拉核验）"
source_type: official
version: 1.0
last_review: 2026-08-19
confidence: high
tags:
  - world
  - unity
  - rendering
  - mesh
  - light
  - probe
  - particle
  - camera
aliases:
  - LightProbeProxyVolume
  - LODGroup
  - Projector
  - SpriteRenderer
  - SpriteMask
  - BillboardRenderer
  - 渲染/2D 杂项组件合并页
related:
  - ../whitelisted-world-components.md
  - lightprobegroup.md
  - camera.md
  - meshrenderer.md
  - skinnedmeshrenderer.md
  - ../vrc-camera-settings.md
  - index.md
type: entity
created: 2026-08-19
sources: "https://creators.vrchat.com/worlds/whitelisted-world-components/ | https://docs.unity3d.com/2022.3/Documentation/Manual/class-LightProbeProxyVolume.html | https://docs.unity3d.com/2022.3/Documentation/Manual/class-LODGroup.html | https://docs.unity3d.com/2022.3/Documentation/Manual/class-Projector.html | https://docs.unity3d.com/2022.3/Documentation/Manual/class-SpriteRenderer.html | https://docs.unity3d.com/2022.3/Documentation/Manual/class-SpriteMask.html | https://docs.unity3d.com/2022.3/Documentation/Manual/class-BillboardRenderer.html"
updated: 2026-08-19
---
# Unity 渲染/2D 杂项组件合并专页（LightProbeProxyVolume / LODGroup / Projector / SpriteRenderer / SpriteMask / BillboardRenderer）

> **白名单地位（[FACT]，2026-08-19 拉取官方 https://creators.vrchat.com/worlds/whitelisted-world-components/ 原文逐项核验）**：6 个组件均**在**官方「Unity Components」清单内，**World 可用**。
> **本页为合并页**：6 个组件各自在 Unity 中角色差异大、但在知识库中均未单独立页，统一收录于此，逐节给出语义与「是否白名单」结论。
> **VRChat 专属限制**：6 个组件 VRChat 侧均**无专属专页/专属文档** → **[FACT] 白名单可用** + **[UNKNOWN] 无 VRChat 侧官方限定**。

---

## 白名单地位总述（逐项核验结论）

以官方白名单页**原文**为准（2026-08-19 拉取；官方页面标注 Last updated Nov 25, 2025）：

| 组件 | 官方白名单是否在列 | 证据（官方原文「Unity Components」清单条目） |
|---|---|---|
| **LightProbeProxyVolume** | ✅ 在列 | 原文含 `[LightProbeProxyVolume]` 条目 |
| **LODGroup** | ✅ 在列 | 原文含 `[LODGroup]` 条目 |
| **Projector** | ✅ 在列 | 原文含 `[Projector]` 条目 |
| **SpriteRenderer** | ✅ 在列 | 原文含 `[SpriteRenderer]` 条目 |
| **SpriteMask** | ✅ 在列 | 原文含 `[SpriteMask]` 条目 |
| **BillboardRenderer** | ✅ 在列 | 原文含 `[BillboardRenderer]` 条目 |

> ⚠️ **本地快照与官方原文差异（数据质量提示，不改变结论）**：知识库本地 [[../whitelisted-world-components.md|Allowlisted World Components]] 页在「Rendering」段**未列出 `LODGroup`**（本地快照抓取于 2026-06-15，Rendering 段从 BillboardRenderer 直接到 …，漏列 LODGroup）。但**官方线上原文明确包含 `LODGroup`**（见上面证据）。结论以官方在线为准：**LODGroup 在白名单、World 可用**。此差异已在本页如实标注，未改动本地快照文件。

> **Quest 例外提示**：官方白名单注明「The Android version of VRChat has some exceptions to this list」，Android/Quest 端个别组件可能例外——本页均按 PC 全量清单标注，Quest 侧须另查 [Quest Content Limitations](https://creators.vrchat.com/platforms/android/quest-content-limitations#components)，本页不臆测。

---

## 统一对照表（[FACT] Unity Manual 2022.3；VRChat 侧见各节）

| 组件 | 作用 / 效果 | 关键参数 | 默认/范围要点 |
|---|---|---|---|
| **LightProbeProxyVolume (LPPV)** | 为无法用烘焙光照图的大型动态对象（大型粒子、蒙皮网格）提供 3D 插值光照探针网格，引入**空间渐变**光照 | Bounding Box Mode（Automatic Local / Automatic World / Custom）、Proxy Volume Resolution、Probe Position Mode、Data Format | 分辨率每轴最终须为 2 的幂、**最大 32**；Data Format 默认 Float（32 位），可选 Half Float（16 位，性能更好）；需 Shader Model 4+ GPU 与 32/16 位浮点 3D 纹理 + 线性过滤；场景须有 LightProbeGroup |
| **LODGroup** | 管理 GameObjects 的**细节层次（LOD）**，对象离相机变远时减少渲染三角形数（性能优化） | LOD 各层 Renderer、Fade Mode（Cross Fade / Speed Tree）、Animate Cross-fading、Fade Transition Width、Lod Bias | LOD 阈值 = 对象屏幕空间高度占比百分比；默认启用 Animate Cross-fading（按时间淡入淡出）；Fade Transition Width 为 0.0–1.0 比例的过渡带；Lod Bias 影响过渡点 |
| **Projector** | 把材质投射到相交其视椎体的物体上：**贴花/弹孔、Blob 阴影、风格化光照、真实投影仪（相机→RenderTexture）** | Near/Far Clip Plane、Field Of View、Aspect Ratio、Orthographic、Orthographic Size、Material、Ignore Layers | 仅 Built-in Render Pipeline 兼容；材质须用 Projector/Light 或 Projector/Multiply Shader（Standard Assets）；Ignore Layers 默认 None |
| **SpriteRenderer** | 渲染 Sprite，控制其在场景中的视觉外观（2D/3D 通用） | Sprite、Color（顶点色，A=不透明度）、Flip、Material、Draw Mode、Sorting Layer、Order In Layer、Mask Interaction、Sprite Sort Point | Draw Mode 默认 Simple（另有 Sliced/Tiled，Tile Mode 默认 Continuous、Stretch Value 0–1）；Mask Interaction 默认 None；新 Sprite 默认材质 Sprites-Default（**不受光照**），要受光改 Default-Diffuse |
| **SpriteMask** | 隐藏或显示 Sprite（或一组 Sprite）的一部分 | Sprite、Alpha Cutoff、Range Start、Sorting Layer、Order in Layer、Range End（Mask All / Custom） | 只影响**带 Sprite Renderer 且其 Mask Interaction 设为参与**的对象；自身在场景中不可见 |
| **BillboardRenderer** | 渲染 **Billboard Asset**，作为远处复杂 3D 网格的简化替代（LOD 手段） | Billboard asset、Cast Shadows、Receive Shadows、Light Probes、Proxy Volume Override、Reflection Probes、Motion Vectors、Dynamic Occlusion | Dynamic Occlusion 默认开启；Light Probes 默认 Blend Probes；常用于 SpeedTree 导出与 LOD 末层 |

---

## 各组件小节

### 1. LightProbeProxyVolume（LPPV）

**白名单：[FACT] ✅ 在列，World 可用。**

**官方语义（[FACT] Unity Manual 2022.3）**：让使用**烘焙光照图之外**的大型动态对象（大型粒子系统、蒙皮网格等无法烘焙的对象）获得更多光照信息。默认情况下被探针照明的 Renderer 只从单个插值光照探针取光（无空间渐变）；LPPV 在包围体积内生成**一盒 3D 插值光照探针**，把球谐系数上传到 3D 纹理，渲染时采样，为对象加上**空间渐变环境光**。

- **关键属性**：`Bounding Box Mode`（Automatic Local 默认 / Automatic World / Custom）、`Proxy Volume Resolution`（分辨率每轴最终须为 2 的幂，**最大 32**）、`Probe Position Mode`（Cell Corner / Cell Center，后者缓解穿墙漏光）、`Data Format`（Float 默认 / Half Float，半浮点线性采样更省性能）。
- **硬件/前置要求**：需 Shader Model 4+ GPU 与 3D 纹理（32/16 位浮点 + 线性过滤）；场景需通过 `Light Probe Group` 存在光照探针（见 [[lightprobegroup.md|LightProbeGroup]]）。
- **触发方式**：Renderer 的 `Light Probes` 属性设为 **Use Proxy Volume** 时才生效；同一 GameObject 或经 `Proxy Volume Override` 借用其他对象的 LPPV。Standard Shader 支持；自定义 Shader 用 `ShadeSHPerPixel`。
- **VRChat 侧**：**[FACT] 白名单可用** + **[UNKNOWN] 无 VRChat 专属限制文档**；光照经 VRChat 光照体系（烘焙 + LightProbeGroup）配合，见 [[lightprobegroup.md|LightProbeGroup]] 与 [[light.md|Light]] 相关光照页。本地无性能基准数值（[UNKNOWN]）。

### 2. LODGroup

**白名单：[FACT] ✅ 在列（官方在线原文含 LODGroup 条目；本地快照漏列，见上文差异提示），World 可用。**

**官方语义（[FACT] Unity Manual 2022.3）**：`LOD Group` 管理 GameObjects 的**细节层次**优化——对象离相机越远，渲染越简化的网格，减少三角形数。LOD 选择条上各层以彩色方块表示，层阈值用**对象屏幕高度占比百分比**表示；可增删 LOD 层、为每层指定 Renderer（通常是子对象）。

- **关键属性**：`Fade Mode`（Cross Fade / Speed Tree）、`Animate Cross-fading`（默认启用，按时间过渡；关闭后改用 `Fade Transition Width`，0.0–1.0 比例的过渡带）、`Lod Bias` 质量设置、`Recalculate Bounds` / `Recalculate Lightmap Scale`、`LODGroup.crossFadeAnimationDuration`（全局时间过渡时长）。
- **与渲染器关系**：LOD 各层挂 Mesh Renderer 或 Billboard Renderer（见 [[meshrenderer.md|MeshRenderer]]、本节 BillboardRenderer）；LOD 淡入淡出信息通过 `unity_LODFade.x` uniform 与 `LOD_FADE_PERCENTAGE` / `LOD_FADE_CROSSFADE` 关键字传至 Shader。
- **VRChat 侧**：**[FACT] 白名单可用**（官方在线清单确认）+ **[UNKNOWN] 无 VRChat 专属限制文档**。LOD 属性能优化手段，与本库 [[../performance-guide.md|Performance Guide]] / [[../occlusion-culling-guide.md|Occlusion Culling]] 目标一致，但无官方 World 侧定量基准（[UNKNOWN]）。

### 3. Projector

**白名单：[FACT] ✅ 在列，World 可用。**

**官方语义（[FACT] Unity Manual 2022.3）**：把材质投影到所有与其视椎体相交的物体上，用于**贴花（弹孔/污渍）、Blob 阴影、风格化光照、以及用另一台相机渲染到 RenderTexture 模拟真实投影仪**。

- **关键属性**：`Near Clip Plane` / `Far Clip Plane`（投影裁剪范围）、`Field Of View`（仅透视时用）、`Aspect Ratio`、`Orthographic`（启用则正交）、`Orthographic Size`（仅正交时用）、`Material`、`Ignore Layers`（默认 None）。
- **材质要求**：材质须用 **Projector/Light 或 Projector/Multiply** Shader（来自 Standard Assets）；Cookie / Falloff 纹理有 Clamp、无压缩、Alpha 通道等配置要求。
- **⚠️ 渲染管线**：官方明确「This workflow is compatible only with the **Built-in Render Pipeline**」。VRChat World 使用 Built-in Renderer（本库与 [[index.md|Scene Components 总览]] 场景一致），但这是官方对 Projector 的管线约束，须如实记录；若管线不符则可能不兼容（[INFERENCE]）。
- **VRChat 侧**：**[FACT] 白名单可用** + **[UNKNOWN] 无 VRChat 专属限制文档**。配合 [[camera.md|Camera]]（相机→RenderTexture→投影）与本库 RenderTexture 体系使用；无本地基准数值（[UNKNOWN]）。

### 4. SpriteRenderer

**白名单：[FACT] ✅ 在列，World 可用。**

**官方语义（[FACT] Unity Manual 2022.3）**：渲染 **Sprite** 并控制其在场景中的视觉外观，2D/3D 项目皆可用。新建 Sprite（GameObject > 2D Object > Sprite）时会自动带 Sprite Renderer。

- **关键属性**：`Sprite`、`Color`（顶点色；A 通道=不透明度）、`Flip`（X/Y，不翻转 Transform）、`Material`、`Draw Mode`（Simple 默认 / Sliced / Tiled；Tile Mode 默认 Continuous，另有 Adaptive + `Stretch Value` 0–1）、`Sorting Layer`、`Order In Layer`（同层内数字越高越后渲染越靠上）、`Mask Interaction`（None 默认 / Visible Inside Mask / Visible Outside Mask）、`Sprite Sort Point`（Center 默认 / Pivot）。
- **受光注意**：默认材质 **Sprites-Default 不受场景光照**；若要让 Sprite 受光照，需改用 **Default-Diffuse** 材质（[FACT]）。
- **与 SpriteMask 关系**：`Mask Interaction` 需设为参与，Sprite Renderer 才会与「SpriteMask」小节交互；排序用 `Sorting Layer` / `Order In Layer`（也可配合 Sorting Group）。
- **VRChat 侧**：**[FACT] 白名单可用** + **[UNKNOWN] 无 VRChat 专属限制文档**。World 内做 2D 面片/屏幕元素时可用；无本地性能基准（[UNKNOWN]）。

### 5. SpriteMask

**白名单：[FACT] ✅ 在列，World 可用。**

**官方语义（[FACT] Unity Manual 2022.3）**：**隐藏或显示**一个 Sprite 或一组 Sprite 的局部。**只影响带 Sprite Renderer 且其 `Mask Interaction` 设为参与（Visible Inside/Outside Mask）的对象**；Sprite Mask 自身在场景中不可见。

- **关键属性**：`Sprite`（用作蒙版的 Sprite）、`Alpha Cutoff`（透明/不透明过渡的判定阈值）、`Range Start`（蒙版开始作用的 Sorting Layer）、`Sorting Layer` / `Order in Layer`、`Range End`（默认 **Mask All**，可选 **Custom** 限定到指定 Sorting Layer/Order）。
- **作用范围控制**：默认影响其后所有较低的排序层；可用 Sorting Group 或 Custom Range 只影响特定 Sprite 组。
- **VRChat 侧**：**[FACT] 白名单可用** + **[UNKNOWN] 无 VRChat 专属限制文档**。与「SpriteRenderer」小节搭配做 UI/2D 裁切；注意本库 [[canvas.md|Canvas]]（世界 UI）走 uGUI Mask/RectMask2D 体系，与本 2D Sprite Mask 是不同机制（勿混用，[FACT] 依据两者官方语义不同）。

### 6. BillboardRenderer

**白名单：[FACT] ✅ 在列，World 可用。**

**官方语义（[FACT] Unity Manual 2022.3）**：渲染 **Billboard Asset**。Billboard 是一种 **LOD 手段**——复杂 3D 网格离相机很远时用简化的 2D Billboard 表示，不再渲染全细节。SpeedTree 等可导出 Billboard Asset，也可自行创建。

- **关键属性**：`Billboard`（指定要渲染的 Billboard Asset）、`Cast Shadows`（On/Off/Two Sided/Shadows Only）、`Receive Shadows`、`Light Probes`（Off / Blend Probes 默认 / Use Proxy Volume / Custom Provided）+ `Proxy Volume Override`、`Reflection Probes`（Off / Blend Probes / Blend Probes and Skybox / Simple）、`Motion Vectors`、`Dynamic Occlusion`（默认开启）。
- **与 LOD 关系**：官方文档 LODGroup 示例即以 Billboard Renderer 作为 LOD 末层；与「LODGroup」「LightProbeProxyVolume（Use Proxy Volume 时）」小节同属 LOD/性能渲染链。
- **VRChat 侧**：**[FACT] 白名单可用** + **[UNKNOWN] 无 VRChat 专属限制文档**。World 内作远处细节替代时可用；无本地性能基准（[UNKNOWN]）。

---

## VRChat 特定注意（汇总）

- **[FACT] 白名单地位**：6 个组件均**在**官方「Unity Components」清单内（官方在线原文逐项核验，2026-08-19）。
- **[FACT] 均无 VRChat 官方专页**：官方文档无针对这些组件的 World 侧专属说明，由 Allowlist + 对应渲染/光照体系覆盖。
- **[UNKNOWN]**：VRChat 侧对这几类渲染/2D 组件是否有限定（数量/性能/平台差异、LOD 生效方式、Projector 管线兼容）——官方未定论，本页不臆测。
- **[FACT 数据质量] LODGroup 本地快照漏列**：本地 [[../whitelisted-world-components.md|Allowlisted World Components]]（2026-06-15 抓取）Rendering 段漏列 LODGroup，官方在线页已含；本页以官方为准。若维护人员希望，可日后修订本地快照页（本稿未改动任何既有文件）。

## 相关页面

[[lightprobegroup.md|LightProbeGroup]] · [[camera.md|Camera]] · [[meshrenderer.md|MeshRenderer]] · [[skinnedmeshrenderer.md|SkinnedMeshRenderer]] · [[../vrc-camera-settings.md|VRCCameraSettings]] · [[../whitelisted-world-components.md|Allowlisted World Components]] · [[index.md|Scene Components 总览]]

> 说明：本页与 **TextMeshPro**（[[textmeshpro.md|TextMeshPro]]）**无关**——TextMesh 不是本页组件，勿错链；本页也不涉及 Canvas 的 uGUI 体系（除 SpriteMask 与 uGUI 机制差异对照，见上）。

## 源清单

- VRChat Allowlist（白名单地位）：https://creators.vrchat.com/worlds/whitelisted-world-components/
- VRChat Quest Content Limitations（Quest 例外）：https://creators.vrchat.com/platforms/android/quest-content-limitations#components
- Unity Manual Light Probe Proxy Volume：https://docs.unity3d.com/2022.3/Documentation/Manual/class-LightProbeProxyVolume.html
- Unity Manual LOD Group：https://docs.unity3d.com/2022.3/Documentation/Manual/class-LODGroup.html
- Unity Manual Projector：https://docs.unity3d.com/2022.3/Documentation/Manual/class-Projector.html
- Unity Manual Sprite Renderer：https://docs.unity3d.com/2022.3/Documentation/Manual/class-SpriteRenderer.html
- Unity Manual Sprite Masks：https://docs.unity3d.com/2022.3/Documentation/Manual/class-SpriteMask.html
- Unity Manual Billboard Renderer：https://docs.unity3d.com/2022.3/Documentation/Manual/class-BillboardRenderer.html
