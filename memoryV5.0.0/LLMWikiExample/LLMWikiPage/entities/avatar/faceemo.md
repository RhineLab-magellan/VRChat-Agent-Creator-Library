---
title: "FaceEmo — VRChat 面部表情创建与配置工具"
category: avatar
knowledge_level: applied
status: active
source: "github.com/suzuryg/face-emo + 官方文档 suzuryg.github.io/face-emo"
source_type: community
version: 1.0
last_review: 2026-08-19
confidence: High
tags:
  - avatar
  - facial-expression
  - gesture
  - expression-menu
  - modular-avatar
  - non-destructive
aliases:
  - "FaceEmo"
  - "face-emo"
  - "Face emote configuration tool for VRChat Avatars 3.0"
related:
  - gesture-manager.md
  - expression-menu.md
  - playable-layers.md
  - blend-shape-troubleshooting.md
  - modular-avatar.md
  - ndmf-tools.md
  - avatar-audit-methodology.md
type: entity
created: 2026-08-19
sources: "github.com/suzuryg/face-emo | suzuryg.github.io/face-emo"
updated: 2026-08-19
---
# FaceEmo — VRChat 面部表情创建与配置工具

> 来源: suzuryg（sai）出品的 FaceEmo 官方文档与研究简报（经官方 web_fetch 核实）
> 置信度: High（官方站点）；个别项标 [UNKNOWN]/[INFERENCE]
> 官方站: https://suzuryg.github.io/face-emo/（含 /ja、/zh）
> GitHub: https://github.com/suzuryg/face-emo | VPM 包: `jp.suzuryg.face-emo`
> 检索日期: 2026-08-19

---

## 1. 工具定位

**FaceEmo** 官方一句定位：「*Face emote configuration tool for VRChat Avatars 3.0*」，即面向 **VRChat Avatars 3.0** 的**面部表情创建与配置工具**。它通过「手势（Gesture）× 表情（Expression Menu）」组合来切换表情，并提供眨眼 / 口型同步 / 眼动 / 表情模式等面部行为。[FACT]

| 维度 | 说明 |
|------|------|
| **设计目标** | 用 Gesture Table + Expression Menu 快速批量创建/配置化身面部表情 |
| **核心能力** | 表情切换、Blink（眨眼）、LipSync（口型同步）、EyeTracking（眼动）、Mouth Morph Canceler |
| **生成范式** | 编辑器内的**构建期生成式（非破坏性）**工具——产出标准 FX 层/Expression Menu/参数 |
| **运行时期权** | 上传/构建后的化身**不残留任何 FaceEmo 组件**，运行期是标准 VRChat Avatars 3.0 资产 |
| **非目标** | 不是运行时组件提供商，不注入 FaceEmo 运行时代码 |

> 💡 **关键认知**：FaceEmo 是「源（source）→ 生成物」的构建期流程。你在编辑器里配置的 FaceEmo 场景对象/Pattern/Expression 是**源**；Apply to Avatar 之后生成的面部控制器、菜单、参数是**生成物**。**不得手工修改生成控制器**——再次生成会被抹掉。详见第 3 章所有权链。

### 1.1 官方站点勘误

权威官方站为 https://suzuryg.github.io/face-emo/（含 `/ja` 日文、`/zh` 中文分支）。曾见 `face-vtuber-book`、`face-emu` 等名与之关联，**该关系未被证实**【UNKNOWN】【未确认】【coverage: low】。引用归属时一律以官方 `.github.io/face-emo` 站点为准，不把未证实名称写入特性。

---

## 2. 核心组件与机制

FaceEmo 的核心组件（官方 Reference 篇）层次如下：[FACT]

### 2.1 Facial Expression Menu（表情菜单对象 = 「源」）

- 场景对象，由 **FaceEmo → New Menu** 创建；是整套配置的**入口与持有者（源）**。
- **可复制**：复制出的新菜单对象**不影响原版**——两者各自独立。
- ⚠️ **例外**：引用的「动画剪辑（Animation Clip）」是**共享**的；复制菜单复用同一 Clip 时编辑共享内容会影响所有引用者。

### 2.2 Expression Pattern（菜单「模式」）

- 一个 Pattern = 一组「**手势 × 表情**」组合的开关方案。
- **Change Default Face / Use ClipName as PatternName**：设定默认表情 / 以 Clip 名为 Pattern 名。
- 可配置 **Blink / LipSync / EyeTracking / Mouth Morph Canceler** 子系统行为。

### 2.3 Expression（单条「条件 → 表情动画」）

