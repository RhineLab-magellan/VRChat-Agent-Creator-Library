---
title: Action / Expression / Puppet Menu — Avatar 菜单三层结构
category: avatar

knowledge_level: applied
status: active

tags:
  - avatar
  - action-menu
  - expression-menu
  - puppet-menu
  - widget
  - ui

aliases:
  - Action Menu
  - Expression Menu
  - Puppet Menu
  - Flick Select
  - Pick Select
  - 动作菜单
  - 表情菜单
  - 控件
  - 手柄菜单

related:
  - playable-layers.md
  - modular-avatar.md
  - standard-hand-poses.md
  - skeletal-input.md
  - animator-system.md
  - ../platform/mobile-ui-optimization.md

source: docs.vrchat.com/docs/action-menu
source_type: official
version: 1.1
last_review: 2026-07-04
confidence: High
type: entity
created: 2026-07-04
sources: docs.vrchat.com/docs/action-menu
updated: 2026-07-04
---

# Action / Expression / Puppet Menu — Avatar 菜单三层结构

> 来源: https://docs.vrchat.com/docs/action-menu
> 本地化日期: 2026-06-30 (原始) / 2026-07-04 (补充 §9.5 + §9.6)
> 状态: ✅ FACT (VRChat 官方 Action Menu 文档) + §9.5/§9.6 官方多源验证
> 关联: `avatar/playable-layers.md` (FX Layer 接收菜单参数) + `avatar/modular-avatar.md` (Full Controller 组件) + `avatar/avatar-30-emulator.md` (Av3Emulator 测试) + `avatar/avatar-30-toggle-tools.md` (非 MA 工具)

---

## 概述

> [FACT] **Action Menu** 是访问 Avatar 控件和**快速访问**控件的**主要方式**。
>
> - 当使用 **Avatars 3.0** Avatar 时，这些控件**完全可定制**
> - 可作为**多种类型**之一（Button/Toggle/Slider/Stage/Puppet...）
> - **驱动** Animator Parameters 告知 Avatar 如何反应

> **🔴 关键事实（创作者角度）**:
> - 玩家在 Action Menu 中做的**任何操作** = 触发 **Animator Parameter 变化**
> - 创作者需在 **FX Layer** 中**监听**这些参数
> - 菜单结构是**三层**: Action Menu → Expression Menu → Puppet Menu

---

## 1. 三层菜单结构

> [FACT] **UI 有 3 个层级**:
> 1. **Action Menu**（根菜单）
> 2. **Expression Menu**（表达式菜单）
> 3. **Puppet Menu**（木偶菜单）

### 1.1 Action Menu（根菜单）

> [FACT] **Action Menu** 是**根菜单**。
>
> 当前**包含内容**:
> - **Gesture Toggle**（手势开关）
> - **Config**（UI 配置）
> - **Emojis**（表情）
> - **Expression Menu**（进入下层）
> - **Close**（关闭）

### 1.2 Expression Menu（表达式菜单）

> [FACT] **Expression Menu** 可包含各种 **Widgets**（控件），包括 **Stages**（舞台）。
>
> - 进入**包含 Puppets 的 Stage** = 进入该 Stage 的 **Puppet Mode**
> - **每个 Expression Menu** 可有**多个 Stage**
> - 可**同时在每只手上**打开菜单

#### 1.2.1 Widgets（控件）

> [FACT] Widgets 包括:
> - **Button**（按钮）
> - **Toggle**（开关）
> - 其他控件类型
>
> **创作者可配置**这些 Widgets，**它们以各种方式作用**于 Animator Parameters。

#### 1.2.2 Stages（舞台）

> [FACT] **Stage** 是一种特殊 Widget，用于**进入 Puppet Mode**。
>
> - Stage **包含 Puppets**（木偶）
> - 进入 Stage 后**进入 Puppet Mode**（驱动手柄/鼠标值）
> - 创作者可定义**多个 Stage**（如 "Emotes" / "Mood" / "Toggles"）

### 1.3 Puppet Menu（木偶菜单）

