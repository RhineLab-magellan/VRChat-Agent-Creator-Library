---
title: "lilToon 颜色与阴影设置"
category: avatar
subcategory: shader
knowledge_level: applied
status: active
source: 本地知识库整理
source_type: official
version: 1.0
upstream_version: "v2.3.2 (2025-10-29)"
last_review: 2026-07-04
confidence: High
tags:
  - avatar
  - shader
  - liltoon
aliases:
  - lilToon
  - "lilToon 颜色与阴影设置"
related:
  - basic-settings.md
  - reflection-settings.md
  - render-modes.md
  - optimization.md
type: entity
created: 2026-07-04
sources: 本地知识库整理
updated: 2026-07-04
---
# lilToon 颜色与阴影设置

---

## メインカラー / 透明度 (Main Color / Alpha)

### 概述
基本颜色设置。透明模式下 Alpha 通道作为透明度使用。

### 参数

| 参数 | 说明 |
|------|------|
| **色 / 透明度** | 主纹理 + 颜色叠加 |
| **色相** | 色调偏移 (0-360°) |
| **彩度** | 饱和度调整 |
| **明度** | 亮度调整 |
| **ガンマ** | 对比度强调 |
| **グラデーションマップ** | 渐变映射色调校正 |
| **色調補正マスク** | 遮罩：R通道，控制校正区域 |
| **焼き込み** | 导出校正后的纹理 |

### 重要提示

> ⚠️ VRChat Safety 激活时纹理颜色会保留，但色调校正后的颜色不会保留。
> 建议编辑后导出纹理。

### 最终输出颜色

```
最终颜色 = 纹理颜色 × 灯光颜色
```

- 无灯光时：纹理原色
- 橙色灯光：纹理颜色 × 橙色

---

## 1.5 HSVG 复合属性 (P1-5, P2-5)

> **来源**: lilToon lts.shader 源码 + GitHub Issue #359
> **状态**: 2026-07-04 验证

### 1.5.1 完整属性列表

lilToon 提供**多个 HSVG Vector4 属性**（每个都是 H/S/V/G 4 维）:

| 属性名 | 默认值 | 用途 |
|--------|--------|------|
| `_MainTexHSVG` | (0, 1, 1, 1) | 主纹理 HSVG |
| `_Main2ndTexHSVG` | (0, 1, 1, 1) | 第 2 层纹理 HSVG |
| `_Main3rdTexHSVG` | (0, 1, 1, 1) | 第 3 层纹理 HSVG |
| `_OutlineTexHSVG` | (0, 1, 1, 1) | 轮廓线 HSVG |
| `_Outline2ndTexHSVG` | (0, 1, 1, 1) | 轮廓线第 2 层 HSVG |
| `_Outline3rdTexHSVG` | (0, 1, 1, 1) | 轮廓线第 3 层 HSVG |

> 源码: https://github.com/lilxyzw/lilToon/blob/master/Assets/lilToon/Shader/lts.shader

### 1.5.2 HSVG 4 维语义

| 分量 | 含义 | 范围 |
|------|------|------|
| X | Hue (色调) | 0-1（循环 0°-360°）|
| Y | Saturation (饱和度) | 0-1 |
| Z | Value (明度) | 0-1 |
| W | Gamma (伽马) | 0-1 |

### 1.5.3 ⚠️ 已知 Bug（P1-5）: VRCFury Toggle 不生效

> **来源**: https://github.com/lilxyzw/lilToon/issues/359 (2025-10-01 报告, Open)
> **状态**: **截至 2026-07-04 未解决**（11 个月）

**症状**:
- VRCFury Toggle → Material Property → 属性设为 `_MainTexHSVG`
- 即使滑块调到 100%，Renderer 也不变化

**根因【L5 推断】**:
- Unity 的 MaterialProperty 序列化在属性值等于 Shader 默认值时**不写入 Material 数据**
- 默认值 `(0, 1, 1, 1)` 与材质默认值一致 → Material 数据未持久化
- VRCFury 通过 Animator 修改时找不到对应数据

**Workaround**:
```
1. 打开 lilToon 材质 Inspector
2. 找到 "Main Color / Alpha" → 展开 Color Adjust
3. 任意 HSVG/Gamma 值改为非默认值（±0.0001 即可）
4. 现在 VRCFury 滑块能正常工作
```

**关键提示**:
- VRCFury Material Property 时，**X / Y / Z / W 都必须设为 1** 才能让 Hue 循环动画
- 仅设 X=1 不动其他维度 → Hue 不响应

### 1.5.4 与 Poiyomi 的关键差异

| 维度 | Poiyomi | LilToon |
|------|---------|---------|
| 属性名 | `Hue Shift` (右键复制) | `_MainTexHSVG` (必须手动) |
| 类型 | Float | Vector4 (HSVG) |
| 锁定机制 | Animated / Renamed (Poiyomi 特有) | 无内置锁定（需手动 Lock）|
| 已知 Bug | 标记 + Copy Animated Property Name | 默认值导致 VRCFury 不响应 |
| 整体复杂度 | 中 | 较高 |

---

## 阴影设置 (Shadow Settings)

### 概述
控制光照阴影的生成，支持 3 层阴影独立配色。

