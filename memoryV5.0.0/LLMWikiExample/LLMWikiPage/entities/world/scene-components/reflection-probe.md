---
title: "ReflectionProbe — 世界反射探针组件"
category: world
subcategory: scene-components
knowledge_level: applied
status: active
source: "Unity Manual 2022.3 + 本地反射探针调研页 + VRChat 官方白名单"
source_type: official
version: 1.0
last_review: 2026-08-19
confidence: high
tags:
  - world
  - rendering
  - reflection
  - probe
  - light
aliases:
  - ReflectionProbe
  - 反射探针组件
related:
  - ../whitelisted-world-components.md
  - lightprobegroup.md
  - camera.md
  - light.md
  - ../../world/reflection-probes.md
type: entity
created: 2026-08-19
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-ReflectionProbe.html | https://docs.unity3d.com/ScriptReference/ReflectionProbe.html | https://creators.vrchat.com/worlds/whitelisted-world-components/"
updated: 2026-08-19
---
# ReflectionProbe — VRChat 原生 Unity 反射探针组件

> **白名单地位**: Unity `ReflectionProbe` 在官方 [Allowlisted World Components](../whitelisted-world-components.md)「Unity Components (77) / Rendering」内，**World 可用**。
> **VRChat 专属限制**: 官方无反射探针专属专页；核心语义见 [[../../world/reflection-probes.md|反射探针与 VRChat World]] 详页与本文。注意 **LightProbeProxyVolume 仅登记（无独立专页）**。

---

## 核心功能（官方，[FACT]）

`ReflectionProbe`（`UnityEngine.ReflectionProbe`）像一台 360° 相机，向四周捕捉环境并存入 **Cubemap**，供使用反射材质（支持反射探针的 shader 的 Mesh Renderer/SkinnedMeshRenderer）的物体使用。官方语义来自 Unity Manual：[FACT] https://docs.unity3d.com/2022.3/Documentation/Manual/class-ReflectionProbe.html

- **Type（探针类型，[FACT]）**：**Baked**（编辑器烘焙静态 cubemap）、**Custom**（烘焙或手动指定，可选 Dynamic Objects）、**Realtime**（运行时更新，可响应动态物体）。
- **Refresh Mode / Time Slicing（[FACT]）**：Realtime 探针可 On Awake / Every Frame / Via Scripting 刷新；Time Slicing 将更新摊分到多帧（All Faces At Once=9 帧、Individual Faces=14 帧、No Time Slicing=单帧完成最贵）——多人世界里实时探针开销大。
- **运行时属性（[FACT]）**：**Importance**（多探针重叠排序优先级）、**Intensity**、**Box Projection / Box Size / Box Offset**（箱形投影）、**Resolution / HDR / Shadow Distance / Clear Flags / Culling Mask / Clipping Planes**（质量与范围）。

## 与相邻组件的关系

- **[[lightprobegroup.md|Light Probe Group]]**：光照探针组（LightProbeGroup）捕获**间接漫反射光照**；反射探针（ReflectionProbe）捕获**镜面反射**。两者互补，同为烘焙/动态光照体系。**LightProbeProxyVolume 在白名单中但本知识库暂无独立专页。**
- **[[camera.md|Camera]]**：反射探针本质是一次方向性相机渲染；多活动相机 + 实时探针叠加会显著增加渲染/纹理开销。
- **[[light.md|Light]]**：探针为反射提供镜面高光来源；点/聚光灯本身在反射探针中无物理实体，光源附近若无亮物体则反射中无对应高光（详见 [[../../world/reflection-probes.md|反射探针详页]]）。

## VRChat 特定注意

- **[FACT] 白名单地位**：ReflectionProbe 在官方白名单「Rendering」段内，World 可用。
- **[FACT] 性能影响**：实时探针每次刷新都是一次 cubemap 渲染，多活动相机 + 多实时探针会占用 RenderTexture/渲染开销；官方建议在多数场景用 Baked/Custom 探针并按需启用（见 [[../../world/reflection-probes.md|反射探针与 VRChat World]]）。
- **[UNKNOWN]**：VRChat World 内实时探针的具体数量/性能预算，以及是否有平台（Quest）差异——官方未单独为反射探针定论，本页不臆测数值。
- **[INFERENCE]**：结合社区语义（反射探针详页），封闭空间启用 Box Projection、Avatar 用腹部锚点做 Anchor Override 是常见实践，但属社区经验层级。

## 相关页面

[[lightprobegroup.md|Light Probe Group]] · [[camera.md|Camera]] · [[light.md|Light]] · [[meshrenderer.md|Mesh Renderer]] · [[../../world/reflection-probes.md|反射探针与 VRChat World]] · [[../whitelisted-world-components.md|Allowlisted World Components]] · [[index.md|Scene Components 总览]]

## 源清单

- Unity Manual Reflection Probe: https://docs.unity3d.com/2022.3/Documentation/Manual/class-ReflectionProbe.html
- Unity Scripting API ReflectionProbe: https://docs.unity3d.com/ScriptReference/ReflectionProbe.html
- VRChat Allowlist: https://creators.vrchat.com/worlds/whitelisted-world-components/