> [FACT] **Puppet Menu** 驱动 Avatar 的各种**值/参数**。
>
> - **构建 Avatars 3.0 Avatar** 时由创作者**映射**这些参数
> - 在此模式中，**移动摇杆** / **触摸 Vive 触摸板** / **使用鼠标** 即可"玩"木偶效果
>
> **使用方式**:
> - **移动摇杆** → 驱动 Animator Parameter 连续变化
> - **停止移动** → 参数回到默认
> - 可实现**混合动画**（如 "happy" + "surprised" 同时存在）

> [FACT] **从 Puppet Menu 返回 Expression Menu**:
> - **拉 Trigger**（手柄归中时）= 回到 Expression Menu
> - 再拉 Trigger = 回到 Action Menu

> **Puppet Menu 示例效果**: 不同方向摇杆 = 不同心情（happy / surprised / sad...），可**连续混合**。
> 详见 §5 示例

---

## 2. 两种选择方式：Flick vs Pick

> [FACT] 径向菜单中有**两种选择方法**。

### 2.1 Flick Select（弹选）⭐ 默认

> [FACT] **Flick** 模式:
> - **推动摇杆**指向想要的扇区方向
> - **推动一段** → 显示**指示器**（预览）
> - **完全推到底** → **触发选择**
>
> 类比: **像"弹"出**选择（flick = 弹）

### 2.2 Pick Select（点选）

> [FACT] **Pick** 模式:
> - **推动摇杆**指向想要的扇区方向
> - **再用 Trigger 确认**选择（桌面用鼠标移动 + LMB）
> - 两步选择
>
> 类比: **像"瞄准后扣扳机"**

### 2.3 默认值

> [FACT] **默认是 Flick**（Vive 除外）。
>
> 切换方法: **Action Menu > Settings > Flick enabled** 设置。

### 2.4 桌面模式

> [FACT] 桌面模式**特殊行为**:
> - **只能打开一个菜单**（不能双开）
> - **必须点击确认**选择
> - **Flick** 模式 = 鼠标移动 + 点击

---

## 3. 关闭与记忆（Closing and Saving）

> [FACT] **菜单关闭机制**:

### 3.1 Back 选项

> [FACT] 在 **Action / Expression 菜单**或任何子文件夹中:
> - 选 **Back** 选项 = 退出文件夹/菜单（breadcrumb 上移）
> - 持续选 Back = 关闭菜单

### 3.2 摇杆点击（短期关闭）

> [FACT] **单次点击摇杆**:
> - **关闭菜单**
> - **记忆**当前所在位置
> - 下次打开 = **回到上次位置**

### 3.3 应用场景

> [FACT] 这个"记忆位置"设计:
> - 玩家**常用**某个深嵌套菜单 → 不必每次都从根目录导航
> - **来回切换** 主菜单和深嵌套菜单的体验**流畅**
> - **创作者** 设计菜单时考虑玩家的"工作流路径"

---

## 4. 控制器操作详解

> [FACT] 各控制器打开 Action Menu 的方法:

### 4.1 Valve Index

> [FACT] **打开方式**: 点击对应手的 **Joystick**（默认）
> **Back / 选 Pick 模式**: 用 **Trigger**

### 4.2 Oculus Touch

> [FACT] **打开方式**: 点击对应手的 **Joystick**（默认）
> **Back / 选 Pick 模式**: 用 **Trigger**

### 4.3 Vive Wand

> [FACT] **打开方式**: **长按**控制器顶部的 **Menu 按钮**
>
> ⚠️ Vive 与 Index/Touch **不同**: Vive 必须长按

### 4.4 其他带摇杆的控制器

> [FACT] **打开方式**: **点击摇杆**（Joystick In）
> **Back / 选 Pick 模式**: 用 **Trigger**

### 4.5 通用方法

> [FACT] **所有控制器**: **长按 Quick Menu 按钮** 也能拉起 Action Menu

### 4.6 桌面模式

