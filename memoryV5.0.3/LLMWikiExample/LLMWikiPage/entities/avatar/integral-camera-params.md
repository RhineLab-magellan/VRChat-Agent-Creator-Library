---
title: "Integral 相机参数详解（Aperture/Zoom/Focus/Exposure/SS）"
category: avatar
knowledge_level: applied
status: active
source: 笔记/06-integral/06-相机参数调整.md + 08-对焦系统详解.md
source_type: community
version: 1.0
last_review: 2026-07-04
confidence: Medium
tags:
  - avatar
  - integral
  - camera-params
  - aperture
  - focus
  - exposure
aliases:
  - Integral 相机参数
  - Integral Aperture
  - Integral 对焦
  - Integral 摄影三要素
related:
  - ../world/vrc-camera-settings.md
  - ../world/performance-guide.md
type: entity
created: 2026-07-04
sources: 笔记/06-integral/06-相机参数调整.md + 08-对焦系统详解.md
updated: 2026-07-04
---

# Integral 相机参数详解

> **范围**: Integral 相机插件的所有参数（Aperture/Zoom/Focus/Exposure/SS）
> **主文档**: [Integral（SuzuFactory Booth）](https://suzufactory.booth.pm/items/4724145)（插件概述、安装、特殊效果等）
> **来源**: 笔记/06-integral/06 + 08

---

## 0. 速查表

| 想了解... | 跳转 |
|----------|------|
| Aperture 光圈机制（Integral 独特：抖动合成） | §1 |
| Zoom 焦距（6 预设 + Value） | §2 |
| Focus 对焦（MF/AF/Visual AF 三模式） | §3 |
| Exposure 曝光（与 ISO 无关） | §4 |
| SS 快门速度（Set + Tune） | §5 |
| 与 VirtualLens2 对比 | §6 |
| 与现实摄影三要素对比 | §7 |

---

## 1. Aperture 光圈

### 1.1 现实 vs Integral 对比

| 维度 | 现实相机 | Integral |
|------|---------|----------|
| 物理意义 | 镜头口径大小 | 决定景深 |
| 副作用 | 影响进光量 | ❌ **不影响画面亮度** |
| 实现方式 | 光圈叶片调整 | **"在镜头有效口径范围内变化相机位置进行多重曝光拍摄"** |

**关键引用**：
> Integral 为了模拟景深效果，采用了"在镜头有效口径范围内**变化相机位置**进行多重曝光拍摄"的方法。通过对这种"剧烈抖动"的画面长时间曝光，最终形成带有景深效果的照片。

### 1.2 重要约束

- **仅当 Composition 构图模式为 Normal 时该功能才会正常工作**
- Aperture 值越大 → 背景抖动越剧烈
- 值过大 → 会带动对焦对象一起抖动
- 取景器上看到"剧烈抖动"是**正常现象**，不是 bug

### 1.3 对焦对象交互

> 您用模型的手指触碰取景器，可以选择对焦对象。此时，您的对焦对象不会移动，而背景会抖动。

**实现机制**：
- 手指触碰取景器 → 记录对焦对象位置
- 多帧抖动时，**对焦对象的位置补偿**
- 累积曝光后 → 对焦对象清晰，背景抖动模糊

### 1.4 与 VirtualLens2 对比

| 维度 | Integral（抖动合成） | VirtualLens2（Shader 模拟） |
|------|---------------------|--------------------------|
| 实现 | 长曝光 + 相机位置变化 | 自有 DoF 模拟引擎 |
| 视觉效果 | "准确" 的物理模拟 | 高质量 + 稳定 |
| 性能 | 高（多帧累积） | 中等（Shader 计算） |
| 适用 | 长曝光创作 | 实时预览 |

---

## 2. Zoom 焦距

| 属性 | 值 |
|------|-----|
| 单位 | 35mm 全画幅等效 |
| 范围 | 6 个预设值 + 手动 Value 调整 |
| 显示 | 当前焦距会显示在取景器上 |
| 作用 | 焦距越大，视角越窄，照片像被放大 |

---

## 3. Focus 对焦（3 种模式）

### 3.1 模式对比

| 模式 | 简称 | 焦点设置方式 | 适用场景 |
|------|------|------------|---------|
| **MF（Manual Focus）** | 手动对焦 | 需要手动指定焦点位置 | 精确对焦固定对象 |
| **AF（Auto Focus）** | 自动对焦 | 左手食指**触摸取景器** | 一般对焦场景 |
| **Visual AF（视觉自动对焦）** | 视觉自动对焦 | 同 AF（左手食指触摸） | **半透明物体对焦** |

### 3.2 Visual AF 特别说明

**关键引用**：
> 注意：使用 Visual AF（视觉自动对焦）时，即使是 **ZWrite 为 Off 的半透明材质**（如一些半透明物体），也能够作为对焦目标。

| 维度 | AF | Visual AF |
|------|-----|-----------|
| 半透明材质（ZWrite Off） | ❌ 不可作为对焦目标 | ✅ 可作为对焦目标 |
| 完全透明材质 | ❌ 不可对焦 | ❌ 不可对焦 |
| 对焦精度 | 高 | **大多数情况下低于普通 AF** |
| 焦点设置方式 | 左手食指触摸取景器 | 左手食指触摸取景器 |

### 3.3 关键技术概念

| 概念 | 说明 |
|------|------|
| **ZWrite** | Z 缓冲写入，决定物体是否参与深度计算 |
| **ZWrite Off** | 关闭 Z 写入，常用于透明物体 |
| **半透明 vs 完全透明** | 半透明（Alpha < 1）有部分信息可被识别；完全透明（Alpha = 0）无法识别 |

### 3.4 焦点设置交互

**关键引用**：
> 焦点位置可通过用左手的食指**触摸取景器**来进行更改。

**操作流程**：
1. 启用 AF 或 Visual AF
2. 左手食指触碰取景器
3. 系统识别触碰位置对应的 3D 物体作为焦点
4. 相机对准该焦点

### 3.5 Pause AF 暂停对焦

在 Shoot 菜单中，**Pause AF** 是一个独立按钮：
- 停止 AF（自动对焦）功能
- **将焦点固定**
- 避免拍摄过程中焦点变动导致的画面问题

> 停止 AF（自动对焦）功能，并将焦点固定。可避免在拍摄过程中焦点变动导致的画面问题。

---

## 4. Exposure 曝光

**关键引用**：
> 在 Integral 中，曝光这个按钮直接影响取景**画面整体的亮度**。这与真实的相机成像不一样——**Integral 中没有 ISO 这个概念**，曝光的时间长短和光圈大小也不会影响画面的亮度。所以，您理解成在 PS 中直接调节亮度即可。

### 4.1 与现实相机对比

| 现实三要素 | Integral 对应 |
|----------|--------------|
| 光圈 | Aperture（仅景深） |
| 快门 | SS（影响长曝光累积） |
| ISO | ❌ 不存在 |
| 亮度调节 | Exposure（独立参数） |

---

## 5. SS 快门速度

| 功能 | 说明 |
|------|------|
| **Set 设定** | 直接设置快门速度 |
| **Tune 微调** | 对当前快门速度进行更精细的调整 |

---

## 6. Selfie 自拍翻转 + Auto Level 自动水平

### 6.1 Selfie 自拍翻转

翻转相机方向，就像手机的"启用前置摄像头"。

### 6.2 Auto Level 自动水平

**基础行为**：
> 启用后，相机会自动调整朝向，使相机的"上方"与世界坐标系的上方向一致。也就是说，照片始终保持垂直于地面的方向，无论你的身体姿势或头部方向如何。

**三个方向选项**：

| 选项 | 行为 |
|------|------|
| **Right（右）** | 启用 Auto Level 的同时，相机会**右旋一个角度**再进行对齐 |
| **Up（上）** | 以**正常方向**启用 Auto Level，不进行旋转 |
| **Left（左）** | 启用 Auto Level 的同时，相机会**左旋一个角度**再进行对齐 |

**取消**：若要取消 Auto Level 功能，**再次点击当前已启用的按钮**即可。

**重要发现**：Auto Level **实际上是 VRChat 原相机的 Anchor + 自动水平功能**。这是第三方相机插件的标准实现。

---

## 7. 摄影三要素对比

### 7.1 现实摄影三要素

```
光圈（Aperture） + 快门（Shutter Speed） + ISO（感光度） = 正确曝光
```

### 7.2 Integral 三要素

```
Aperture（仅景深） + SS（仅长曝光累积） + ❌ 无 ISO = 间接通过 Exposure 控制亮度
```

### 7.3 VirtualLens2 三要素

| 参数 | 单位 | 范围 | 同步 |
|------|------|------|------|
| Zoom | 35mm 等效焦距（mm） | "最小焦距" 到 "最大焦距" | 可远程同步 |
| Aperture | F 值（如 f/1.4） | "最小 F" 到 "最大 F" | 可远程同步 |
| Exposure | EV（Exposure Value） | "曝光范围" | 50% = 0EV 默认；可远程同步 |

### 7.4 三个工具对比

| 维度 | 现实 | Integral | VirtualLens2 | VRCLens |
|------|------|----------|--------------|---------|
| 光圈 | 物理口径 + 景深 + 亮度 | 仅景深（抖动合成） | 景深（Shader 模拟） | 景深 |
| 快门 | 时间 + 亮度 | 仅累积帧数 | N/A | N/A |
| ISO | 感光度 | ❌ 不存在 | N/A | 曝光补偿 |
| 曝光/亮度 | 受三要素共同影响 | **独立 Exposure 按钮** | EV 曝光补偿 | 曝光补偿 |
| 对焦 | MF/AF | MF/AF/Visual AF | AF (AI 脸/瞳) | AF (Avatar 检测) |

---

## 相关文档

### 内部知识库

- [Integral（SuzuFactory Booth）](https://suzufactory.booth.pm/items/4724145) - Integral 主文档（含兼容性、安装、特殊效果）
- [[entities/world/vrc-camera-settings]] - VRCCameraSettings（含 Anchor 模式）
- [[entities/avatar/modular-avatar]] - MA 必需依赖
- [[entities/avatar/vrcfury-reference]] - VRCFury 参考

### 外部资源

- **VirtualLens2 Features**: https://vlens2.logilabo.dev/en/about/
- **VRCLens 最新版**: https://forum.ripper.store/topic/17280/latest-version-of-vrclens

---

**最后更新**: 2026-07-04
**维护者**: Knowledge Curator Agent
**状态**: ✅ 活跃

---

## 相关页面

[Integral（SuzuFactory Booth）](https://suzufactory.booth.pm/items/4724145) · [[../world/vrc-camera-settings.md]] · [[../world/performance-guide.md]]
