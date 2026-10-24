---
title: "Avatar 优化工具链组合指南（7 工具职责边界与共存风险）"
category: avatar
knowledge_level: applied
status: active
source: "anatawa12/AvatarOptimizer · RamType0/Meshia.MeshSimplification · ReinaS-64892/TexTransTool · limitex/avatar-compressor · VRCFury · Null-K/MA2BT · zhuozhi233/MA2BT-Pro · Thryrallo/VRC-Avatar-Performance-Tools"
source_type: community
version: 1.0
last_review: 2026-10-24
confidence: Medium
tags:
  - avatar
  - performance
  - optimization
  - ndmf
  - unity
aliases:
  - "Avatar 优化工具链组合指南"
  - avatar-optimization-stack.md
related:
  - "[[entities/avatar/avatar-optimizer.md]]"
  - "[[entities/avatar/meshia-mesh-simplification.md]]"
  - "[[entities/avatar/tex-trans-tool.md]]"
  - "[[entities/avatar/lac-avatar-compressor.md]]"
  - "[[entities/avatar/vrcfury-reference.md]]"
  - "[[entities/avatar/ma2bt.md]]"
  - "[[entities/avatar/thry-avatar-evaluator-metrics.md]]"
  - "[[entities/avatar/ndmf-tools.md]]"
  - "[[entities/avatar/optimization-guide.md]]"
  - "[[entities/avatar/performance-rank.md]]"
type: entity
created: 2026-10-24
sources: "https://github.com/anatawa12/AvatarOptimizer | https://github.com/RamType0/Meshia.MeshSimplification | https://github.com/ReinaS-64892/TexTransTool | https://github.com/limitex/avatar-compressor | https://github.com/VRCFury/VRCFury | https://github.com/Null-K/MA2BT | https://github.com/zhuozhi233/MA2BT-Pro | https://github.com/Thryrallo/VRC-Avatar-Performance-Tools"
updated: 2026-10-24
---

# Avatar 优化工具链组合指南（7 工具职责边界与共存风险）

> **本页定位**：横向把"阿凡达性能优化"常用的 7 个工具放进同一张职责表，回答三个既有单页答不了的问题——
> ① 每个工具**到底压哪个维度**（面数 / 材质槽 / 贴图 / 参数 / 动画层）；
> ② 它们**能不能全装**、谁和谁真冲突；
> ③ 出问题时**先撤谁**。
>
> **这不是工具教程**。每个工具的组件细节、参数、API 请走对应专页（见文末索引）。
> 本页只沉淀 `meta/network-search-protocol.md` §6 允许的两类内容：**兼容性信息**与**安装 / 使用边界**。

---

## 1. 来源声明与证据等级

本页内容是**基于项目自述（README / package.json / 源码结构）的工程经验整理**，按 `meta/network-search-protocol.md` 属 **L3 `[PROJECT-SOURCE]`**：

- **只证明"每个工具自身声明了什么行为"**，不构成 VRChat 平台事实；
- **不把"项目自身支持"误写成"VRChat 平台支持"**；
- 所有"哪个更快 / 哪个更好"类结论均为**经验性排序**，标 `[INFERENCE]`，需以自己工程的实测日志为准。

> ⚠️ 本页**不收录**任何"某工具设计有问题"式的主观评价；只收录可复现的现象、官方声明与命令级证据。

---

## 2. 7 工具职责矩阵（压什么 / 不压什么）

| 工具 | 主要压制维度 | 具体手段 | 明确**不**负责 | 是否 NDMF |
|------|--------------|----------|----------------|-----------|
| **AAO** (AvatarOptimizer) | 网格 / 骨骼 / PhysBone / 材质槽 | `Trace And Optimize` 自动追踪；`Merge Skinned Mesh`；`Merge Bone`；`Merge PhysBone`；`Freeze BlendShape`；`Remove Mesh By *` | 贴图压缩（Texture Memory） | ✅ 是 |
| **Meshia** (Mesh Simplification) | 面数（多边形） | `MeshiaMeshSimplifier`（单 Mesh 控制）；`MeshiaCascadingAvatarMeshSimplifier`（按目标 Performance Rank 级联分配） | 材质槽 / 贴图 / 动画 / 骨骼 | ✅ 是 |
| **TexTransTool** (TTT) | 材质球数量 / 贴图 VRAM | `AtlasTexture` 图集化；`AllMaterialMergeReference` | **材质槽（Material Slot）**、网格合并、PhysBone | ✅ 是 |
| **Avatar Compressor** (LAC) | 贴图 VRAM | 全自动纹理压缩（复杂度感知 / 平台自动 / 类型感知） | Mesh / Bone / PhysBone / Animator | ✅ 是 |
| **VRCFury** | 同步参数 / 动画层 / BlendShape VRAM / WD | `Parameter Compressor`（Float/Int → 16 bit）；`Direct Tree Optimizer`；`Blendshape Optimizer`；`Fix Write Defaults` | 网格 / 骨骼 / PhysBone / 贴图压缩 | ⚠️ 运行于 NDMF 之上但**自成体系**（见 §5） |
| **MA2BT / MA2BT-Pro** | Animator 层数（FX） | 把 `MA Responsive: <Name>` 层合并进单一 Direct BlendTree | Mesh / Bone / PhysBone / Texture | ✅ 是 |
| **Thry Performance Tools** | **不压任何维度** | 仅测量与报告（`Evaluator` / `VRAM` 右键菜单） | 一切修改性操作 | ❌ **否**（纯 Editor 工具） |

