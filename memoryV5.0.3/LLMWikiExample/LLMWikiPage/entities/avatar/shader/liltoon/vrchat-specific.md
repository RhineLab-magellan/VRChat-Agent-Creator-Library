---
title: "lilToon VRChat 特定配置"
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
  - "lilToon VRChat 特定配置"
related:
  - basic-settings.md
  - color-settings.md
  - optimization.md
  - troubleshooting.md
type: entity
created: 2026-06-21
sources: 本地知识库整理
updated: 2026-11-10
---
# lilToon VRChat 特定配置

---

## Custom Safety Fallback

### 概述
VRChat Safety 系统激活时的着色器回退配置。

### 参数

| 参数 | 说明 |
|------|------|
| **Shader Type** | 着色器类型 |
| **Rendering Mode** | 透明模式 |
| **Facing** | 描画面（单/双面） |
| **Shading** | ToonStandard 阴影类型（Custom 时可选 Ramp 纹理） |

### 输出预览

`Result` 显示基于设置生成的标签，用于确定回退目标着色器。

---

## Safety 系统说明

### 默认行为

VRChat Safety 激活时：
- 使用 ToonStandard 着色器
- 不透明化处理

### 自定义回退

可通过设置自定义回退着色器：
```
Custom Safety Fallback > Shader Type > 自定义选择
```

---

## 注意事项

> ⚠️ 文档撰写时 ToonTransparent 不存在
> 回退目标为 Unlit/Transparent

---

## VRC Light Volumes（VRCLV）支持

| lilToon 版本 | VRCLV 支持程度 |
|--------------|----------------|
| 2.1.0 | 改为**逐像素**计算 VRCLV（毛发着色器仍按顶点计算，出于负载考虑）；新增调节 VRCLV 边缘光的属性 |
| 2.1.5 | **毛发的 VRCLV 计算移到像素着色器**（`Fur shader VRCLV calculation now performed in pixel shader`）；Shader optimization |
| 2.1.6 | VRCLV optimization；兼容 VRCLV 1.0.0；VRCLV 边缘光范围上限提高到 3 |
| 2.1.9 / 2.1.10 | 修复 VRCLV 边缘光 Border ≥ 1 不生效；Border 默认值改为 3 |
| **2.4.0 起** | **最小 VRCLV 3.0 支持**（"Minimal VRCLV 3.0 support"）；接 VRCLV v3 世界的最低门槛即此版本 |

> ℹ️ 2.4.0 的措辞是 "Minimal"，即先保证 VRCLV 3.x 的 World 里材质不崩、亮度大致正确；不是 3.0 全部新特性都实现。World 侧要完整使用 VRCLV 3.0.1 的新特性，仍以 VRCLV 官方包为准。
> 证据：https://github.com/lilxyzw/lilToon/blob/master/Assets/lilToon/CHANGELOG.md （2.4.0 / 2.1.10 / 2.1.9 / 2.1.6 / 2.1.0 条目）
> World 侧的 VRCLV 组件、v2→v3 迁移步骤与「兼容 Shaders」版本门槛一览见 [[entities/world/vrc-light-volumes]]（该页 v3 门槛表已同步标注 lilToon 2.4.0+）。

---

## VRChat 特定参数

### 距离裁剪取消

| 参数 | 说明 |
|------|------|
| **距離クリッピングキャンセラー** | 防止近裁剪导致的消失 |

> ℹ️ 仅在 Near Clip > 0.1 时生效

### VR 视差强度

多个效果包含 VR 视差控制：

| 效果 | 参数 |
|------|------|
| MatCap | VR時の視差の強さ |
| RimLight | VR時の視差の強さ |
| 闪光 | VR時の視差の強さ |
| 宝石 | VR時の視差の強さ |

---

## VRChat 亮度建议

### 亮度设置

| 参数 | 建议值 | 说明 |
|------|--------|------|
| **明るさの下限** | 0.0 - 0.2 | 防止过暗 |
| **明るさの上限** | 保持默认或略低 | 防止过曝 |
| **影色への環境光影響度** | 0.3 - 0.5 | 暗世界阴影控制 |

---

## VRChat 优化建议

### Avatar 容量

```
lilToon 自动优化着色器
Build 时移除未使用功能
```

### 纹理优化

| 优化项 | 方法 |
|--------|------|
| 未使用纹理 | 移除或 Bake |
| 纹理分辨率 | 合理尺寸（1024-2048） |
| 图层合并 | 使用 Bake 功能 |

### 渲染模式选择

| 场景 | 推荐模式 |
|------|---------|
| 皮肤 | 不透明 |
| 头发 | 半透明 |
| 服装 | 不透明/半透明 |
| 配件 | 根据需求 |

---

## 常见 VRChat 问题

### 阴影过强

**解决方案**：
- 降低 影色への環境光影響度
- 使用遮罩隔离面部阴影
- 使用 ライト方向のオーバーライド

### 亮度不一

**解决方案**：
- 对多 Mesh 对象执行 Fix lighting
- 统一材质设置
- 检查顶点灯强度

### Safety 表现异常

**解决方案**：
- 检查 Custom Safety Fallback 设置
- 测试不透明模式
- 验证纹理 alpha 通道

---

## 相关文档

- [基本设置](basic-settings.md) — 渲染参数
- [阴影设置](color-settings.md#阴影设置) — 阴影配置
- [优化指南](optimization.md) — 性能优化
- [故障排除](troubleshooting.md) — 常见问题

---

## 相关页面

[[basic-settings.md]] · [[color-settings.md]] · [[optimization.md]] · [[troubleshooting.md]]

