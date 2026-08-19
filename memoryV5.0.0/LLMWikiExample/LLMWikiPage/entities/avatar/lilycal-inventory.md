---
title: "lilycalInventory (lilycal 衣装切换/衣柜)"
category: avatar
knowledge_level: applied
status: active
source: "github.com/lilxyzw/lilycalInventory"
source_type: community
version: 1.0
last_review: 2026-08-19
confidence: High
tags:
  - avatar
  - wardrobe
  - non-destructive
  - animator
  - expression-menu
  - ndmf
aliases:
  - lilycalInventory
  - lilycal-inventory.md
  - "lilxyzw 衣柜"
related:
  - dressing-tools.md
  - modular-avatar.md
  - ndmf-tools.md
  - expression-menu.md
  - animator-system.md
  - unity-mcp.md
  - faceemo.md
  - gesture-manager.md
  - avatar-audit-methodology.md
type: entity
created: 2026-08-19
sources: "github.com/lilxyzw/lilycalInventory"
updated: 2026-08-19
---
# lilycalInventory — 非破坏性衣装切换/衣柜工具

> 来源: lilxyzw/lilycalInventory 官方仓库 + 官方文档（检索日期 2026-08-19）
> 置信度: High（核心机制为官方可核实事实，边界为推断并标注）
> 官方一句话: "**Modify avatar at build time.**"（构建时修改 Avatar）
> GitHub: https://github.com/lilxyzw/lilycalInventory
> 文档: https://lilxyzw.github.io/lilycalInventory/
> VPM 仓库: https://lilxyzw.github.io/vpm-repos/vpm.json

---

## 1. 工具定位

**lilycalInventory（下称 LCI）** 由 lilxyzw（lilToon 的作者）出品，是一个**非破坏性"衣柜 / 衣装切换"生成式工具**。它不在编辑期直接改模型，而是通过组件声明"要怎么做"，在**上传 / 构建时**自动完成动画、菜单、参数的生成。官方将其定义为一句话 "**Modify avatar at build time.**" [FACT, github.com/lilxyzw/lilycalInventory, 2026-08-19]。

| 维度 | 说明 |
|------|------|
| **设计目标** | 让制作者用组件配置"衣装开/关、多衣装排他切换、无级调节"，构建时自动产出菜单 + Animator + 参数 |
| **核心哲学** | **不生成资产（アセットを生成しない）**——构建时生成的东西是输出，不是可编辑源 [FACT, 2026-08-19] |
| **同步事故防范** | 当所有参数全关时，生成结果的外观与 prefab 源状态保持一致（避免"关掉所有开关后模型消失/残留"的经典事故）[FACT, 2026-08-19] |
| **非目标** | 骨骼装配 / 合并不在其职责内（推断边界，见 §5） |
| **所属生态** | NDMF 插件（有 NDMF 时作为 NDMF 插件运行），也可独立运行 |

> 💡 LCI 与 lilToon **没有依赖关系**：lilToon 只是同一作者的另一套独立资产；LCI 只关心"衣装怎么切换"，不关心 shader。

---

## 2. 核心组件与机制

### 2.1 组件表（前缀 `LI`）

| 组件 | 类型 | 生成内容 | 参数类型 |
|------|------|----------|----------|
| **LI MenuFolder** | 菜单容器 | 菜单分组的文件夹 | 不生成参数 |
| **LI ItemToggler** | 对象开关 | 对象开关 + BlendShape + 材质替换 | Bool |
| **LI CostumeChanger** | 衣装切换 | 多衣装排他切换，构建时转 AnimationClip + AnimatorController State | Int |
| **LI AutoDresser** | 换装根部 | 构建时转成 CostumeChanger，统一管理多件衣装 | Int |
| **LI AutoDresserSettings** | 换装设置 | 每 Avatar 0/1 个，配置 AutoDresser 行为 | — |
| **LI SmoothChanger** | 无级调节 | Float 参数驱动的连续值切换 | Float |
| **LI Preset** | 预设 | 一组衣装/对象的组合 | — |
| **LI Prop** | 道具 | 道具放置/切换 | — |
| **LI MaterialModifier** | 材质修改 | 构建时改写材质 | — |
| **LI MaterialOptimizer** | 材质优化 | 可选（协作者可装） | — |
| **LI Comment** | 注释 | 无运行时效果，纯说明 | — |
| **LI AutoFixMeshSettings** | 网格修复 | 自动修正 Mesh 设置 | — |