### 阴影系统特点

```
- 动漫涂 (0 模糊)
- 2D CG 涂 (模糊 + 强度变化)
- 3D CG 涂 (复杂阴影 + AO)
- 逆光/背光/边缘光组合
```

### 遮罩类型

| 类型 | 说明 |
|------|------|
| **强度** | 黑色 = 阴影弱，白色 = 阴影强 |
| **平面化** | 黑色 = 法线平展，白色 = 保持法线 |

### 主要参数

| 参数 | 说明 |
|------|------|
| **マスクと強度** | 阴影强度遮罩 (R通道) |
| **LOD** | 遮罩模糊量（MipMap） |
| **ぼかし量マスク** | RGB = 影色1/2/3 模糊遮罩 |
| **影色1/2/3** | 三层阴影颜色 |
| **範囲** | 阴影范围 |
| **ぼかし** | 阴影模糊量 (RGB=影色1/2/3) |
| **ノーマルマップ強度** | 法线贴图对阴影的影响 |
| **影を受け取る** | 接收其他物体阴影 |
| **境界の色** | 阴影边界叠加色（次表面散射效果） |
| **境界の幅** | 边界颜色宽度 |
| **コントラスト** | 阴影对比度 |
| **影色への環境光影響度** | 环境光对阴影色的影响 |
| **AO Map** | 遮罩阴影附着度 (RGB=影色1/2/3) |
| **Min/Max** | AO Map 重新映射范围 |

### 阴影计算公式

```hlsl
// 阴影色计算
阴影色 = (纹理阴影色 ? 纹理阴影色 : 主色) × 阴影色 × lerp(1.0, 主色, コントラスト)
最终阴影色 = lerp(阴影色, 主色, 环境光 × 影色への環境光影響度)

// 阴影混合
影色2/3结果 = lerp(影色3, 影色2, 3影光照)
全影结果 = lerp(影色2/3结果, 影色1, 2影光照)
最终 = lerp(全影结果, 主色, 1影光照)
```

---

## リムシェード (RimShade)

### 概述
轮廓阴影，与 RimLight 相反，用作阴影效果。

### 参数

| 参数 | 说明 |
|------|------|
| **色 / マスク** | 阴影色 + 遮罩 (R通道) |
| **ノーマルマップ強度** | 法线影响 |
| **範囲** | 阴影范围 |
| **ぼかし** | 模糊量 |
| **リムライトの細さ** | 轮廓细度（对比度） |

---

## 発光設定 (Emission)

### 概述
自发光效果，如 LED、灯光等。

### 参数

| 参数 | 说明 |
|------|------|
| **色** | 发光颜色（可设 UV） |
| **マスク** | 发光区域遮罩 (RGBA) |
| **点滅の強さ** | 闪烁强度 |
| **点滅タイプ** | Off=平滑, On=锐利 |
| **点滅の速度** | 闪烁速度 |
| **点滅のズレ** | 闪烁时间偏移 |
| **グラデーション** | 时间颜色渐变 |
| **視差の強さ** | 视角 UV 偏移 |
| **蛍光** | 仅暗处发光（暗室眼神光效） |

---

## メインカラー2nd・3rd (Layer System)

### 概述
图层系统，叠加颜色用于贴花或细节纹理。

### 参数

| 参数 | 说明 |
|------|------|
| **テクスチャ** | 图层纹理 |
| **MSDFテクスチャ** | 矢量图纹理 |
| **UV Mode** | UV0-3 / MatCap |
| **デコール化** | 作为贴花使用 |
| **マスク** | 图层遮罩 (R通道) |
| **ライティングを適用** | 应用灯光亮度 |
| **合成モード** | 通常/叠加/屏幕/乘法 |
| **焼き込み** | 导出图层 |
| **開始/終了距離** | 距离淡入淡出 |
| **Tiling/Offset/角度** | UV 控制 |

### 贴花模式附加参数

| 参数 | 说明 |
|------|------|
| **ミラーモード** | UV 单侧显示 |
| **複製モード** | UV 对称 |
| **X/Y座標** | 贴花位置 |
| **X/Y軸サイズ** | 贴花大小 |
| **フレーム数/FPS** | GIF 动画帧数/速度 |

---

## アルファマスク (Alpha Mask)

### 概述
灰度透明度遮罩，替代主纹理 Alpha 通道。

### 参数

| 参数 | 说明 |
|------|------|
| **アルファマスク** | None/替换/乘算 |
| **Invert** | 反转遮罩 |
| **Transparency** | 透明度 |
| **Cutoff** | 透明度阈值 |
| **Scale/Offset** | 遮罩强度调整 |
| **焼き込み** | 应用到主纹理 |

### 重要提示

> ⚠️ VRChat Safety 激活时 Alpha Mask 会被忽略。

---

## 相关文档

- [基本设置](basic-settings.md) — 基础参数
- [反射设置](reflection-settings.md) — 光泽/反射
- [渲染模式](render-modes.md) — 透明模式
- [优化指南](optimization.md) — 纹理烘焙

---

## 相关页面

[[basic-settings.md]] · [[reflection-settings.md]] · [[render-modes.md]] · [[optimization.md]]