> [FACT] 桌面模式: **按 R 键** 拉起 Action Menu。
> - 移动鼠标选择
> - 点击确认
> - 点击 Back 选项退出

> 详细见 `avatar/standard-hand-poses.md` §5 (Action Menu 唤起方式对照表)

---

## 5. Puppet Menu 示例

> [FACT] **Puppet Menu 的典型用法**:
>
> <Image title="Puppet Menu" align="center" width="800px" src="https://media2.giphy.com/media/c63LdtLhEHe95HGbWC/giphy.gif" />
>
> 玩家移动摇杆/触摸板/鼠标到**不同方向** = 驱动**不同** Animator Parameter 值。
> 示例设置允许在**各种心情间混合**（如 "happy" 和 "surprised" 任意混合）。

> **关键事实**:
> - **任何可参数化的东西**都能从 Puppet Menu 读取
> - 包括 **Toggle on/off 按钮**和其他 UI 元素
> - **不止**示例中展示的"心情"用法
> - ⚠️ 部分 UI 元素类型（如新的 Toggle）**VRChat 仍在开发中**（源文档原文 "still working on"）

### 5.1 双菜单支持

> [FACT] **菜单可同时在两只手打开**（或单手）。
> - 默认: Flick 模式
> - 在 Action Menu UI 内的 **Settings** 中可切换为 Trigger 确认

### 5.2 退出 Puppet Menu

> [FACT] **退出 Puppet 回到 Expression 菜单**:
> - **拉 Trigger** 一次 = 回到 Expression 菜单
> - **再拉 Trigger** 一次 = 回到 Action 主菜单

---

## 6. Debug Menu（创作者/高级玩家）⭐

> [FACT] **Debug Menu** 在 Action Menu 中**可用**。
>
> - 弹出**文本显示**
> - 显示 Avatar **当前 animator state**
> - 包含: **parameter values / tracking states / current motion states** 等详情
> - **调试时非常有用**

> **创作者**应:
> - 在调试 Avatar 时**使用** Debug Menu
> - 查看参数是否按预期变化
> - 检查 motion state 是否正确触发

---

## 7. 平台限制

> [FACT] **Quest Hand Tracking Beta**:
> - **目前无法访问** Action Menu
> - **替代方法**: 使用 **Quick Menu wing** 控制 Avatar
>
> 详见 `platform/mobile-ui-optimization.md`

---

## 8. 创作者设计建议

> [FACT] **菜单设计的最佳实践**（基于官方文档）:

### 8.1 控件选择

> [FACT] **根据功能选 Widget**:
> - **瞬时动作**（如挥手）→ **Button**
> - **二值状态**（如开/关帽子）→ **Toggle**
> - **进入子菜单**（如情绪控制）→ **Stage**
> - **连续控制**（如表情强度）→ **Puppet**

### 8.2 菜单深度

> [FACT] **考虑玩家导航成本**:
> - 玩家每次"Back"操作**消耗操作**
> - **太深嵌套** = 玩家不常用该功能
> - **记忆位置**机制可缓解，但**不要**滥用嵌套

### 8.3 双手同时打开

> [FACT] 玩家可**同时在两只手打开菜单**（独立 Expression Menu）。
>
> **创作者角度**:
> - **可利用**这个特性做**双手协调**功能
> - 例如: 左手调音量 / 右手调表情
> - **不要假设**玩家只用单手

### 8.4 Puppet Menu 设计

> [FACT] Puppet Menu 适合:
> - **连续变化**参数（不限于二值）
> - **多维控制**（如 2D 摇杆 = 2 个参数）
> - **混合动画**（让玩家"摸索"有趣的中间状态）

### 8.5 Bindings 自定义

> [FACT] 玩家在 **Action Menu > Settings > Bindings** 中可**自定义**:
> - **打开 Action Menu** 的方式
> - 常见自定义示例: **"clicking in the joystick"**（设置点击摇杆打开 Action Menu）
>
> 创作者应**避免**假设默认绑定（玩家可能改过）。

---

## 9. 玩家常见问题

### Q1: Quest Hand Tracking 怎么用 Action Menu?

