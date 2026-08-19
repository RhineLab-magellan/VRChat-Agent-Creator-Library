---
title: "VRChat Runtime Texture Compression"
category: rules
knowledge_level: applied
status: active
source: VRChat 2026.2.2 Release Notes
source_type: official
version: 1.0
last_review: 2026-08-13
confidence: High
tags:
  - rules
  - platform
  - optimization
  - android
  - vrchat
  - storage
aliases:
  - Runtime Texture Compression
  - 运行时纹理压缩
related:
  - "../../entities/world/udon/image-loading.md"
  - "../../entities/platform/android-development.md"
  - "../../entities/avatar/avatar-size-limits.md"
  - "performance-rules.md"
type: concept
created: 2026-08-13
sources:
  - https://docs.vrchat.com/docs/vrchat-202622
updated: 2026-08-13
---
# VRChat Runtime Texture Compression

## 1. 定义

**[FACT]** VRChat 2026.2.2 引入客户端侧 Runtime Texture Compression：使用 GPU-side compressor，将 Stickers、Prints、Emojis、User Icons 等图像压缩为 ASTC 或 BC7 格式，并异步执行。

来源：[FACT] [VRChat 2026.2.2 Release Notes](https://docs.vrchat.com/docs/vrchat-202622)

## 2. 技术影响

- 官方说明目标是节省约 75% 的原始图像 RAM/VRAM 占用。
- ASTC 主要对应移动/Android 路径，BC7 主要对应桌面 GPU 路径；具体格式选择由客户端实现决定。
- 异步压缩降低了把压缩工作直接放在交互主线程上的必要性，但不能推断所有图像都在同一时刻完成压缩。
- 该机制属于客户端资源管理，不是 Udon 可调用的压缩 API。

## 3. 创作者边界

- **[FACT]** 创作者无需在 UdonSharp 中调用该功能。
- **[INFERENCE]** 该机制可能降低用户端图像资源的运行时内存压力，但不改变 Avatar/World 上传前的资源预算、材质兼容性和平台性能等级要求。
- 不应以运行时压缩替代贴图尺寸、格式、MipMap、材质数量和 Quest 兼容性的构建期优化。

## 4. 与本地知识库的关系

本页与 `entities/world/udon/image-loading.md` 的 Udon 图片加载机制互补：前者描述客户端通用图像资源压缩，后者描述 Udon 侧图片加载行为。与 Avatar 大小限制和 Android 开发页面交叉阅读时，应区分上传/构建限制与运行时资源管理。

## 相关页面

[[../../entities/world/udon/image-loading.md]] · [[../../entities/platform/android-development.md]] · [[../../entities/avatar/avatar-size-limits.md]] · [[performance-rules.md]]