---
title: "Avatar 3.0 Toggle 创建工具（非 MA 路径）"
category: avatar

knowledge_level: applied
status: active

tags:
  - avatar
  - toggle
  - expression-menu
  - animator
  - tools
  - third-party
  - non-ma-path

aliases:
  - Avatar 3.0 Toggle Tools
  - VRChat Toggle Assistant
  - Auto-Toggle-Creator
  - VRCToggleToolkit
  - AV3ToggleUtil
  - Toggle Creator
  - 非 MA Toggle 工具
  - GoGo Loco 兼容性

related:
  - expression-menu.md
  - playable-layers.md
  - avatar-30-emulator.md
  - modular-avatar.md
  - avatar-modding-guide.md
  - vrcfury-reference.md

source: 多个第三方工具综合
source_type: community
version: 1.1
last_review: 2026-07-04
confidence: High
type: entity
created: 2026-07-04
sources: 多个第三方工具综合
updated: 2026-07-04
---

# Avatar 3.0 Toggle 创建工具（非 MA 路径）

> **本地化日期**: 2026-07-04
> **状态**: ✅ 综合多个第三方工具（K01 + K12）
> **定位**: **与 Modular Avatar 平行的工具路径**——适合不用 MA 的创作者

---

## 概述

> Avatar 3.0 Toggle 创建有**两条主要路径**：
> 1. **MA 路径**（NDMF 编译时）：使用 Modular Avatar 的 Object Toggle / Merge Animator 等组件
> 2. **第三方工具路径**（编辑器时）：使用 VRChat Toggle Assistant / Auto-Toggle-Creator 等
>
> 本文档描述**第二条路径**的工具生态。

---

## 1. 工具分类

### 1.1 一键生成类（无需手动 Animator）

| 工具 | 作者 | 许可 | 关键特点 | 维护状态 |
|------|------|------|---------|---------|
| **VRChat Toggle Assistant** | Shatteredfur | 个人非商业 | Simple / Advanced / Radial Puppet / Submenu | 活跃 |
| **Auto-Toggle-Creator** | CascadianVR | MIT | 轻量替代，选项较少 | 活跃 |
| **VRCToggleToolkit** (fork) | SuperFlue | 派生 | VPM + WD Off 默认 + Initialization state | 活跃 |
| **AV3ToggleUtil** | d4rkc0d3r | MIT | 右键直接创建 | 维护中 |

### 1.2 测试/调试类

| 工具 | 作者 | 用途 | 详见 |
|------|------|------|------|
| **Avatar 3.0 Emulator** | lyuma | Unity 内模拟 Avatar3 运行时 | `avatar-30-emulator.md` |
| **GestureManager** | 社区 | 提供美观的径向菜单（与 Av3Emulator 配合） | 工具主页 |

### 1.3 图标/资源类

| 资源 | 类型 | 来源 |
|------|------|------|
| Avatars 3.0 资产自定义图标 | 资产图标插件 | https://booth.pm/en/items/2567604 |
| VRChat AV3 Icons + Template | 按钮图标包 | https://booth.pm/en/items/2577519 |

---

## 2. VRChat Toggle Assistant（详细）

> [FACT] **来源**: https://shatteredfur.gumroad.com/l/vrctoggle
> **多镜像**: https://vrmodels.store/other/scripts/50366-vrchat-toggle-assistant-v0120.html（v0.12.0）

### 2.1 关键功能

1. **Simple Toggle**：为单个 accessory 生成开关
2. **Advanced Toggle**：同时控制**多个对象**
3. **Radial Puppets with Blendtrees**：用 BlendTree 创建 Radial Puppet
4. **Submenu Creator**：创建新的子菜单
5. **Toggle Remover**：删除参数和关联资源（动画、菜单项等）

### 2.2 工具自动完成的工作

> [FACT] 工具官方描述: "all input is error" — Shatteredfur 的设计哲学是自动化避免手动出错

- 添加新参数到 **Expression Parameters**（选中 Avatar）
- 添加参数到 **Animator Controller**（FX 或 Gesture Controller）
- 创建两个 **Animation**（on / off）切换对象状态
- 生成 **Animation Layer**（含所有必要 states 和 transitions）
- 添加 **Toggle** 到 **Expression Menu**
- 同步创建 **Submenu**