> 目前**不支持**。用 **Quick Menu wing** 替代控制 Avatar。

### Q2: 菜单"记住位置"怎么清掉?

> **完全关闭**菜单（Back 到顶层）即可。下次打开从**根菜单**开始。

### Q3: Flick vs Pick 怎么选?

> [FACT] **Flick 优势**:
> - **单手单步**操作（更快）
> - 适合**频繁切换**（如战斗中切换装备）
>
> **Pick 优势**:
> - **误触少**（需二次确认）
> - 适合**重要但罕见**操作（防误触）

### Q4: Puppet Menu 只能控制"心情"吗?

> **不是**。任何 Animator Parameter 都可被 Puppet 驱动。示例中的"心情"只是**最直观**的用法。

### Q5: 两手同时开菜单有什么区别?

> [FACT] 每只手**独立**打开 **Expression Menu**（可在每个手上配不同 Expression）。
> - 玩家可在**左手**开 "Emotes" + **右手**开 "Toggles"
> - **互不干扰**

---

## 9.5 Expression Parameters 字段详解（官方）

> [FACT] **来源**:
> - https://creators.vrchat.com/avatars/expression-menu-and-controls/
> - https://wiki.vrchat.com/wiki/Expressions
> - https://vrc.school/docs/Avatars/Expressions-Menu-Params/
> - **本地化日期**: 2026-07-04
> - **状态**: ✅ FACT (VRChat 官方文档 + 社区权威多源验证)
> - **关联**: 本节补充原文档缺失的 Expression Parameter 字段语义

> **🔴 关键事实**:
> - 每个 Expression Parameter 有 **5 个字段**（Name/Type/Default/Saved/Synced）
> - **Saved** 和 **Synced** 是最容易被混淆的两个字段
> - Default 字段定义"首次穿戴/重置 Avatar 时的初始值"

### 9.5.1 五个字段完整语义

> [FACT] VRChat 官方 Expression Parameters 资产每个参数有 **5 个字段**：

| 字段 | 说明 | 行为 |
|------|------|------|
| **Name** | 参数名 | 必须与 Animator parameter 名**完全匹配** |
| **Type** | 类型 | `Bool` / `Int` / `Float` |
| **Default** | 默认值 | Avatar **重置**时使用的值（首次穿戴/重连世界） |
| **Saved** | 持久化 | 是否在**会话间保存**最后一次值 |
| **Synced** | 同步 | 是否发送给其他玩家 |

### 9.5.2 Saved vs Synced 关键区分

> [FACT] 这是**最容易被混淆**的两个字段：

| 维度 | Saved | Synced |
|------|-------|--------|
| 作用范围 | 本地用户 | 其他玩家 |
| 行为 | 记住本地用户上次值 | 发送给其他玩家 |
| 关闭原则 | 临时效果、调试用 | 仅本地逻辑用 |

> **关键引用**: "Saved and Synced are different settings. Saved remembers the local user's value; Synced sends the current value to other players when that remote visibility is needed." — https://vrcreators.net/docs/vrchat/avatar-creation/saving-toggles

### 9.5.3 Default 字段的设计原则

> [FACT] 来源: https://vrcreators.net/docs/vrchat/avatar-creation/saving-toggles
> "Saved does not replace a good default. The default is what new users see when they first wear the avatar, and what unsaved parameters return to after reset."

**"Boring / predictable defaults" 原则**:
- 服装在正常 safe state
- 道具默认隐藏
- 效果默认关闭
- sliders 设为视觉中性值

### 9.5.4 何时启用 Saved

| 应该 Saved | 不应该 Saved |
|-----------|-------------|
| 玩家选定的服装 | 一次性动画 |
| 发型 / 材质颜色 | 调试 toggle |
| 经常使用的 accessories | 一次性效果 |
| 玩家偏好保持的状态 | 临时状态 |

### 9.5.5 同步预算

> [FACT] VRChat 同步限制（来源: https://creators.vrchat.com/avatars/animator-parameters/）:

| 指标 | 数值 |
|------|------|
| **总同步预算** | 256 bits（约 128 bits 老 SDK） |
| **总参数数** | 8192 个 custom Expression Parameters |
| **Bool 占用** | 1 bit |
| **Int / Float 占用** | 8 bits |
| **Float 同步精度** | 1/127（-1.0 到 1.0） |
| **Int 同步范围** | 0-255 |

### 9.5.6 跨平台同步

> [FACT] 跨平台（PC / Quest）参数同步规则:
> - **按位置 + 类型**同步,**不是按名称**
> - 同位置的参数 + 同类型 = 跨平台同步
> - 改变参数顺序会导致跨平台同步错位

> **关联工具**: VRChat Toggle Assistant 的 "Save State" 字段对应此处的 `Saved`，"Default State" 对应 `Default`。

### 9.5.7 参数类型位宽(2026-07-04 来自 yexcadocs)

> 来源:yexcadocs §Parameters(P2)
> 本节为社区实操经验补充。

**Avatar 总参数限额 = 256 bits**(完整同步预算与 9.5.5 一致)

| 类型 | 占用 | 取值范围 |
|------|------|---------|
| **int** | 8 bits | -128 ~ 127(同步 0-255) |
| **float** | 8 bits(8-bit minifloat) | -1.0 ~ 1.0 |
| **bool** | 1 bit | true / false |

**计算示例**:
- 8 个 Float = 64 bits
- 16 个 Int = 128 bits
- 64 个 Bool = 64 bits
- **总计 256 bits = 满预算**

---

## 9.55 菜单控件官方原文(2026-07-04 来自 yexcadocs)

> 来源:yexcadocs §VRChat Menu(P2)
> 本节为社区实操经验补充。

### Button 控件

> **Button**: Cannot be held down(VRChat 官方原文)
> - Button **不能长按**
- 触发后立即恢复默认状态
- 与 Unity Button 的"按下"行为不同
- 适合瞬时动作

### Sub-Menu 控件

> **Sub-Menu**: You can put sub-menus into sub-menus!(VRChat 官方原文)
> - **支持子菜单嵌套子菜单**
- 允许创建深嵌套结构
- 考虑 §8.2 菜单深度设计原则(避免过度嵌套)

---

## 9.6 自定义 Menu 替换默认 Emotes（关键警告）⭐

> 🔴 **[FACT] 关键行为**:
> **来源**: https://wiki.vrchat.com/wiki/Expressions
> **本地化日期**: 2026-07-04
> **关联**: 本节填补创作者"自定义 Menu 替换默认 Emotes"的关键警告

> **当 Avatar Descriptor 的 Menu 字段被赋值为自定义 Expression Menu 资产时，默认的 Emotes Menu 会被替换。**

### 9.6.1 默认行为

> "When no expressions are configured, a default expression menu with base animations is added to the avatar, containing the following animations:"
> — VRChat Wiki

| Avatar 状态 | 玩家看到的菜单 |
|------------|---------------|
| **未配置自定义 Menu** | 自动显示默认 Emotes Menu（含基础动画） |
| **配置了自定义 Menu** | 默认 Emotes Menu **消失**,只显示你的自定义 Menu |

### 9.6.2 含义

- 玩家**失去**默认表情 / 姿势选择能力
- 必须**自己创建**等效的子菜单
- 这是 VRChat 的**官方行为**,不是 bug

### 9.6.3 解决方案

**方案 A：手动重建 Emotes 子菜单**（推荐）

```
1. 创建新的 Expression Menu 资产
2. 添加 "Emotes" 子菜单（包含原默认表情）
3. 8 个以内控制（或更深的子菜单嵌套）
```

**方案 B：使用 Prefabulous 工具**

- 工具：💬 Blank Expressions Menu and Parameters
- 来源：https://docs.hai-vr.dev/docs/products/prefabulous/vrchat/blank-expressions-menu-and-parameters
- 作用：用**空白** menu 和 parameters **替换**原 menu
- 优势：**早期处理**，确保其他 non-destructive 工具从 blank state 开始
- 版本：1.2.0+