- 单条定义为「**条件 → 表情动画**」，是 Pattern 下最小触发单元。

### 2.4 Expression Editor（BlendShape / 参数编辑器）

- 用于编辑 **BlendShape** 与参数，可把结果**保存到用户自定义路径**（用户拥有的稳定 Clip），实现「自建稳定来源」。

### 2.5 Gesture Table（手势 × 表情组合与优先级）

- 定义「手势 × 表情」组合的映射与**优先级**，决定手部手势触发表情。

### 2.6 工具栏 / 组件

| 元素 | 作用 |
|------|------|
| 工具栏 **Apply to Avatar** | 触发生成（构建期），告知生成完成 |
| 工具栏 **New Menu** | 新建表情菜单对象 |
| **Launcher 组件** | 管理生成目标/运行配置（face-emo 的入口组件） |
| **AV3Setting** | VRChat Avatars 3.0 平台/参数相关设置 |

---

## 3. 条件与证据边界

### 3.1 条件系统（★重要更正，以官方 1.7.0 为准）

**1.7.0 实际条件**由两层构成：[FACT]

**手部类型 × 手势类型 × 等于/不等于**：

- 手部类型：`LeftHand` / `RightHand` / `BothHands` / `OneHand`（任一非双手）/ `AnyHand(s)`（任一含双手）
- 手势类型：`Fist` / `Open` / `Point` / `Victory` / `Rock` / `Gun` 等
- 比较：等于 / 不等于

**附加条件**：

- **Trigger 挤压**（Use Left/Right Trigger）：仅在对应手为 `Fist` 时有效
- **菜单触发**：切换 Pattern（Emote Select / Emote Lock ON 属于**菜单/选项层**）

> ⚠️ **反模式警示**：`Viseme` / 音频 / 视频 / 情绪 / Contact / AFK **在 FaceEmo 官方文档中不存在**。【UNKNOWN】通用模板中的"情绪/音频/视频触发话术"**不得当作 FaceEmo 特性写入**——FaceEmo 是手势 + 菜单驱动，不是情绪/音频驱动。

### 3.2 每个表情的行为开关

每个表情（Expression）可由以下开关注入/关闭动画：[FACT]

| 开关 | 含义 |
|------|------|
| **Enable Eye Tracking** | 启用眼动 |
| **Enable Blink** | 启用眨眼 |
| **Enable Lip Sync** | 启用口型同步 |
| **Mouth Morph Canceler** | 说话时把 Mouth Morph 相关 BlendShape 退回默认（防止说话/表情冲突） |

> 💡 闭眼表情排障常与 Blink 相关：详见第 5 章 BlendShape 警告。

---

## 4. 所有权（Ownership）与证据边界

沿用 playbook 的「源 / 生成物」责任链，FaceEmo 的归属如下：[INFERENCE]（基于官方文档 + playbook 归纳）

| 角色 | 对象 |
|------|------|
| **源（可编辑入口）** | 表情菜单对象 / Expression Pattern / Expression 定义 / 用户自建稳定 Clip（Expression Editor 保存到用户路径）/ 场景配置 / Launcher 目标 |
| **生成物（不可手改）** | `FaceEmoFX`（生成的 FX 层）、生成的 Expression Menu + 参数、备份 / 预览 / 缩略图、**`FaceEmoPrefab`**、示例资产 |

> ⚠️ **删 FaceEmoPrefab = 移除表情菜单**——它是生成物，但删除它会连带移除整个表情菜单接线。
> ⚠️ **不要手工改写生成控制器**：再次 Apply/重建会抹掉手改内容。任何变更都应回到「源」层做。

### 4.1 排障归属纪律

- 手势/菜单不触发 → 先查「源」的条件/Clip，**不要直接 patch 生成控制器**（regeneration 会销毁该改动）。[INFERENCE]
- 生成资产缺失/过期/重复 → 标记旧生成路径 `STALE`，确认所有消费者后再处置；不要臆测删除。[INFERENCE]

---

## 5. 常见故障

### 5.1 手势不变表情五查（官方 FAQ）[FACT]

按顺序检查：

1. **未加入菜单**（该表达式没被加进 Expression Menu）
2. **Pattern 未选中**（当前菜单栏未选中对应 Pattern）
3. **Emote Select 打开**（处于手动选择 Emote 模式）
4. **Emote Lock ON**（表情被锁定）
5. **Settings → Emote Lock**（全局表情锁定开启）

### 5.2 其他官方故障点 [FACT]