### 2.1 最容易误判的两条边界

**边界 A：TTT 压的是"材质球"，不是"材质槽"**

> `[PROJECT-SOURCE]` TTT 官方文档（`ttt.rs64.net`）明确：`AtlasTexture` 的材质结合**只合并材质**，**不能合并材质槽**，并建议与 `Anatawa12/AvatarOptimizer` 的 `TraceAndOptimize` / `MergeSkinnedMesh` 併用。
> 详见 [[entities/avatar/tex-trans-tool.md]]。

即：

- **降低材质球数量 / VRAM** → TTT 可以独立完成；
- **降低 Material Slot 数量（真正影响 DrawCall 的那一项）** → **必须**由 AAO 的 `Merge Skinned Mesh` 完成；
- 因此"TTT 必须结合 AAO 才能压材质槽"是**正确的**；单独装 TTT 无法降低材质槽。

**边界 B：Thry 不改变任何指标**

> `[PROJECT-SOURCE]` `de.thryrallo.vrc.avatar-performance-tools@1.3.7` 的 `package.json`：`vpmDependencies: {}`、`type: "tool"`；仓库仅含 `Editor/`（无 `Runtime/`），未引用 `nadena.dev.ndmf`。

它只负责**告诉你哪里超了**（VRAM / GrabPass / BlendShape / AnyState / Layer / WD / Empty States），**修复动作仍要落到上表其余工具**。
详见 [[entities/avatar/thry-avatar-evaluator-metrics.md]]。

---

## 3. 冲突与张力清单（只列有据可查的）

| # | 组合 | 现象 / 冲突 | 依据 | 严重度 |
|---|------|-------------|------|--------|
| 1 | **VRCFury × MA** | 两者互不知道对方做了什么，层合并与 WD 处理容易互相踩 | [[entities/avatar/ndmf-tools.md]]（执行顺序：`MA + 新增内容工具 → Fury → 其他最佳化工具`） | ⚠️ 中 |
| 2 | **VRCFury `Parameter Compressor` × MA `Sync Parameter Sequence`** | **硬冲突**：两者都重排参数槽位 | [[entities/avatar/vrcfury-reference.md]] §9.4 | 🔴 高 |
| 3 | **VRCFury `Direct Tree Optimizer` × MA2BT** | 两者都做"合并动画层"，会重复处理同一批层 | [[entities/avatar/vrcfury-reference.md]] §对比矩阵 | ⚠️ 中 |
| 4 | **AAO `Animator Optimizer` × MA2BT** | AAO 会进一步合并 MA2BT 产出的 `MA_To_BlendTree_Layer`；难以单独评估 MA2BT 效果 | [[entities/avatar/ma2bt.md]] §9.2 | ⚠️ 中（可测性问题） |
| 5 | **TTT × AAO** | **非冲突，反而是必需协作**：TTT 图集化 → AAO 合并 Mesh+材质槽 | [[entities/avatar/optimization-guide.md]] §与 AvatarOptimizer 协作 | ✅ 互补 |
| 6 | **LAC × AAO** | **非冲突**：LAC 在 AAO **之前**跑，让 AAO 基于已压缩贴图决策 | [[entities/avatar/lac-avatar-compressor.md]] §关键执行顺序 | ✅ 互补 |
| 7 | **Meshia × AAO** | **非冲突**：AAO 先去遮挡/合并，Meshia 再减面效果更好 | [[entities/avatar/meshia-mesh-simplification.md]] §与 AAO 互补 | ✅ 互补 |
| 8 | **AAO `Trace And Optimize` × OSC 开关 / 动画开关场景** | 仅靠 OSC 读取、未在 Animator/Expression 中定义引用的参数，可能被判定"未使用"并移除对应 PB/Contact | [[entities/avatar/avatar-optimizer.md]] §FAQ / §8 | 🔴 高（静默失效） |

