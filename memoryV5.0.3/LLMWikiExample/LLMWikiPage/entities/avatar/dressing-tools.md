---
title: "DressingTools (DT) — 非破坏性 cabinet 衣橱/装配系统"
category: avatar
knowledge_level: applied
status: active
source: "github.com/poi-vrc/DressingTools"
source_type: community
version: 1.0
last_review: 2026-08-19
confidence: High
tags:
  - avatar
  - wardrobe
  - non-destructive
  - animator
  - ndmf
  - physbone
aliases:
  - DressingTools
  - dressing-tools.md
  - "poi-vrc DT"
related:
  - lilycal-inventory.md
  - modular-avatar.md
  - ndmf-tools.md
  - expression-menu.md
  - blend-shape-troubleshooting.md
  - physbones.md
  - faceemo.md
  - gesture-manager.md
  - unity-mcp.md
  - avatar-audit-methodology.md
type: entity
created: 2026-08-19
sources: "github.com/poi-vrc/DressingTools"
updated: 2026-08-19
---
# DressingTools — 非破坏性 cabinet（衣橱）/wardrobe 装配系统

> 来源: poi-vrc/DressingTools 官方仓库 + Issues + 社区（检索日期 2026-08-19）
> 置信度: High（核心定位/机制为官方可核实事实；部分兼容细节标 [社区] 或 [UNKNOWN]）
> GitHub: https://github.com/poi-vrc/DressingTools
> VPM 仓库: https://vpm.chocopoi.com/index.json
> 文档: poi-vrc 官方文档站 + VCC 仓库描述

---

## 1. 工具定位

**DressingTools（DT）** 是 poi-vrc 出品的**非破坏性 cabinet（衣橱）/wardrobe 系统**，把多个 wearables（穿戴物）装配到一起，"一键完成"。`2.x`（DressingTools2）已重构为**橱柜系统**（Cabinet）模型 [FACT, github.com/poi-vrc/DressingTools, 2026-08-19]。

| 维度 | 说明 |
|------|------|
| **设计目标** | 把多个 wearable 快速装配进一个 Avatar，"一键完成"衣装管理 |
| **平台属性** | **不是 VRChat 专属**：无 VRCSDK 也可用于新版 Unity / 其他游戏工程 [FACT, 2026-08-19] |
| **核心哲学** | 非破坏性（构建期生成结果，可重算），避免直接手改动画/菜单 |
| **核心特性** | One-click Dressing / Configure in One-screen（自动设置）/ Animation Generation（自动生成 toggle、BlendShape 动画与菜单）[FACT, 2026-08-19] |
| **所属生态** | 基于 DressingFramework（NDMF 支持，实验性多 runtime） |

> 💡 与 `[[lilycal-inventory.md]]` 定位互补：DT 侧重"自动装配 wearable + 生成动画菜单"，LCI 侧重"基于预置组件的衣装切换生成"。选择取决于资产作者支持的工作流与既有工具链。

---

## 2. 核心组件与机制

### 2.1 组件/概念

| 概念 | 说明 |
|------|------|
| **Cabinet（衣柜）** | 衣橱容器，收纳多个 wearable/outfit 条目 |
| **wearable / outfit 条目** | Cabinet 内装的一件/一套衣装 |
| **Dressing Tool 窗口** | 统一配置面板（One-screen） |
| **DressingFramework** | 底层框架（NDMF 支持） |
| **Move wearable root 模块** | 处理无 armature 的配件 |
| **Mapping editor** | 有 armature 的配件手动骨骼映射 |

### 2.2 右键自动装配流程

右键衣装对象 `DressingTools → Setup wearable with editor` 自动装配（[FACT, 2026-08-19]）：

