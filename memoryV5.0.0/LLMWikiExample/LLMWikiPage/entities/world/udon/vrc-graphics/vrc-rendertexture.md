---
title: "VRCRenderTexture | 临时渲染纹理分配"
category: world
subcategory: udon
knowledge_level: applied
status: active
source: "VRC.SDKBase.VRCRenderTexture（SDK vrcsdk，暴露树 VRC.Rendering）+ VRChat Feedback #1304"
source_type: official
version: 1.0
last_review: 2026-08-19
confidence: Medium
tags:
  - world
  - udon
  - rendering
  - rendertexture
  - vrc-graphics
aliases:
  - "VRCRenderTexture"
  - "VRCRenderTexture.GetTemporary"
related:
  - index.md
  - asyncgpureadback.md
  - ../../../api/udon-type-exposure.md
  - ../../vrc-camera-settings.md
type: entity
created: 2026-08-19
sources: "https://www.matthewherber.com/VRCSDK-Documentation-Project/docs/doxygen/html/classVRC_1_1SDKBase_1_1VRCRenderTexture.html | VRChat Feedback #1304 | 暴露树 VRC.Rendering 段"
updated: 2026-08-19
---
# VRCRenderTexture | 临时渲染纹理分配

> **类**: `VRC.SDKBase.VRCRenderTexture`（Udon 暴露树 `VRC/VRCRenderTexture`，**100% 暴露**）
> **来源**: SDK 类参考 + [VRChat Feedback #1304](https://feedback.vrchat.com/udon/p/1304-vrcrendertexturegettemporary-crashing-when-antialiasing-parameter-is-the-de)

---

## 概述

Udon 提供对 Unity `RenderTexture.GetTemporary` / `ReleaseTemporary` 的封装，用于在运行时创建**临时渲染纹理**（脱屏渲染、镜面、摄像头输出、纹理处理等），避免常驻 `new RenderTexture` 的内存占用。

---

## API

### `GetTemporary`（静态）

多签名版（对应 Unity `RenderTexture.GetTemporary` 全参数版）：

```csharp
static RenderTexture GetTemporary(
    int width, int height, int depthBuffer = 0,
    RenderTextureFormat format = 7,       // 默认 ARGB32
    RenderTextureReadWrite readWrite = 0, // 默认 sRGB
    int antiAliasing = 0,                 // ⚠️ 见陷阱
    RenderTextureMemoryless memorylessMode = 0,
    VRTextureUsage vrUsage = 0,
    bool useDynamicScale = false
);

// 简写
static RenderTexture GetTemporary(RenderTextureDescriptor descriptor);
```

### `ReleaseTemporary`（静态）

```csharp
static void ReleaseTemporary(RenderTexture renderTexture);
```

> 另有 `OnReleaseTemporaryRenderTexture` 事件（释放时触发，可监听）。

---

## ⚠️ 已知陷阱：antiAliasing 默认 0 崩溃（重点）

> **FACT**: `GetTemporary` 的 `antiAliasing` 参数**默认 0**，但 Unity 只允许 **1 / 2 / 4 / 8**。直接使用默认值（或显式传 0）会抛异常崩溃：

```
RenderTextureDesc msaaSamples must be 1, 2, 4, or 8.
Parameter name: desc.msaaSamples
```

- **复现时段**: SDK 3.6.1、3.7.0 均已报告（[Feedback #1304](https://feedback.vrchat.com/udon/p/1304-vrcrendertexturegettemporary-crashing-when-antialiasing-parameter-is-the-de)，作者称"尚未修复"）。当前 SDK（3.10.x）情况以实测为准。
- **规避**: 显式传 `antiAliasing: 1`（或 2/4/8）；不要依赖默认。

**反例（崩溃）**:
```csharp
var t = VRCRenderTexture.GetTemporary(3840, 2160, 0, RenderTextureFormat.ARGB32, RenderTextureReadWrite.sRGB);
```

**正确（显式 MSAA=1）**:
```csharp
var t = VRCRenderTexture.GetTemporary(3840, 2160, 0, RenderTextureFormat.ARGB32, RenderTextureReadWrite.sRGB, 1);
```

---

## 使用注意

- **必须配对被释放**: `GetTemporary` 分配的纹理用完要用 `ReleaseTemporary` 归还（否则临时缓冲被回收后出现"Attempting to re-release not gotten as temp buffer"类似的隐患）。Unity 官方：temp buffer 必须用 `ReleaseTemporary` 释放，而非 `Release()`/`Destroy()`。
- **depth 与 Blit**: 配合 [[index|VRCGraphics]] 的 `Blit` 做脱屏渲染时，可设置 `renderTexture.depth = 0` 降低开销。
- **勿对同一纹理二次 release**（Unity 旧 bug 会崩溃）。

---

## 相关

- [[index|VRC Graphics 总览]] — Blit / Shader 渲染
- [[asyncgpureadback|VRCAsyncGPUReadback]] — 异步读回渲染结果
- [[../../vrc-camera-settings|VRCCameraSettings]] — 相机输出纹理场景
- [[../../../api/udon-type-exposure|Udon 类型暴露]] — 暴露率（100%）