> 💡 表中"互补"三项说明：**不是所有同装都冲突**。真正需要警惕的是 #1–#4 与 #8。

---

## 4. 推荐启用顺序（构建管线视角）

三条已核实的顺序建议，来源各自独立：

```
① NDMF 官方/生态顺序（混合优化工具的统一基线）
   MA → MA2BT → LAC → AAO → TTT → Meshia
   来源: [[entities/avatar/ndmf-tools.md]] §NDMF 内部执行顺序

② 含 VRCFury 时的顺序
   MA + 其他新增内容 NDMF 工具 → Fury → 所有 NDMF 最佳化工具
   来源: [[entities/avatar/ndmf-tools.md]] §MA 与 VRCFury 兼容性问题

③ TTT + AAO 的局部严格顺序
   TTT AtlasTexture → AAO MergeSkinnedMesh → AAO TraceAndOptimize
   来源: [[entities/avatar/optimization-guide.md]] §与 AvatarOptimizer 协作 (v0.9.0+)
```

**这三条并不互相矛盾**：③ 是 ① 中"TTT 与 AAO 相邻段"的细化；② 把 Fury 插在 MA 之后、最佳化工具之前。

---

## 5. 该不该一次全装——分批启用建议

> 结论先行：**不要一次性全加**。7 个工具同时引入会把"哪个改动导致外观/行为变化"变成不可归因问题。

### 5.1 建议分三批

| 批次 | 装什么 | 为什么这一批放一起 | 验证点 |
|------|--------|--------------------|--------|
| **第 1 批（核心，低风险）** | AAO + Thry 工具 | AAO 是基线优化，Thry 只测量；两者无修改性重叠 | Thry `Evaluator` 前后对比 + Build 后外观比对 |
| **第 2 批（贴图/面数）** | LAC、Meshia | 各自独立维度（贴图 / 面数），与第 1 批是互补关系（§3 #6 #7） | VRAM 数值 + 面数 + 破洞检查 |
| **第 3 批（动画/材质，高张力）** | TTT、MA2BT 或 MA2BT-Pro、**VRCFury 最后** | 这批才开始动 Animator 层与材质槽，冲突面最大 | Toggle 逐个点一遍 + WD 检查 + 材质槽数 |

> 若目标是 Quest 版本，先按 [[entities/platform/android-development.md]] 的原则**在副本工程里做**，不要直接改 PC 主工程。

### 5.2 VRCFury 放最后的理由

> `[PROJECT-SOURCE]` 社区与库内一致的执行顺序建议均把 Fury 排在"新增内容工具"之后、"最佳化工具"之前；
> 而 `meta/network-search-protocol.md` 要求区分**事实**与**经验**——"Fury 导致 MA 动画合并/WD 出问题"属 `[COMMUNITY]` 经验，**不是**已证实的平台缺陷。

可确证的**硬冲突**只有一条（§3 #2：`Parameter Compressor` × `Sync Parameter Sequence`）。
其余属"执行顺序没排对就会出错"，可通过顺序规避。

**因此更准确的表述是**：

- ❌ 不是"VRCFury 设计有问题"；
- ✅ 是"**VRCFury 与 MA 各自管理参数槽位与动画层，二者对同一批资源的处理顺序必须显式排序**"。

---

## 6. 出问题时的排查顺序（撤换法）

当出现"外观变化 / Toggle 失效 / 表情异常 / WD 报错"时，按**引入顺序倒推**：

```
① 先撤最后一批（通常是 VRCFury）        ← 冲突面最大、最易定位
② 再撤 MA2BT / MA2BT-Pro                ← 它改的是 Animator 层结构
③ 再撤 TTT                              ← 它改材质与 UV
④ 保留 AAO + Thry 作为最小复现基线      ← 两者只做优化/测量，最稳定
```

**为什么先撤 VRCFury 而不是先撤 AAO**：

1. VRCFury 与 MA 存在已记录的顺序张力（§3 #1）与一条硬冲突（§3 #2）；
2. AAO 默认设置的设计保证是"**绝不改变外观或行为**"（[[entities/avatar/avatar-optimizer.md]] §4.2 设计保证），若外观变了应视为 AAO 的 **Bug** 而非配置问题；
3. 保留 AAO + Thry 时，你仍有一份**可测量的基线**用于对比。

> 💡 若撤到只剩 AAO + Thry 仍异常，问题大概率不在"工具叠加"，而在 Avatar 自身资源（缺 Mesh、缺 Motion、WD 混用等）。此时按 [[entities/avatar/avatar-audit-methodology.md]] 走资源级排查。

---

## 7. 各维度达标判据（测什么、用什么）