```
1. Armature Mapping          — 骨骼映射（自动/手动）
2. Cabinet animations        — 对象开/关草稿动画
3. Avatar on-wear toggles & blendshapes（可选）
   └ 如胸/脚/高跟鞋体型；按需启用
4. 同名字 BlendShape 同步     — 同名 BlendShape 与 Avatar 同步
5. Add to cabinet            — 生成预览与缩略图
```

> ⚠️ `Move wearable root`（无 armature 配件）与 mapping editor（有 armature 配件）两条路径不同，需按配件是否含骨骼选择。

---

## 3. 工作流步骤（推荐顺序）

```
1. 建 Cabinet（衣柜）
2. 放入衣装（wearable）
3. Setup（右键自动装配）
4. Mapping 检查（骨骼映射是否完整）
5. 配置 animations / on-wear（胸、脚、高跟鞋等体型修正）
6. 【每次装新 wearable】检查同名字 BlendShape 同步  ← 易漏项
7. Add to cabinet（生成预览/缩略图）
8. Play Mode / [[gesture-manager.md]] 验证
```

> ⚠️ 第 6 步"同名字 BlendShape 同步"是关键且易漏：新加衣装后若不重跑同步，原有衣装的 BlendShape 规则可能不同步。这也是排查"换装后身体部分不跟着变"的首要怀疑点（见 §5 故障）。

---

## 4. 底层与规划

### 4.1 DressingFramework 能力（NDMF 支持）

底层基于 DressingFramework（NDMF 支持，实验性多 runtime），提供（[FACT, 2026-08-19]）：

| 能力 | 说明 |
|------|------|
| pass 排序 | NDMF 插件 pass 顺序控制 |
| 动画路径重映射 | 布置过程中把动画绑定路径重映射到目标 Avatar |
| Animator/menu 克隆 | 构建期生成控制器/菜单 |
| 读 DynamicBones / VRCPhysBone | 兼容旧版 DynamicBones 与新版 VRCPhysBone |

### 4.2 DT2 未来规划

DT2 规划（未来支持，[FACT, 2026-08-19]）：

- 自动菜单参数管理
- 混合衣装 toggle（混搭多件衣装的开关）
- 从其他工具转换（迁移入口）
- 可共享 wearable 配置（跨 Avatar 复用）

> [INFERENCE] 这些规划项反映 DT 的演进方向；作为"迁移到其他衣柜工具组/从其他工具迁入"的取舍依据，需按已发布版本核实是否已落地。

---

## 5. 常见故障

以下混合标注来源：官方可核实标 `[FACT]`，社区/Issue 标 `[社区]`，未确认标 `[UNKNOWN]`。

| 故障 | 来源 | 说明 / 处理 |
|------|------|--------------|
| 参数类型不匹配无法上传 | [FACT] | 生成的参数类型与 Animator/菜单引用冲突，检查参数类型与引用一致 |
| VCC 安装/加载失败 | [社区] | VCC 仓库源/版本冲突，核对仓库 URL 与伴随包版本 |
| DT Smart Control Motion Time 默认值无法设置 | [社区] | Smart Control 相关默认值设置异常，检查字段/文档 |
| Cabinet 最多只收录 2 个对象 | [UNKNOWN] | 疑似边界问题（少数案例），**未确认**，需复现核对版本与配置【coverage: low】 |
| 与 MPB（Modular Blend Shapes / 其他玩法生成器）执行顺序冲突 | [社区] | NDMF pass 顺序冲突，调整执行顺序 |
| 不兼容胸骨 → 配重不随动 | [社区] | 骨骼重映射/重命名后配重（Physics/PhysBone）不随动，重映射或重命名骨骼修正 |

---

## 6. 安装与版本

| 项目 | 值 |
|------|-----|
| **VPM 稳定版** | **2.6.0**（Unity ≥2019.4；测试于 2019.4.31f1 / 2022.3.6f1 / 6000.0.32f1）[FACT, 2026-08-19] |
| **Beta 版** | **2.7.0-beta.1** |
| **伴随包** | `com.chocopoi.vrc.dressingframework`（v2.1.1）＋ `com.chocopoi.vrc.avatarlib` |
| **许可证** | GPLv3 |
| **VCC 仓库** | `https://vpm.chocopoi.com/index.json` |
| **安装方式** | VCC / ALCOM、OpenUPM、unitypackage（仅非 Beta 提供）、UPM Git |

