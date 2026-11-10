---
title: "VRChat Avatar 中英术语对照表"
category: avatar

knowledge_level: applied
status: active

tags:
  - avatar
  - glossary
  - translation
  - terminology
  - 中文社区
  - vrcd-21-supplement

aliases:
  - 中英术语对照表
  - "VRChat Avatar Glossary"
  - "Avatar 术语"
  - 欠标准译名
  - 非标准译名

related:
  - animator-system.md
  - expression-menu.md
  - playable-layers.md
  - avatar-dynamic-bone-limits.md
  - modular-avatar.md
  - teaching-methodology.md

source: "VRCD 文档库 — 关键词对照表（本地 raw 参考资料附录）"
source_type: community
version: 1.0
last_review: 2026-07-04
confidence: High
type: entity
created: 2026-07-04
sources: "VRCD 文档库 — 关键词对照表（本地 raw 参考资料附录）"
updated: 2026-07-04
---

# VRChat Avatar 中英术语对照表

> **来源**: VRCD 文档库「Avatar 与你」附录: 关键词对照表(36 条)
> **URL**: https://docs.vrcd.org.cn/books/avatar
> **本地化日期**: 2026-07-04
> **重要程度**: 🟡 教学参考价值,主要服务中文社区翻译规范

---

## 1. 文档定位

本表为中文社区**翻译规范参考**,目的:
1. **统一术语**: 减少中文社区内的术语混乱
2. **警示非标准译名**: 标注常见但**不被官方推荐**的译名
3. **辅助教学**: 帮助新创作者建立中英术语映射

> ⚠️ **重要提示**: 本表以 VRChat 官方英文原名为准,**官方文档和 SDK 中出现的术语一律以英文为准**。中文译名仅供中文社区沟通使用。

---

## 2. 完整对照表(36 条)

| 英文 | 常见中文代称或解释 | ⚠️ 欠标准译名(社区警示) |
|------|------------------|------------------------|
| 自动脚本,自动插件 | 能够自动将符合某种规格的预制件自动安装到 Avatar 上的 Unity 插件(例:MA、VRCFury) | — |
| **Avatar** | 模型,虚拟形象 | ⚠️ 头像,皮肤,外观 |
| **VRChat Creator Companion** | VCC,工程管理器 | — |
| **Armature** | 骨架 | — |
| **Mesh** | 网格,模型 | ⚠️ 模型(易混淆) |
| **Texture** | 贴图,纹理 | ⚠️ 材质(易与 Material 混淆) |
| **Material** | 材质球,材料球 | — |
| **Shader** | 着色器 | ⚠️ 光影(不准确) |
| **Animator** | 动画器,状态机 | ⚠️ 动画师(职业混淆) |
| **Animation** | 动画 | — |
| **(Unity) Editor** | Unity,Unity 编辑器,编辑器 | — |
| **(Unity) Project** | 工程,工程文件 | — |
| **World** | 世界 | ⚠️ 地图(不准确) |
| **Prefab** | 预制体,预制件 | — |
| **Hierarchy** | 场景列表,列表 | — |
| **Hips** | 髋部,腰部,胯部,臀部,大腿骨和骨盆相连的骨骼部位 | — |
| **Upload Ready** | 完整,完备;即刻可以上传的 | — |
| **VRChat** | VRC | — |
| **Object** | 物体,组件 | — |
| **Component** | 组件,脚本;在 Inspector 中添加给物体的组件 | — |
| **Asset Window** | 资产窗口,资源窗口;显示了你 Project 中所有资源的窗口 | — |
| **Inspector** | 检查器 | ⚠️ 信息窗口(不准确) |
| **Skinned Mesh** | 蒙皮,蒙皮网格;被骨头控制的 Mesh | — |
| **liltoon** | 一款常用的 Shader,以功能完备著称;常用于 Booth 上的商品和 Upload Ready 的 Avatar | — |
| **poiyomi toon** | 一款很多人使用的 shader;以**难以维护**和极其强大和多样的功能著称 | — |
| **Renderer** | 渲染器;一个用来渲染某个 Mesh 的组件,有 Mesh 和 Skinned Mesh 两种 | — |
| **VRC Avatar Descriptor** | Avatar 的核心组件 | — |
| **Playable Layers** | VRC Avatar Descriptor 的一部分,Avatar 不同用途的 5 个 Animator | — |
| **Physbone Collider** | 动骨碰撞,碰撞;一个让它所在的物体可与 Physbone 进行碰撞的 Component | — |
| **Physbone** | 动骨,新动骨;VRChat Avatar Dynamics 更新中取代 Dynamic Bone 的系统 | — |
| **Parameter** | 参数 | — |
| **Expression Parameters** | 参数表,同步参数表,同步表,参数 | — |
| **Expression Menu** | 菜单,轮盘菜单 | — |
| **Remote Player** | 远端玩家,其他人,其他玩家;自己之外的任何人 | — |
| **Local Player** | 我,自己;低头看到的那副身体 | — |
| **State** | 状态;Animator 里的一个方块,代表一个状态,里面可以包含一个 Animation | ⚠️ 动画(易混淆) |
| **Transition** | 过渡,动画过渡 | — |
| **Condition** | 条件 | ⚠️ 过渡限制(不准确) |
| **Weight** | 权重 | ⚠️ 重量,影响(不准确) |

