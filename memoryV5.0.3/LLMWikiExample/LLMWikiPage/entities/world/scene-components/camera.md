---
title: "Camera 组件参考（VRChat 白名单）"
category: world
subcategory: scene-components
knowledge_level: applied
status: active
source: "creators.vrchat.com/worlds/whitelisted-world-components/ + creators.vrchat.com/worlds/udon/vrc-graphics/vrc-camera-settings/"
source_type: official
version: 1.0
last_review: 2026-08-19
confidence: High
tags:
  - world
  - scene-components
  - camera
  - unity-native
  - whitelist
aliases:
  - "Unity Camera"
  - Camera 组件
related:
  - ../../world/vrc-camera-settings.md
  - index.md
  - ../whitelisted-world-components.md
  - ../../udon/vrc-graphics/index.md
  - ../udon/vrc-graphics/vrc-rendertexture.md
type: entity
created: 2026-08-19
sources: "https://creators.vrchat.com/worlds/whitelisted-world-components/ | https://creators.vrchat.com/worlds/udon/vrc-graphics/vrc-camera-settings/"
updated: 2026-08-19
---
# Camera 组件参考（VRChat 白名单）

> **白名单地位**: Unity `Camera` 在官方 [Allowlisted World Components](../whitelisted-world-components.md)「Unity Components」内，**World 可用**。
> ⚠️ **限制**: Camera 受官方 VRCCameraSettings 体系约束（见 [vrc-camera-settings](../../world/vrc-camera-settings.md)）。世界内自建 Camera 用于脱屏渲染/画中画时须注意交互规则。

---

## 概述

`Camera`（`UnityEngine.Camera`）是 Unity 渲染世界的核心组件。在 VRChat World 中主要用于：
- **脱屏渲染**（Render Texture 输出 → 显示器/UI）
- **画中画 / 监控画面**（配合 `RenderTexture`）
- **自定义视口渲染**

**不为**替代玩家主相机——玩家视角由 VRChat 控制（详见 [[../../world/vrc-camera-settings.md|VRCCameraSettings]]：ScreenCamera / PhotoCamera、CameraMode 枚举）。

---

## VRChat 特定注意

- **多相机开销**: 每个活动 Camera + RenderTexture 都会增加渲染开销；结合 [[../udon/vrc-graphics/vrc-rendertexture.md|VRCRenderTexture]] 需用 `ReleaseTemporary` 释放。
- **Clear Flags / CullingMask**: 官方 VRCCameraSettings 暴露 `ClearFlags` / `BackgroundColor` / `CullingMask` 等可写属性（世界相机参照同规则）。
- **LayerCullDistances**: VRCCameraSettings 暴露该属性，控制各层裁剪距离。

---

## 相关

- [[../../world/vrc-camera-settings.md|VRCCameraSettings]] — VRChat 相机 API（CameraMode/可写属性）
- [[../udon/vrc-graphics/vrc-rendertexture.md|VRCRenderTexture]] — 临时渲染纹理分配
- [[../whitelisted-world-components.md|Allowlisted World Components]] — 白名单全集
- [[index.md|Scene Components 总览]]