| 维度 | 达标判据来源 | 测量工具 | 修复工具 |
|------|--------------|----------|----------|
| 面数 / 多边形 | [[entities/avatar/performance-rank.md]] | VRChat SDK Build & Test、Thry | Meshia、AAO `Remove Mesh By *` |
| 材质槽 / DrawCall | Performance Rank | SDK Build & Test | **AAO `Merge Skinned Mesh`** |
| 贴图 VRAM | [[entities/avatar/thry-avatar-evaluator-metrics.md]] §指标速查 | Thry `VRAM` | LAC、TTT、AAO `Max Texture Size` |
| 同步参数位宽 | 官方参数预算 + VRCFury 实测 | VRCFury Parameter Compressor 面板 | VRCFury |
| Animator 层数 | Thry `Layer Count` 五级阈值 | Thry `Evaluator` | MA2BT / VRCFury Direct Tree、AAO Animator Optimizer |
| BlendShape 开销 | Thry `Blendshape 影响三角形` | Thry | AAO `Freeze BlendShape`、VRCFury `Blendshape Optimizer` |
| PhysBone 开销 | lilAvatarUtils（见 [[entities/avatar/ndmf-tools.md]]） | lilAvatarUtils | AAO `Merge PhysBone` |
| Write Defaults 一致性 | Thry `Write Defaults` | Thry | VRCFury `Fix Write Defaults`、手动统一 |

---

## 8. 常见误区（按证据纠正）

| 误区 | 纠正 | 依据 |
|------|------|------|
| "这 7 个都是 NDMF 组件，可以一起装" | **错**：Thry 不是 NDMF 插件；VRCFury 自成体系 | 本页 §2；[[entities/avatar/thry-avatar-evaluator-metrics.md]] |
| "装 TTT 就能压材质槽" | **错**：TTT 只合并材质球，材质槽需 AAO `Merge Skinned Mesh` | [[entities/avatar/tex-trans-tool.md]] §能力边界 |
| "VRCFury 有设计缺陷，所以要避开" | **不准确**：可确证的硬冲突只有 `Parameter Compressor` × `Sync Parameter Sequence`；其余靠顺序规避 | [[entities/avatar/vrcfury-reference.md]] §9.4 |
| "AAO 优化后外观变了，是正常取舍" | **错**：AAO 设计保证"绝不改变外观"，变了应报 Bug | [[entities/avatar/avatar-optimizer.md]] §4.2 |
| "MA2BT-Pro 和 MA2BT 是同一个包" | **错**：Pro 是 fork，包名 `com.zhuozhi.ma2bt-pro`，VPM 源也不同 | [[entities/avatar/ma2bt.md]] §分叉对比 |
| "数量最多=性能最差，只看总数就行" | **不准确**：PhysBone 成本 = `Bones × Colliders` 乘积；材质槽与材质球不是同一指标 | [[entities/avatar/ndmf-tools.md]] §lilAvatarUtils 指标体系 |

---

## 9. 关联文档

- [[entities/avatar/avatar-optimizer.md]] — AAO 组件与 API 完整知识库
- [[entities/avatar/meshia-mesh-simplification.md]] — Meshia 算法、Options、与 lilNDMF/Mantis 对比
- [[entities/avatar/tex-trans-tool.md]] — TTT 15 组件、AtlasTexture 参数表、能力边界
- [[entities/avatar/lac-avatar-compressor.md]] — LAC 压缩策略与执行顺序
- [[entities/avatar/vrcfury-reference.md]] — VRCFury 全组件、参数压缩、冲突矩阵
- [[entities/avatar/ma2bt.md]] — MA2BT / MA2BT-Pro 层合并与分叉差异
- [[entities/avatar/thry-avatar-evaluator-metrics.md]] — 7 项指标阈值 + 工具性质边界
- [[entities/avatar/ndmf-tools.md]] — NDMF 生态总览与执行顺序
- [[entities/avatar/optimization-guide.md]] — 逐维度实操指南
- [[entities/avatar/performance-rank.md]] — 官方 PC / Quest 性能等级
- [[entities/avatar/avatar-audit-methodology.md]] — 资源级审查方法

---

## 相关页面

[[entities/avatar/index.md]] · [[entities/avatar/avatar-optimizer.md]] · [[entities/avatar/meshia-mesh-simplification.md]] · [[entities/avatar/tex-trans-tool.md]] · [[entities/avatar/lac-avatar-compressor.md]] · [[entities/avatar/vrcfury-reference.md]] · [[entities/avatar/ma2bt.md]] · [[entities/avatar/thry-avatar-evaluator-metrics.md]] · [[entities/avatar/ndmf-tools.md]] · [[entities/avatar/optimization-guide.md]]
