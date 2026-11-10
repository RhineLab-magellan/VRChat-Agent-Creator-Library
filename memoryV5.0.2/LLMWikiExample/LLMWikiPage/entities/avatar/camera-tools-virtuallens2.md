---
title: "VirtualLens2 — 子工具文档"
category: avatar
knowledge_level: applied
status: active
source: docs.vrcd.org.cn + vlens2.logilabo.dev + 分类审查归纳
source_type: community
version: 1.0
last_review: 2026-07-13
confidence: Medium
tags:
  - avatar
  - camera
  - virtual-lens2
  - non-destructive
aliases:
  - VirtualLens2
  - 虚拟镜头2
  - 虚拟镜头工具
  - VL2
related:
  - camera-tools.md
  - modular-avatar.md
  - vrcfury-reference.md
  - ../world/vrc-camera-settings.md
type: entity
created: 2026-07-13
sources: docs.vrcd.org.cn + vlens2.logilabo.dev + 分类审查归纳
updated: 2026-07-13
---

# VirtualLens2 — 子工具文档

> 本页承载 VirtualLens2 的安装、依赖、工作流与功能细节；总览对比与选型建议见 `camera-tools.md`。

## 1. 定位
- **平台**：PCVR 主体使用，支持 `Remote Mode / Remote Only Mode` 作为 Android 侧补充能力。
- **设计哲学**：**Non-Destructive**。
- **实现路径**：依赖 Modular Avatar + NDMF，在构建链路中注入相机功能，而不是直接长期污染 Avatar 源资源。

## 2. 稳定依赖门槛
| 项目 | 要求 |
|---|---|
| Unity | 2022 LTS |
| VRCSDK | 3.7.0+ |
| Modular Avatar | 1.8.0+ |
| Expression Parameters | 8-bit |

> 说明：更细的版本迁移链应视为历史说明，不应当作长期固定主结论。

## 3. 关键能力
- Zoom / Aperture / Exposure
- Point AF / Face AF / Selfie AF / Manual Focus
- Auto Leveling / Stabilizer / Pin to World / Reposition
- Pin[1-4] 保存机位
- Drone Mode
- Quick Selfie
- Mesh Hiding / Mask System / Grid / Focus Peaking / Preview HUD
- 4K Capture（需结合 Stream Camera 等整体链路）
- Remote / Remote Only 模式

## 4. 工作流要点
### 4.1 Non-Destructive 工作流
- 核心优势是不直接把相机逻辑永久写死进 Avatar 源资源。
- 更适合共享 Avatar、持续迭代 Avatar、或与其他非破坏性工具（如 MA / VRCFury）共存。

### 4.2 相机模型参考对象
- `VirtualLensOrigin`：相机组件参考点
- `VirtualLensPreview`：预览屏幕参考点
- `NonPreviewRoot`：非预览根（自定义网格替换等）

### 4.3 兼容性提醒
- 若第三方预制件含 `VirtualLensOrigin` 与 `VirtualLensPreview`，通常说明它为 VirtualLens2 兼容预制件。
- 没有可见多边形沿 `VirtualLensOrigin` 的 +Z 视线超出约 0.05m，可降低反射 / 网格异常风险。

## 5. 版本迁移说明（保守）
- `v2.10.0` 起：Non-Destructive 更明确成为标准流程
- `v2.11.0` 起：明确要求 Unity 2022

> 这些条目更适合视为**历史迁移说明**，不是长期唯一判断依据；日后若主版本继续变化，应以总览页中的稳定门槛为准。

## 6. Memory 准入原则
### 可长期保留
- Non-Destructive 设计哲学
- 稳定依赖门槛
- 关键能力边界
- 参考对象与兼容检测思路

### 不应过度沉淀
- 逐步截图式操作教程
- 强时点版本号列表
- 发布态 / 销售态细节

---

## 相关页面

[[camera-tools.md]] · [[modular-avatar.md]] · [[vrcfury-reference.md]] · [[../world/vrc-camera-settings.md]]