> ⚠️ 组件名与语义以**实际安装版本**为准，升级后会变化；排查前先经 Unity MCP 确认识别到的具体字段名（见 §7 方法论）。

### 2.2 LI CostumeChanger 构建机制

构建时对每套衣装执行（官方机制，[FACT, 2026-08-19]）：

```
1. 为每套衣装生成 AnimationClip
   └ 组件设定值 + prefab 初始值「合成」为一个 Clip
2. 注入同名 Int 参数
   └ 同时写入 AnimatorController 与 ExpressionParameters
3. 复制「有効状態を保存 / ローカルのみ」语义
   └ 决定换 Avatar / 换世界时是否保留、是否仅本机生效
4. 新增 Animator 层 / State / Clip / Transition
5. Toggle 菜单写 Int 值
```

> 关键点：生成 Clip 是「组件设定 + prefab 初始值」的合成结果。若 prefab 初始状态与组件设定冲突，最终以合成结果为准——排查"为什么生成的默认状态不对"时应同时核对源 prefab 活性与组件设定。

### 2.3 菜单项参数语义

LCI 生成的菜单项包含（[FACT, 2026-08-19]）：

| 概念 | 含义 |
|------|------|
| 图标 | 菜单按钮显示的图标 |
| 菜单・参数名 | 生成到菜单/参数系统的名称 |
| **有効状態を保存** | 换 Avatar / 换 World 后该开关是否被保留 |
| **ローカルのみ** | 仅本地生效（不上传/不参与网络同步） |
| 默认值 | 首次进入时的参数初值 |

> ⚠️ **有効状態を保存 / ローカルのみ** 是"已保存/同步"语义，必须在生成结果里核对，不能只看菜单按钮能不能按。

---

## 3. 体积/参数与成本

- LCI 本身是**运行时 + 编辑器**包，构建时产物是菜单/控制器/参数/动画（不占 Avatar 运行时脚本开销，机制在构建期完成）[FACT, 2026-08-19]。
- **参数预算**：每个 ItemToggler 占用 1 个 Bool；每个 CostumeChanger / AutoDresser / SmoothChanger 占用 1 个 Int（或多个值）。参数数量与衣装数量成正比，**多衣装 Avatar 尤需关注 256 位参数预算**。
- **疯狂装衣装的后果**：参数接近或溢出预算。排查"SDK 提示参数超预算"时，先核对 LCI 生成的参数清单与服务商贡献，不要靠数参数名估算（见 §6 故障）。
- [UNKNOWN] LCI 生成物对最终 SDK 体积的具体增量未逐一量化，需授权构建比对后才下结论（【coverage: low】）。

---

## 4. 与 MA・NDMF・其他工具关系

| 关系 | 说明 |
|------|------|
| **独立运行** | 无 NDMF 时也能工作（自生成菜单/控制器/参数）[FACT, 2026-08-19] |
| **NDMF 插件** | 有 NDMF 时作为 NDMF 插件运行，遵循 NDMF 运行顺序预设 [FACT, 2026-08-19] |
| **Modular Avatar 集成** | 组件可设 **MA Menu Item 覆盖**；纳入 **MA Menu Group** 时自动生成 MA Menu Item，菜单并入 MA 体系 [FACT, 2026-08-19] |
| **lilToon** | 非依赖，可有可无 |
| **VPM manifest** | **无 `vpmDependencies`**：NDMF / lilToon / MA 均为**可选集成、非硬依赖** [FACT, 2026-08-19] |

> 💡 与 `[[dressing-tools.md]]` 同为"衣柜"工具，但定位不同：LCI 是"基于预置组件 + 构建时生成"，DressingTools2 是"自动装配 wearable 的 cabinet 系统"。选择取决于资产作者支持的工作流与团队既有工具链。

---

## 5. 常见故障（CHANGELOG 修复项）