---

## 3. 关键术语辨析

### 3.1 ⚠️ Texture vs Material 混淆

- **Texture** = 贴图/纹理(图片)
- **Material** = 材质球(包含 Texture + Shader + 渲染属性)
- 常见错误: 把 Texture 翻译为"材质" → 应译为"贴图/纹理"
- 教学话术: "Material 是盒子,Texture 是装在盒子里的画"

### 3.2 ⚠️ State vs Animation 混淆

- **State** = 状态(Animator 中的方块)
- **Animation** = 动画(状态内播放的内容)
- 常见错误: 把 State 翻译为"动画" → 应译为"状态"
- 教学话术: "State 是播放器的按钮,Animation 是按钮播放的歌曲"

### 3.3 ⚠️ Avatar ≠ 头像

- **Avatar** = 虚拟形象(VRChat 中的 3D 角色)
- "头像" 通常指 2D 图像(如社交媒体头像)
- 官方文档中 Avatar 专指 3D 虚拟形象
- 教学话术: "Avatar 是 3D 角色,不等于 2D 头像"

### 3.4 ⚠️ Animator ≠ 动画师

- **Animator** = 动画器/状态机(Unity 组件)
- "动画师" 是职业称谓
- 常见错误: 把 Animator 翻译为"动画师" → 应译为"动画器/状态机"
- 教学话术: "Animator 是 Unity 里的组件,不是做动画的人"

---

## 4. 翻译规范建议

### 4.1 推荐做法

- ✅ 首次出现时同时标注英文和中文:"Animator(动画器)"
- ✅ 后续可仅用中文,但保留英文别名
- ✅ 跨文档引用时使用统一译名
- ✅ 复杂术语优先使用英文原文,避免翻译歧义

### 4.2 避免做法

- ❌ 避免使用"欠标准译名"(在表 §2 中标 ⚠️ 的)
- ❌ 避免自创译名(除非已在社区广泛流通)
- ❌ 避免在 SDK、API、代码相关场景使用中文译名

### 4.3 教学场景中的术语使用

- **入门教学**: 多用中文译名(降低理解门槛)
- **进阶教程**: 中英双语(建立工程对应)
- **专业文档**: 优先英文(保持准确性)

---

## 5. 引用

- [L1] https://docs.vrcd.org.cn/books/avatar - VRCD 21「Avatar 与你」附录: 关键词对照表
- [L2] https://docs.vrcd.org.cn/books/vrc - VRCD 入门包 Unity 术语表
- [L3] https://creators.vrchat.com/avatars/ - VRChat 官方(英文原名)

---

## 相关页面

[[animator-system.md]] · [[expression-menu.md]] · [[playable-layers.md]] · [[avatar-dynamic-bone-limits.md]] · [[modular-avatar.md]] · [[teaching-methodology.md]]