### 2.3 字段映射（与官方 Expression Parameters 字段对应）

| 工具字段 | 对应官方字段 | 说明 |
|---------|-------------|------|
| **Save State** | `Saved` | 重新加载 Avatar 时是否**记住状态** |
| **Default State** | `Default` | 加载 Avatar 时**默认显示/隐藏** |

> 详见 `expression-menu.md §9.5`

### 2.4 使用要求

- **Unity 2022.3.6**
- **VRCSDK3**
- **Avatar 3.0**
- 导入 `.unitypackage` 即可

### 2.5 UI 入口

- **Tools 菜单**：`Tools > Toggle Creator` / `Tools > Toggle Remover`
- **GameObject 右键菜单**：Hierarchy 右键 → `Toggle` / `Advanced Toggle`

### 2.6 已知问题

> ⚠️ 报告：**toggles created with this tool are not working when using GoGo Loco**

#### 2.6.1 GoGo Loco 兼容性背景(2026-07-04 补充)

**GoGo Loco** 是 franada 开发的 VRChat Avatar **简易动作管理工具**,由 VRCFury 驱动,在 Play Mode 中完成非破坏性安装。详见 [VRC-Avatar-Performance-Tools（GitHub）](https://github.com/Thryrallo/VRC-Avatar-Performance-Tools) GoGo Loco 章节。

**冲突原因**(**【推断】**):
- GoGo Loco 通过 VRCFury 重新组织 Avatar 的 Animator Controller
- VRChat Toggle Assistant 创建的 toggles **依赖原始 Controller 结构**
- 重新组织后,VRChat Toggle Assistant 的 toggles 失去对原始状态的引用,导致不工作

**创作者建议**:
1. 在启用 GoGo Loco **之前**创建所有 toggles(避免后续冲突)
2. 或使用 VRCFury Toggle 替代(同生态兼容,详见 `vrcfury-reference.md` §2.1)
3. 不要混合使用 VRChat Toggle Assistant + GoGo Loco

### 2.7 许可

- **个人非商业用途**
- 不可再分发
- 可以免费帮别人创建 toggle

---

## 3. Auto-Toggle-Creator（详细）

> [FACT] **来源**: https://github.com/CascadianVR/Auto-Toggle-Creator
> **VPM**: https://github.com/CascadianWorks/VRC-Auto-Toggle-Creator/releases
> **许可**: MIT
> **Stars**: 34+

### 3.1 关键功能

1. 自动生成 **Animation Clip** 和 keyframes（基于 toggle object 名称）
2. 自动设置 **Animator Controller**（新增 layer + parameter）
3. 自动配置 **VRCExpressionsParameters** 资产
4. 自动配置 **VRCExpressionsMenu** 资产

### 3.2 使用步骤

```
1. 下载 .unitypackage 并导入项目
2. 菜单 Cascadian/AutoToggleCreator
3. Auto Fill 按钮自动填入字段
4. 拖入要 toggle 的 GameObject
5. 点 "Create Toggles!" 按钮
6. 上传 VRChat
```

### 3.3 派生项目 VRCToggleToolkit

> [FACT] 来源: https://github.com/SuperFlue/VRCToggleToolkit

- fork of CascadianVR/Auto-Toggle-Creator
- 添加 VPM 安装支持
- **默认 Write Defaults Off**（符合 VRC 最佳实践）
- 每个 toggle 都有 **initialization state**（防止 Avatar 加载异常）
- 集成 Toggle 组合模式（"Combine Toggles"）

---

## 4. AV3ToggleUtil（轻量替代）

> [FACT] **来源**: https://github.com/d4rkc0d3r/av3toggleutil
> **许可**: MIT

- 极简工具，右键 GameObject → Create AV3 Toggle
- 自动添加 Expression Parameter + Animation + Layer
- Stars: 8（社区知名度低但功能可用）

---

## 5. 工具选择决策树

```
开始
├─ 使用 Modular Avatar？
│  ├─ 是 → 用 MA Object Toggle（编译时 + 模块化）
│  └─ 否 → 继续 ↓
├─ 想要一键最简单？
│  ├─ 是 → VRChat Toggle Assistant（功能最全）
│  └─ 否 → 继续 ↓
├─ 倾向 MIT 开源 / VPM 安装？
│  ├─ 是 → VRCToggleToolkit（fork，活跃）
│  └─ 否 → Auto-Toggle-Creator（原始）
├─ 需要最轻量？
│  └─ AV3ToggleUtil（MIT，简单）
└─ 完成后需要测试？
   ├─ Av3Emulator（运行时模拟，详见 avatar-30-emulator.md）
   └─ GestureManager（更美观的菜单）
```

---

## 6. 各工具的限制汇总

| 工具 | 已知限制 |
|------|---------|
| VRChat Toggle Assistant | ⚠️ GoGo Loco 兼容性报告；非商业许可 |
| Auto-Toggle-Creator | 选项较少 |
| VRCToggleToolkit | fork 项目，主分支活跃度需关注 |
| AV3ToggleUtil | 功能极简，社区知名度低 |
| Av3Emulator | ⚠️ 不模拟 VRChat 客户端，模拟的是 PlayableGraph 行为 |

---

## 7. 与 MA 路径的对比

| 维度 | 第三方工具 | Modular Avatar |
|------|-----------|----------------|
| 集成时机 | 编辑器时 | 编译时（NDMF） |
| 学习曲线 | 低（一键生成） | 中（需理解 MA 概念） |
| 依赖 | 无 | 需 MA |
| 模块化 | 弱 | 强（Prefab-based） |
| 与其他 MA 组件集成 | 弱 | 强（Reactive 系统） |
| 适合场景 | 快速原型、简单 Avatar | 复杂 Avatar、模块化 Prefab |

> **推荐**：复杂 Avatar 用 MA 路径，简单/快速原型用第三方工具路径

---

## 8. 完整 Toggle 创建工作流

### 8.1 准备工作

1. **复制 Avatar 沙盒**（避免破坏原始 Avatar）
   - 详见 `avatar-modding-guide.md` 的"Avatar 复制工作流"章节
2. **复制 FX Layer / Expression Menu / Expression Parameters**
3. **创建 Toggles 文件夹**（管理动画文件）

### 8.2 创建 Toggle

1. 选中要开关的 GameObject
2. 选择工具（参考决策树）
3. 配置字段（参考工具详细说明）
4. 创建 Simple Toggle / Advanced Toggle

### 8.3 测试

1. **Av3Emulator 启用**（详见 `avatar-30-emulator.md`）
2. 选中 Avatar
3. "Animator To Debug" 选择 FX Layer
4. 通过 Lyuma Av3 Menu 测试 Toggle
5. 确认行为正确

### 8.4 私有上传测试

1. SDK 上传（Private/Private Friends Only）
2. 在实际 VRChat 中测试
3. 确认 Saved/Default/Synced 行为
4. 公开上传

---

## 9. Missing Information（【未确认】项）

1. ❓ 各工具最新版本与兼容性
2. ❓ Auto-Toggle-Creator 与 VRCFury 的兼容性
3. ❓ 各工具的 Quest 平台支持情况
4. ❓ 各工具的 Performance Rank 影响
5. ❓ Advanced Toggle 在复杂 blendshape 联动时的边缘情况

---

## 来源

- VRChat Toggle Assistant: https://shatteredfur.gumroad.com/l/vrctoggle
- VRChat Toggle Assistant 镜像: https://vrmodels.store/other/scripts/50366-vrchat-toggle-assistant-v0120.html
- Auto-Toggle-Creator: https://github.com/CascadianVR/Auto-Toggle-Creator
- VRCToggleToolkit: https://github.com/SuperFlue/VRCToggleToolkit
- AV3ToggleUtil: https://github.com/d4rkc0d3r/av3toggleutil
- lyuma/Av3Emulator: https://github.com/lyuma/Av3Emulator
- Iconoclass 教程原始: https://docs.vrcd.org.cn/books/avatar-30
- 本地化参考: `笔记/09-avatar-30-toggle/K01-vrchat-toggle-assistant.md` + `K12-avatar-3-toggle-ecosystem.md`

---

## 相关页面

[[expression-menu.md]] · [[playable-layers.md]] · [[avatar-30-emulator.md]] · [[modular-avatar.md]] · [[avatar-modding-guide.md]] · [VRC-Avatar-Performance-Tools（GitHub）](https://github.com/Thryrallo/VRC-Avatar-Performance-Tools) · [[vrcfury-reference.md]]
