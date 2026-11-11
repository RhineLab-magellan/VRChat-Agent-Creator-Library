---
title: "VRCLens — 子工具文档"
category: avatar
knowledge_level: applied
status: active
source: docs.vrcd.org.cn + Gumroad 页面 + 分类审查归纳
source_type: community
version: 1.0
last_review: 2026-07-13
confidence: Medium
tags:
  - avatar
  - camera
  - vrclens
  - invasive
aliases:
  - VRCLens
  - VRC镜头
  - Avatar摄影镜头工具
  - VRCLens Camera Tool
related:
  - camera-tools.md
  - vrcfury-reference.md
  - ../world/vrc-camera-settings.md
type: entity
created: 2026-07-13
sources: docs.vrcd.org.cn + Gumroad 页面 + 分类审查归纳
updated: 2026-07-13
---

# VRCLens — 子工具文档

> 本页承载 VRCLens 的安装、Apply 逻辑、工作方式与兼容性细节；总览对比与选型建议见 `camera-tools.md`。

## 1. 定位
- **平台**：Desktop + VR
- **设计哲学**：**Invasive**
- **实现路径**：会直接修改 Avatar 现有动画 / 菜单 / 参数等相关资源，或依赖自动化工具代为接入。

## 2. 稳定依赖门槛
| 项目 | 要求 |
|---|---|
| Unity | 2022 LTS |
| VRCSDK | 3.0.8+（细节随版本变化） |
| Expression Parameters | 8-bit |
| 关键依赖 | AV3Manager / Filament PBR / DoF Shader 等生态组件 |

## 3. 关键能力
- 曝光 / 缩放 / 光圈 / 传感器尺寸
- Picture Style
- **HDR + White Balance**
- Image Stabilization + auto-straighten
- Built-in drone movement + tracking
- Selfie Autofocus / Avatar-detect autofocus
- Stereographic 3D camera mode
- Quick Selfie front camera mode
- World-fixed / Avatar-fixed camera mode
- Viewfinder overlay

## 4. 工作流要点
### 4.1 Apply / 接入逻辑
- VRCLens 的实际使用重点不只是“导入 Prefab”，而是**如何接入 Avatar 现有 FX / Menu / Parameters 体系**。
- 因为它偏 Invasive，重新 Apply、重复接入、或与其他工具混搭时，更容易出现资源被覆盖或结构漂移。

### 4.2 原相机开启要求
- 文章层经验显示：理解“原相机仍需开启 / VRCLens 替换原相机画面”的工作方式很关键。
- 这属于**使用前提认知**，比一步步操作截图更适合长期保留在 knowledge 条目中。

### 4.3 与非破坏性工具的关系
- VRCLens 自身偏 Invasive，但其生态中存在用 VRCFury 等方式对其进行更安全封装的实践。
- 因此：
  - **裸用 VRCLens**：更像资源修改型工作流
  - **通过生态封装使用**：可部分降低侵入性与回滚成本

## 5. 已知兼容性问题
### APS 兼容性
- 与 Avatar Pose System (APS) 同用时，曾有“灵魂出窍后相机仍固定在原位置”的社区问题。
- 思路：把相关对象纳入 `unfixed objects` 一类白名单，而不是把这类问题误判为相机本身彻底失效。

### 一般后处理冲突
- 可能出现景深冲突、AO 无效、运动模糊无效等现象。
- 这些问题更应作为“已知兼容性提醒”长期保留，而不是逐条做教程式排障。

## 6. Memory 准入原则
### 可长期保留
- Invasive 设计哲学
- 稳定依赖门槛
- HDR / White Balance / 3D 模式等独特能力边界
- 原相机开启要求与 Apply 工作方式
- 已知兼容性问题的高层提醒

### 不应过度沉淀
- 逐步截图式安装流程
- 强时点版本号与发布信息
- 销售页 / 价格页细节

---

## 相关页面

[[camera-tools.md]] · [[vrcfury-reference.md]] · [[../world/vrc-camera-settings.md]]