| 现象 | 处置 |
|------|------|
| **原有表情被禁用** | FaceEmo 接管后需**重建进 FaceEmo**；原表情不会自动保留 |
| **移除 FaceEmo** | 删除 **`FaceEmoPrefab`**（连带移除表情菜单） |
| **UI 错乱** | 进入 **Play 模式一次** 恢复 |
| **「Don't show again」对话框误关** | 到 **Editor Settings** 复位 |
| **BlendShape 警告** | 含 blink/lip-sync 的需处理：闭眼表情把 **Enable Blink** 关闭 |
| **GM 预览注意** | Play 模式时**别选 GestureManager 对象**（与 Gesture Manager 联动，见 [[gesture-manager.md]]） |

### 5.3 playbook 方法论（非官方故障表，归因纪律）[INFERENCE]

- **手势/菜单不触发排查顺序**：入口/Pattern 与参数值 → 手势权重/过渡条件与左右手语义 → 生成参数类型/默认值/FX 最终消费者 → 状态过渡/打断/层权重/掩码/Write Defaults/更高优先级状态 → 绑定路径对照构建后层级 → 检查是否被其他表情所有者或自定义 FX 层写了同一 BlendShape[INFERENCE]
- **Blink / LipSync / 眼动六失败类**：表情 Clip 显式写眼/嘴曲线；取消/保护设置缺失或过宽；过渡或 Write Defaults 状态重置曲线；其他 provider 或自定义 FX 层后有所有权；复制的全脸 Clip 含意外零值曲线；构建后绑定路径失效（层级重映射）[INFERENCE]——逐一验证，**一个通过不代表另两个通过**。
- **跨化身污染**：多化身共享同一源 prefab 时，明确共享/隔离决策；优先化身专属源 [INFERENCE]。

---

## 6. 安装与版本

### 6.1 版本基线 [FACT]

| 项目 | 值 |
|------|-----|
| **当前版本** | **1.7.0**（releases 2026-03-01；VPM Catalog 列 31 个版本） |
| **Unity** | **2022.3+** |
| **VPM 依赖** | **无**（pure 包） |

### 6.2 安装方式 [FACT]

- **VCC / ALCOM**：添加仓库 `https://suzuryg.github.io/vpm-repos/vpm.json` 后安装
- **BOOTH**：BOOTH 发行版**内置 Modular Avatar**（见 6.3 依赖）

### 6.3 依赖表（About Bundled Software）[FACT]

| 打包依赖 | 说明 |
|----------|------|
| Animator As Code | 生成器使用 |
| ComboGestureExpressions Integrator | 集成 |
| Custom Animator Controllers（Pocho） | 集成 |
| **Modular Avatar** | **仅 BOOTH 发行版内置**（VPM/GitHub 版不打包） |
| VPMPackageAutoInstaller | 仅 BOOTH |
| UniRx | 生成器使用 |
| Extenject | 生成器使用 |

> **NDMF 未列入直接依赖**。FaceEmo 产出物是**经 MA/NDMF 管线在构建期合并**进入最终化身——「NDMF 生成式」成立角度是「产物经管线合并」，而非 FaceEmo 自带 NDMF Pass【未直接证实】【UNKNOWN】【coverage: low】。见 [[modular-avatar.md]] 与 [[ndmf-tools.md]]。

### 6.4 1.7.0 变更亮点 [FACT]

- 降低参数开销
- UI 菜单项可选
- zh-CN 翻译
- FX 关闭开关检测
- 检测到眨眼动画时关闭分支眨眼

> ⚠️ **post-1.7.0 PR#259**（float 参数由表情动画控制并写入 MA Parameters）= **未发布路线图，勿当事实**【UNKNOWN】。

---

## 7. 来源与证据

| 事实层级 | 定义 | 本文用法 |
|----------|------|----------|
| **[FACT]** | 官方可核实（URL + 检索日期 2026-08-19） | 第 2/3/5/6 章主体 |
| **[INFERENCE]** | 推断（必须标） | 第 4 章所有权归纳、第 5.3 排查顺序 |
| **[UNKNOWN]** | 未确认 | `face-emu` 关系、NDMF 直接依赖、PR#259 |

- 官方站: https://suzuryg.github.io/face-emo/（/ja、/zh）[FACT]
- GitHub: https://github.com/suzuryg/face-emo [FACT]
- VPM 仓库: https://suzuryg.github.io/vpm-repos/vpm.json [FACT]
- 检索日期: 2026-08-19 [FACT]

---

## 相关文档

[[gesture-manager.md]] · [[expression-menu.md]] · [[playable-layers.md]] · [[blend-shape-troubleshooting.md]] · [[modular-avatar.md]] · [[ndmf-tools.md]]
