---
title: "Gesture Manager — VRChat 化身动画预览与调试工具"
category: avatar
knowledge_level: applied
status: active
source: "github.com/BlackStartx/VRC-Gesture-Manager + 官方文档"
source_type: community
version: 1.0
last_review: 2026-08-19
confidence: High
tags:
  - avatar
  - gesture
  - preview
  - debug
  - expression-menu
  - animator
  - osc
aliases:
  - "Gesture Manager"
  - "VRC-Gesture-Manager"
  - "GM"
related:
  - faceemo.md
  - expression-menu.md
  - playable-layers.md
  - avatar-audit-methodology.md
  - unity-mcp.md
  - blend-shape-troubleshooting.md
type: entity
created: 2026-08-19
sources: "github.com/BlackStartx/VRC-Gesture-Manager | vrchat.blackstartx.gesture-manager"
updated: 2026-08-19
---
# Gesture Manager — VRChat 化身动画预览与调试工具

> 来源: BlackStartx 出品的 Gesture Manager 官方文档与研究简报（经官方 web_fetch 核实）
> 置信度: High（官方文档）；证据边界须严格限定（见第 3 章）
> 官方站: https://github.com/BlackStartx/VRC-Gesture-Manager
> VPM 包: `vrchat.blackstartx.gesture-manager`（注意拼写，见 6.2）
> 检索日期: 2026-08-19

---

## 1. 工具定位

**Gesture Manager** 官方一句定位：「*preview and edit your avatar animation directly in Unity*」——一个 **Unity 编辑器内 Play Mode** 的手势/表情**预览与调试工具**（simulator / emulator）。[FACT]

| 维度 | 说明 |
|------|------|
| **设计目标** | 在 Unity`Play` 模式里模拟客户端手势、表情菜单、Contact、Animator 层、OSC 等 |
| **平台支持** | SDK2 / SDK3（Unity 2018/2019 起；现行 2022.3+） |
| **核心岗位** | **Unity 编辑器预览/诊断**——**不是创作 provider，不是正式运行时证明** |
| **非目标** | 不能证明 SDK 构建/上传、桌面/VR/多人真实行为、网络同步、性能上传校验 |

> ⚠️ **定位红线**：Gesture Manager 是 **Unity 编辑器 Play Mode 预览**，其任何"客户端行为"都是由工具在编辑器内重建/模拟的，**绝不可提升为 VRChat desktop/VR/multiplayer/SDK build/upload 的证明**。证据标注必须写 `CLIENT_RUNTIME (Unity/Gesture Manager preview)`。见 [[avatar-audit-methodology.md]] 的证据分层。

---

## 2. 核心组件与机制

### 2.1 手势模拟按钮 [FACT]

- 左右手各手势按钮；**SDK3.0** 用 **RadialMenu** 测试 Expression Menu。

### 2.2 Options（选项）[FACT]

| 分组 | 项目 |
|------|------|
| **Locomotion** | Walking / Running / Crouch / Prone / Falling |
| **States** | AFK / Seated / IK / T Pose |
| **Tracking** | 追踪点数 / VRMode |
| **Extra** | Gesture Weights / MuteSelf / IsLocal / InStation（**灰字 = 化身未使用该参数**） |

### 2.3 Edit Mode（编辑模式）[FACT]

- **克隆化身** + 全 VRChat 控制器层动画；可在 **Animation 标签**直接编辑。

### 2.4 Debug 标签 [FACT]

- Animator Parameters / Tracking Controls / Layer Weights / Animator States / Locomotion·Pose Space。

### 2.5 Animator Debugging [FACT]

- 选化身 → 双击 **Playable Layer** → 打开 Unity Animator 预览。

### 2.6 OSC 集成 [FACT]

- 监听/发送 Message / Bundle；从 **LocalLow** 加载设置；**端口必须空闲**，否则冲突。

### 2.7 Clickable Contacts & 新特性 [FACT]

- **Clickable Contacts**：鼠标点击触发 Contact，支持 **tag 过滤**；3.9.9 支持 **Box 形状**。
- **3.9.9 新增**：Raycast / Box-Shaped Contacts / **Local-Only**（Parameter Drivers、Contact Sender/Receiver **远端不生效**）。
- **3.9.8 新增**：Avatar Culling Simulation（`IsAnimatorEnabled`）、`_PreviewMode_` 参数、Scene Camera 匹配。

---

## 3. 条件与证据边界（★关键）

Gesture Manager 在 **Unity Play Mode 内重建/模拟**以下行为 [FACT]：

- 客户端表情菜单 / 手势 / 径向菜单参数 / Contact / Animator 层 / OSC。

**证据边界（必须遵守）**：

| 由工具设置 | 非真实客户端/服务器 |
|------------|------------------------|
| IsLocal / InStation / 手势权重 | 由工具设定，非真实客户端/服务器判定 |