| 故障现象 | 说明 / 处理 |
|----------|-------------|
| 旧 VRCSDK / 空处理对象 / 文件夹循环引用 | CHANGELOG 修复项：检查 VRCSDK 版本、清理空对象、避免文件夹循环引用 |
| MA 未装时 AnimatorController 克隆问题 | 无 MA 场景下控制器克隆异常；装 MA 或按官方依赖指引补齐 |
| 参数重复错误报告 | 参数名冲突/重复 → 有超预算风险；核对生成的参数清单并去重 |
| 被引用文件夹组件关闭时不显示 | 组件被关但被别处引用时表现异常，检查引用链 |
| VRCAnimatorPlayAudio 报错 | 音频动画节点相关错误，检查相关播放对象 |
| 子菜单为空报错 | 生成的子菜单无内容，核对菜单放置定义 |

> [UNKNOWN] 官方**无独立中文/英文排障页**【未确认】（官方文档以日文为主，英文化覆盖有限）。排查仍以 CHANGELOG + GitHub Issues + 官方仓库为准。【coverage: low】

---

## 6. 安装与版本

| 项目 | 值 |
|------|-----|
| **最新版本** | **1.5.2**（Unity 2022.3）[FACT, 2026-08-19] |
| **许可证** | MIT |
| **VCC 仓库** | `https://lilxyzw.github.io/vpm-repos/vpm.json` |
| 安装方式 | VCC / ALCOM（仓库添加）、vrc-get、.unitypackage、UPM Git |

> ⚠️ 版本升级（尤其破坏性版本）可能要求迁移既有 `LI` 组件，升级前备份 Avatar / 记录组件版本。

---

## 7. 所有权与质量责任（方法论提炼）

此节来自本地 playbook（`lilycal-inventory-playbook.md` / `generated-and-shared-assets.md`）的**所有权映射语言**，标注为方法论而非官方功能陈述。

### 7.1 可编辑源 vs 生成输出

| 边界 | 内容 |
|------|------|
| **可编辑源（源）** | 可穿戴源 / provider 组件 / Inventory / AutoDresser 配置 / 菜单放置 / 参数映射 |
| **生成输出（不得直接改）** | 生成的菜单 / 控制器 / 参数 / 构建克隆 / NDMF 结果 |

> ⚠️ 官方声明构建时**不产生持久资产**；但 NDMF 构建过程会产生生成物。这些生成物**不是编辑入口**——要改就改源组件，再走 provider 预览/NDMF 重算，不要手改生成控制器/菜单。

### 7.2 质量责任边界

- **LCI 会负责**：对象开关、BlendShape 表达、材质替换、菜单/参数/动画的生成（能表达"穿模/体型变化"这类依赖 Bool/BlendShape 的结果）。
- **[INFERENCE] LCI 不负责**：骨骼装配/骨骼合并（armature merge、bone proxy）。这类责任归 `[[modular-avatar.md]]`（MA Bone Proxy / Merge Armature）或其他装配工具。这个边界是**推断**，不是官方功能陈述，需按具体组件确认。
- **一个可见菜单按钮 ≠ 衣装正确**：菜单可达只证明"能打开"，不证明 fit（合身）、穿模、动作、已保存状态、同步、物理正确。最终要以构建克隆 + 授权运行层验证。

---

## 8. 使用流程（方法论建议）

```
1. 记录目标 Avatar / 衣装层级、LCI 版本、组件清单（经 Unity MCP/本地证据）
2. 建立责任映射：入口/参数/可见性/贴体/网格/物理/预设/生成结果 各找唯一 owner
3. 添加最小 provider 支持的源组件（LI ItemToggler / CostumeChanger / AutoDresser）
4. 谨慎选择 参数/默认值/保存/同步 语义，不复制兄弟项的值
5. 走 provider / NDMF 路径重新生成
6. 核对生成菜单、参数、Animator、层级与责任映射的差异
7. 只在授权运行层验证合身/可见/运行时行为，再判完成
```

> 💡 LCI 的"衣装"与 MA 的"Avatar 参数/菜单体系"协同最稳的姿势：LCI 管衣装切换生成，MA 管菜单体系/骨骼/合并，二者按 NDMF 顺序协同。

---

## 9. 相关文档

- [[dressing-tools.md]]（姊妹页，另一衣柜系统）
- [[modular-avatar.md]]（菜单/骨骼/装配协作）
- [[ndmf-tools.md]]（NDMF 生态与执行顺序）
- [[expression-menu.md]]（菜单体系）
- [[unity-mcp.md]]（Unity 内核对组件字段的方法）
- [[avatar-audit-methodology.md]]（责任映射与审计方法）
