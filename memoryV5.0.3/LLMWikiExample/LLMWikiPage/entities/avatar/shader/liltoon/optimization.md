---
title: "lilToon 优化指南"
category: avatar
subcategory: shader
knowledge_level: applied
status: active
source: 本地知识库整理
source_type: official
version: 1.1
upstream_version: "v2.4.1 (2026-10-20)"
last_review: 2026-11-10
confidence: Medium
tags:
  - avatar
  - shader
  - liltoon
aliases:
  - lilToon
  - "lilToon 优化指南"
related:
  - basic-settings.md
  - color-settings.md
  - render-modes.md
  - troubleshooting.md
type: entity
created: 2026-06-21
sources: 本地知识库整理
updated: 2026-11-10
---
# lilToon 优化指南

---

## 优化概述

lilToon 提供多种优化手段：
- 自动着色器优化
- 纹理烘焙
- 版本转换
- 构建尺寸削减

---

## 纹理烘焙 (Bake)

### 烘焙选项

| 选项 | 说明 |
|------|------|
| **全て焼き込み** | 主色+校正+图层2&3 合并 |
| **色調補正を焼き込み** | 仅烘焙色调校正 |
| **レイヤー2を焼き込み** | 主色+图层2 合并 |
| **レイヤー3を焼き込み** | 主色+图层3 合并 |

### 烘焙优势

```
✓ 减少运行时计算
✓ 简化材质复杂度
✓ 跨着色器兼容性
✓ 减少纹理采样
```

### 使用建议

| 场景 | 建议 |
|------|------|
| 最终发布 | 全量烘焙 |
| 迭代开发 | 按需烘焙 |
| 跨着色器迁移 | 色调烘焙优先 |

---

## 版本转换

### lilToonLite 转换

| 源 | 目标 | 优势 |
|----|------|------|
| 通常版 | Lite 版 | 减少负荷 |

### lilToonMulti 转换

| 源 | 目标 | 说明 |
|----|------|------|
| 通常版 | Multi 版 | 使用 Shader Keyword |

> ⚠️ Multi 版可能导致 Build 尺寸增大

### MToon (VRM) 转换

| 源 | 目标 | 说明 |
|----|------|------|
| lilToon | MToon | VRM 兼容性 |

> ⚠️ 参数不完全兼容，视觉效果可能变化

---

## 构建尺寸优化

### Shader 设置优化

| 选项 | 说明 |
|------|------|
| **ForwardAddパスで影を有効化** | ForwardAdd 阴影 |
| **ForwardAddパスを使用** | ForwardAdd Pass |
| **頂点ライトを使用** | 顶点光 |
| **ライトマップを使用** | 灯光贴图（VRChat 不需要） |

### 未使用属性移除

```
Assets/lilToon/[Material] Remove unused properties
```

### 2.3.x / 2.4.x 与优化相关的变更

| 版本 | 变更 | 对优化的影响 |
|------|------|-------------|
| 2.3.4 | URP 下附加灯（Additional Light）移到像素着色器 | URP 项目里附加灯的光照质量/开销变化，顶点光方案不再等价 |
| 2.4.0 | 支持 VRC Light Volumes 3.0（最小实现）；修正 UI Toolkit 场景构建崩溃；修正 Multi 版轮廓线 Render Queue | 用了 VRCLV 3.x 的 World 里材质亮度由新版计算；构建期崩溃属稳定性修复 |
| 2.4.1 | 修正 light probe 亮度 | 依赖探针照明的场景升级后亮度会校正，重新检查 Avatar 整体曝光 |

> 证据：https://github.com/lilxyzw/lilToon/blob/master/Assets/lilToon/CHANGELOG.md

---

## 纹理导入设置

### 忽略的设置

以下设置可能被忽略：

| 设置 | 说明 |
|------|------|
| **Wrap Mode** | 纹理环绕模式 |
| **Filter Mode** | 采样过滤模式 |

### 固定 SamplerState

| 纹理类型 | SamplerState |
|---------|--------------|
| グラデーションマップ | sampler_linear_clamp |
| 大部分遮罩 | sampler_linear_repeat |
| 形状遮罩 | sampler_linear_clamp |

---

## 性能优化建议

### 渲染模式选择

```
不透明 > カットアウト > 半透明 > 高负荷模式
```

### 遮罩使用

| 建议 | 说明 |
|------|------|
| 合并遮罩 | 减少纹理数量 |
| 合理通道 | 充分利用 RGBA |
| 烘焙遮罩 | 减少采样 |

### 材质数量

| 建议 | 说明 |
|------|------|
| 合并相似材质 | 减少 Draw Call |
| 使用图层系统 | 减少材质数量 |

---

## 编辑器工具

### 菜单项

| 工具 | 用途 |
|------|------|
| **Refresh shaders** | 修复着色器错误 |
| **Remove unused properties** | 移除未使用属性 |
| **Run migration** | 执行版本迁移 |
| **Setup from FBX** | FBX 自动设置 |
| **Convert Gif to Atlas** | GIF 转图集（2.3.4 changelog 记有 "Added a built-in GIF parser"） |
| **Pixel art reduction** | 像素画降采样 |

### Fix Lighting

```
GameObject/lilToon/[GameObject] Fix lighting
```

修复多 Mesh 物体亮度不一致。

---

## 纹理命名规则 (Setup from FBX)

### 主纹理

命名规则：
- 包含材质名
- 非特殊名称

```
✓ face.png, texture_face.png
✗ face_outline_mask.png, face_smoothness.png
```

### 轮廓线遮罩

包含 `outline`：
```
✓ face_outline.png, face_outline_mask.png
✗ face_mask.png, face_ol_mask.png
```

### 阴影遮罩

包含 `shadow`/`shade` + `mask`/`strength`：
```
✓ face_shadow_mask.png, face_shadow_strength_mask.png
✗ face_shadow_color.png, face_shadow.png
```

### 排除关键词

```
mask, shadow, shade, outline, normal, bumpmap,
matcap, rimlight, emittion, reflection, specular,
roughness, smoothness, metallic, metalness,
opacity, parallax, displacement, height, ambient, occlusion
```

---

## 相关文档

- [基本设置](basic-settings.md) — 渲染参数
- [颜色设置](color-settings.md) — 图层系统
- [渲染模式](render-modes.md) — 性能对比
- [故障排除](troubleshooting.md) — 常见问题

---

## 相关页面

[[basic-settings.md]] · [[color-settings.md]] · [[render-modes.md]] · [[troubleshooting.md]]