> 🚫 **证据提升禁令**：`IsLocal / InStation / Gesture Weights` 等由工具设定，**不反映真实 VRChat 客户端/服务器状态**。任何 `CLIENT_RUNTIME` 证据必须带限定词 **`Unity/Gesture Manager preview`**，**绝不**写成 `VRChat desktop` / `VR` / `multiplayer` / `SDK build` / `upload`。此约定与 [[avatar-audit-methodology.md]] 的证据分层呼应。[FACT + INFERENCE]

**不能证明**（超出工具能力）：SDK 构建与上传、桌面/VR/多人真实行为、网络同步、性能上传校验。[FACT + INFERENCE]

---

## 4. 与 FaceEmo・MA・其他工具关系

### 4.1 与 FaceEmo 联动 [FACT]

- FaceEmo 官方教程也把 Gesture Manager **当作 preview 工具**使用。
- **联动注意**：Play 模式时**别选 Gesture Manager 对象**；进 Play 后再选；**Game 视图不显示**看 Scene 视图（见 [[faceemo.md]] 第 5.2 节）。

### 4.2 与 MA・NDMF 关系 [INFERENCE]

- Gesture Manager 只是**预览/调试**层，**不参与 NDMF 构建合并**、不产出 Avatar 资产、不改写生成控制器。
- 与 [[modular-avatar.md]] / [[ndmf-tools.md]] 是「预览端 ↔ 构建端」的关系：预览验证可走 GM，最终进入化身仍需 MA/NDMF 管线。此关系为推断，官方未系统阐述【INFERENCE】。

---

## 5. 常见注意

| 注意点 | 说明 |
|--------|------|
| **OSC 端口冲突** | OSC 集成要求端口空闲；冲突会失败 |
| **Unity / VRCSDK 硬性下限** | 有明确版本下限，低于则工具不可用（见 6.1） |
| **GM 预览注意** | Play 按钮时别选 GM 对象、进 Play 后再选、Game 视图不显示看 Scene（见 4.1） |
| **Edit Mode 克隆** | 编辑的是克隆化身，注意勿把临时修改当成源 Avatar 变更 [INFERENCE] |
| **灰字参数** | Extra 中灰字 = 化身未使用该参数，属正常提示 |

### 5.1 与排障方法论对接 [INFERENCE]

- 用 GM 排查「手势/菜单不触发」时，先在 **Edit Mode / Debug 标签**核对参数与层，再回源（如 FaceEmo 的 Pattern/Expression）修改；**不要 patch 生成控制器**。
- 涉及 BlendShape / Blink / LipSync 冲突时，参考 [[blend-shape-troubleshooting.md]] 与 [[faceemo.md]] 第 5.3 节失败类。

---

## 6. 安装与版本

### 6.1 版本基线 [FACT]

| 版本流 | 关键数据 |
|--------|----------|
| **3.9.8 稳定** | 2026-02-02；Unity 2022 + VRCSDK 3.5.2+ |
| **3.9.9 pre-release** | 2026-05-09；Unity 2022 + VRCSDK 3.10.4+；VPM Catalog **现列 3.9.9**（官方 source.json 精确 tag 为 **3.9.9-pr6**，2026-08-19 核实） |

**依赖**：`com.vrchat.avatars >= 3.10.4 < 3.11.X`。[FACT]

### 6.2 安装 [FACT]

- **VCC Curated 社区包**（推荐），或 `.unitypackage` / `.zip` 导入。
- **VPM 包名拼写**：`vrchat.blackstartx.gesture-manager`（**不是** `blackstartx.gesture-manager`）。
- 旧版 legacy 目录：`Assets\GestureManager`。
- **SDK2.0 用户**：可用 3.x（仍兼容）或 2.0 专用 release。

> ⚠️ 拼错 VPM 包名或安装低版本会触发 Unity/VRCSDK 下限不满足问题。

---

## 7. 来源与证据

| 事实层级 | 定义 | 本文用法 |
|----------|------|----------|
| **[FACT]** | 官方可核实（URL + 检索日期 2026-08-19） | 第 2/3/6 章主体 |
| **[INFERENCE]** | 推断（必须标） | 第 4.2 章节、第 5.1 节方法对接 |
| **[UNKNOWN]** | 未确认 | 无（本文证据边界均已明确标注） |

- GitHub: https://github.com/BlackStartx/VRC-Gesture-Manager [FACT]
- VPM 包: `vrchat.blackstartx.gesture-manager` [FACT]
- VCC Curated 社区包列表 [FACT]
- 检索日期: 2026-08-19 [FACT]

---

## 相关文档

[[faceemo.md]] · [[expression-menu.md]] · [[playable-layers.md]] · [[avatar-audit-methodology.md]] · [[unity-mcp.md]] · [[blend-shape-troubleshooting.md]]