### 9.6.4 创作者设计清单

设计自定义 Expression Menu 时,必须考虑:
- [ ] 是否提供等效的 Emotes 子菜单？
- [ ] 是否提供 Reset Avatar 的方式？
- [ ] 玩家是否知道怎么"回退"到根菜单？
- [ ] 默认子菜单的命名是否直观？

### 9.6.5 关联警告

> "使用自定义 Expression Menu 将删除默认的 emotes menu，因此需要创建一个类似于 emotes menu 的新子菜单来替代。" — Iconoclass 教程（VRCLibrary）

---

## 10. 与其他文档的关系

| 文档 | 关系 |
|------|------|
| `avatar/playable-layers.md` | FX Layer 接收菜单驱动参数 |
| `avatar/animator-system.md` | Animator Parameter 来源之一就是菜单 |
| `avatar/modular-avatar.md` | `Full Controller` 组件合并完整 animator/expression menu/parameters |
| `avatar/standard-hand-poses.md` | 控制器按钮唤起 Action Menu 的细节 |
| `avatar/skeletal-input.md` | Hand Tracking 当前不支持 Action Menu |
| `platform/mobile-ui-optimization.md` | Quest 替代 UI 方案 |
| **`avatar/avatar-30-emulator.md`** ⭐NEW 2026-07-04 | Av3Emulator 测试 Expression Menu 行为 |
| **`avatar/avatar-30-toggle-tools.md`** ⭐NEW 2026-07-04 | 非 MA 路径的 Toggle 工具生态（VRChat Toggle Assistant / Auto-Toggle-Creator） |
| **`avatar/avatar-modding-guide.md`** ⭐更新 2026-07-04 | Avatar 复制工作流（沙盒创建） |

---

## 11. Missing Information（【未确认】项）

> 以下信息需要进一步验证或在官方文档中查找:

1. ❓ **Widgets 的完整类型列表**（除 Button/Toggle/Stage/Puppet 之外是否还有其他）
2. ❓ **Stages 嵌套 Stages** 是否支持
3. ❓ **菜单打开/关闭** 的精确事件回调（创作者能否在 Udon 中监听）
4. ❓ **Puppet Menu 持续时间**对 Performance Rank 的影响
5. ❓ **Flick 灵敏度** 是否可调
6. ❓ **菜单 Position 记忆** 的具体机制（持久化到哪个文件）
7. ❓ **多 Stage 切换** 是否消耗带宽
8. ❓ **Bindings 自定义** 能否保存到云端
9. ❓ **Saved 字段的云端备份机制**（是否与玩家账号绑定）
10. ❓ **Saved 与 Performance Rank 的关系**

---

## 来源

- [VRChat Action Menu](https://docs.vrchat.com/docs/action-menu)
- [What is Avatars 3.0?](https://docs.vrchat.com/docs/what-is-avatars-30)
- [VRChat Creator Docs - Expressions Menu and Controls](https://creators.vrchat.com/avatars/expression-menu-and-controls/)
- [VRChat Creator Docs - Animator Parameters](https://creators.vrchat.com/avatars/animator-parameters/)
- [VRChat Wiki - Expressions](https://wiki.vrchat.com/wiki/Expressions)
- [VRC School - Expressions Menu and Parameters](https://vrc.school/docs/Avatars/Expressions-Menu-Params/)
- [VRC Avatars for Dummies - Saving Toggles](https://vrcreators.net/docs/vrchat/avatar-creation/saving-toggles)
- [Prefabulous - Blank Expressions Menu and Parameters](https://docs.hai-vr.dev/docs/products/prefabulous/vrchat/blank-expressions-menu-and-parameters)
- 本地化版本: （来源：SP 文档本地译本）

---

## 相关页面

[[playable-layers.md]] · [[modular-avatar.md]] · [[standard-hand-poses.md]] · [[skeletal-input.md]] · [[animator-system.md]] · [[../platform/mobile-ui-optimization.md]]