> ⚠️ **unitypackage 仅对非 Beta 版发布**；使用 Beta（2.7.0-beta.1）时需走 VPM/OpenUPM/Git，而非 unitypackage。

> 💡 **版本证据（2026-08-19 核实）**：VPM Catalog（chocopoi listing，`vpm.chocopoi.com/index.json`）稳定版列为 **2.6.0**（Unity 2019.4+；运行时依赖 avatarlib 1.1.0 / dressingframework 2.1.1 / Newtonsoft.JSON，vpmDependencies 含 `com.vrchat.avatars >=3.2.0`）；pkglnk 目录最新为 **2.7.0-beta.1**。注意 GitHub Releases 的 `latest` 标签仍停在 **2.5.1**（2024-10）——作者经 VPM 渠道发布新版，**勿把 GitHub Releases 的 latest 标签当最新版**。`[FACT, 2026-08-19]`

---

## 7. 与 MA・NDMF・其他工具关系

| 关系 | 说明 |
|------|------|
| **Modular Avatar** | 官方**无直接 Menu Item 覆盖集成声明**【未确认】【coverage: low】；DT 走**自生成菜单/参数**而非依赖 MA [FACT/UNKNOWN 混合，2026-08-19] |
| **NDMF** | 经 DressingFramework 支持 NDMF，pass 排序与重映射受 NDMF 管理 [FACT, 2026-08-19] |
| **VRCSDK** | 非必需（非 VRChat 专属）；但 VRChat 场景用其 VRCPhysBone 兼容 |
| **lilycalInventory** | 同为衣柜工具，可并行评估（见 §1 定位对比） |

> 💡 走 NDMF 时与 `[[ndmf-tools.md]]` 的执行顺序规则协同；需要骨骼/菜单体系深度融合时，MA 仍是可选项而非硬绑定。

---

## 8. 所有权与责任（方法论提炼）

此节来自本地 playbook（`extended-tool-support.md` / `generated-and-shared-assets.md`）的所有权映射语言，标注为方法论而非官方功能陈述。

| 边界 | 内容 |
|------|------|
| **可编辑源** | wardrobe / wearable 配置：armature mapping、mesh 设置、BlendShape 同步规则、对象切换、约束、菜单设置 |
| **生成输出（不得直接改）** | 生成的动画、菜单、控制器、Animator、NDMF 结果 |

> ⚠️ **不要仅凭"菜单项能用 / 对象可见"判定 success**：必须在目标 root 中核对骨骼路径、BlendShape 绑定、renderer、材质、PhysBones、Contacts 与默认值。菜单可见 ≠ 合身/物理/同步正确。

---

## 9. 常见场景与验证要点

```
"换装后配重不随动" → 查骨骼重映射/重命名（§5 故障）
"新衣装后身体不跟着变" → 重跑同名字 BlendShape 同步（§3 第 6 步）
"上传报参数错误" → 核对生成的参数类型与引用（§5）
"迁移其他衣柜工具组" → 每个旧责任给新 owner 后再移除旧组件（方法论）
```

---

## 10. 相关文档

- [[lilycal-inventory.md]]（姊妹页，另一衣柜系统）
- [[modular-avatar.md]]（菜单/骨骼/装配协作，DT 非硬依赖）
- [[ndmf-tools.md]]（NDMF 生态与执行顺序）
- [[blend-shape-troubleshooting.md]]（BlendShape 同步排查）
- [[gesture-manager.md]]（Play Mode 预览/诊断验证）
- [[physbones.md]]（物理/配重责任）
