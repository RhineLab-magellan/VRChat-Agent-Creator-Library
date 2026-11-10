---
title: "NDMF 工具生态体系"
category: avatar
knowledge_level: applied
status: active
source: 本地知识库整理 + VRCFury 官网 + MA GitHub (工具选型决策树, 2026-07-04) + lilAvatarUtils 官方文档(2026-07-04)
source_type: community
version: 1.3
last_review: 2026-08-13
confidence: Medium
tags:
  - avatar
  - physbone
  - json
  - modular-avatar
  - vrcfury
  - decision-tree
  - lilavatarutils
aliases:
  - "NDMF 工具生态体系"
  - ndmf-tools.md
related:
  - performance-rank.md
  - optimization-guide.md
  - ma-component-cards.md
  - ma2bt.md
  - modular-avatar.md
  - vrcfury-reference.md
type: entity
created: 2026-07-04
sources: 本地知识库整理 + VRCFury 官网 + MA GitHub (工具选型决策树, 2026-07-04) + lilAvatarUtils 官方文档(2026-07-04) + NDMF GitHub release 1.14.0 (2026-06-29)
updated: 2026-08-13
---
# NDMF 工具生态体系

> 来源: Kuriko Avatar 最佳化笔记
> 置信度: High

---

## 什么是 NDMF

**NDMF (Non-Destructive Modular Framework)** 是一种非破坏性最佳化框架。

| 特性 | 说明 |
|------|------|
| **非破坏性** | 原始文件完整保留，最佳化仅在编译时生效 |
| **组件化** | 每个优化步骤是独立组件，便于组合 |
| **MA 依赖** | 大部分工具依赖 Modular Avatar 运行 |
| **可叠加** | 可以多个工具组合使用（组合技）|

> 💡 NDMF 体系的工具，最佳化不会破坏原始档案，且支援用 Modular Avatar 穿的衣服

---

## 工具一览

### 核心依赖

| 工具 | VPM 链接 | 说明 |
|------|----------|------|
| **Modular Avatar (MA)** | `vcc://vpm/addRepo?url=https://vpm.nadena.dev/vpm.json` | 必装，NDMF 核心 |
| **Non-Destructive Modular Framework** | 随 MA 自动安装 | NDMF 基础框架 |

### 最佳化工具

| 工具 | VPM 链接 | 价格 | 主要功能 |
|------|----------|------|----------|
| **Avatar Compressor (LAC)** ⭐NEW 2026-06-17 | `https://vpm.limitex.dev/` | 免费 | **全自动纹理压缩**（复杂度感知 + 平台自动 + VRAM 预估 + 类型感知）。详见 `lac-avatar-compressor.md` |
| **AvatarOptimizer (AAO)** | `https://vpm.anatawa12.com/add-repo` | 免费 | 合并 Skinned Mesh/Bones/PhysBone、Remove Mesh、各类最佳化 |
| **lilAvatarUtils** | 随 AAO repo | 免费 | 性能检测（贴图/材质/动画/Skinned Mesh/PhysBone/光照等） |
| **TexTransTool** | `vcc://vpm/addRepo?url=https://vpm.rs64.net/vpm.json` | 免费 | 材质球 Atlas 化（合并贴图）— 详见 `tex-trans-tool.md` |
| **MA2BT** ⭐NEW 2026-06-17 | `https://null-k.github.io/vpm-listing/index` | 免费 | **MA 响应式层合并优化**：将 `MA Responsive:*` 层压缩为单一 Direct BlendTree，减少 FX 层数与空动画。详见 `ma2bt.md` |

### 减面工具

| 工具 | VPM 链接 | 价格 | 优点 | 缺点 | 状态 |
|------|----------|------|------|------|------|
| **Mantis LOD Editor** | Unity Asset Store | **$50** | 模型不易破洞 | 调整时卡顿 | ✅ 活跃 |
| **Mantis NDMF 化工具** | Booth 直接下载 | 免费 | NDMF 封装 | 依赖 Mantis | ✅ 活跃 |
| **Meshia Mesh Simplification** | `https://ramtype0.github.io/VpmRepository/index.json` | 免费 | Burst+Job 高速、BlendShape 完整保留、`PreserveBorderEdges` 防破洞 | 需要手动配置选项 | ✅ **活跃（推荐）** |
| **lilNDMFMeshSimplifier** | `https://lilxyzw.github.io/vpm-repos/redirect.html` | 免费 | 导入即见效 | 看不到面数，已废弃 | ❌ **已废弃**（README 明确指向 Meshia） |

> 📖 **Meshia 完整技术文档**: `meshia-mesh-simplification.md`（架构、API、Options、与 lilNDMF/Mantis 对比）

### 辅助工具

| 工具 | VPM 链接 | 说明 |
|------|----------|------|
| **ContinuousAvatarUploader** | 随 AAO repo | 批次上传 Avatar |
| **Upload-without-preCheck** ⭐NEW | `https://sayabeans.github.io/vpm` | **绕过 VRCSDK 预构建硬限制检查**（社区工具，VRCSDK 3.8.1-beta.1 兼容性警告）|
| **VRCQuestTools** | `https://kurotu.github.io/VRCQuestTools/` | Android/Quest Avatar 转换 + VQT Avatar Builder 绕过预检查 |

### 衣物专用工具（2026-07-04 新增）

#### Avatar Clothing Tool

| 字段 | 值 |
|------|-----|
| 名称 | Avatar Clothing Tool |
| 作者 | hrenact |
| Booth | https://hrenact.booth.pm/items/5954005 |
| 用途 | 合并衣物骨骼与模型骨骼，减少骨骼数量 |
| 价格 | 免费 |
| 类型 | Booth 工具 |

**核心价值**:
- Android 平台 Bones 限制（Excellent 75 / Good 90 / Medium 150 / Poor 150）
- 衣物通常有额外骨骼（裙子飘动、袖子等）
- 通过合并衣物骨骼到模型骨骼，**减少总骨骼数**

**与 AAO Merge Bone 的对比**:

| 工具 | 用途 | 适用 |
|------|------|------|
| **AAO Merge Bone** | 通用骨骼合并 | 各种骨骼 |
| **Avatar Clothing Tool** | 衣物骨骼专用 | 衣物 → 模型骨骼合并 |

**实践建议**:
- 复杂衣物推荐先用 Avatar Clothing Tool
- 简单骨骼合并可用 AAO Merge Bone
- 两者不冲突，可串联使用

**适用场景**:
- 制作 Android 版本时
- 衣物骨骼过多导致超 Bones 限制
- 需要快速减少骨骼数

---

## 工具选型决策树: MA vs VRCFury ⭐NEW 2026-07-04

> 来源: VRCFury 官网 (vrcfury.com) + Modular Avatar GitHub (bdunderscore/modular-avatar) + VRCD 文档库 16-vrchat-avatar-BzV
> 置信度: High (官方资料 + 第三方文档交叉验证)

### 官方定位对比

| 工具 | 官方定位 | 核心特性 | 适合人群 |
|------|---------|---------|---------|
| **VRCFury** | "Non-Destructive Tools for VRChat Avatars" | GUI 简化配置 / 自动 WD 处理 / "asset artists" 友好 | 想要**少写代码、自动化生成**的创作者 |
| **Modular Avatar** | "Modular Avatar is a suite of non-destructive tools for modularizing your avatars" | drag-and-drop 组件 / 模块化拆分 / 组件分发 | 想要**精细控制、模块化组装**的开发者 |

### 真正的差异

| 维度 | VRCFury | Modular Avatar |
|------|---------|----------------|
| **架构哲学** | 生成式 (生成完整 Animator Controller) | 模块化 (拆分现有 Animator) |
| **WD 处理** | 自动维护 default states | 需手动配置 `match avatar write defaults` |
| **配置方式** | GUI 集中配置 | 组件式分散配置 |
| **路径处理** | 自动重写(prefab 根相对 → avatar 绝对) | 自动修正 (Merge Animator) |
| **上手难度** | 低 (开箱即用) | 中 (需理解组件体系) |
| **性能影响** | 较小 (直接生成) | 较小 (合并) |

### 选型决策树

```
[需求是什么?]
   │
   ├─ 想要"少写代码,自动生成"
   │   → ✅ VRCFury
   │
   ├─ 想要"模块化拆分,组件化分发"
   │   → ✅ Modular Avatar
   │
   └─ 想要两者结合
       → 先 MA(模块化) → 后 VRCFury(自动化)
       → 或: 使用 VRCFury 的 CompatibleMode 与 MA 协同
```

### ⚠️ 原文措辞偏差说明

> 第三方文档(VRCD)原文:"VRCFury 面向新手玩家的一站式傻瓜化解决方案,MA 面向的是有一定 VRChat Avatar 开发知识的开发者"
>
> **实际情况**:
> - VRCFury 官方定位强调 **"asset artists"**(创作者)而非"新手玩家",措辞过强
> - MA 官方定位强调 **"drag-and-drop"**(易用性)而非"需要经验",与原文描述有偏差
>
> **结论**: 两类工具的**相对定位差异**(VRCFury 生成式 vs MA 模块化)**是准确的**,但对用户群体的描述需修正。
>
> 来源对比:
> - VRCFury 官网: https://vrcfury.com/  → "Easy to use / Great for asset artists / No more write defaults pain"
> - MA GitHub: https://github.com/bdunderscore/modular-avatar  → "drag-and-drop / modularizing / distributing components"

---

## MA 与 VRCFury 兼容性问题

> ⚠️ Fury 与 MA 互相不知道对方做了什么，容易出错

### 推荐执行顺序（来源：きくじん）

```
1. MA + 其他新增内容 NDMF 工具（Mantis、MeshSimplifier 等）
2. Fury
3. 所有 NDMF 最佳化工具（整合、删除系工具等）
```

### NDMF 内部执行顺序（2026 更新）

> ⚠️ 同一阶段内的多个工具仍需注意顺序

```
1. Modular Avatar（穿脱服装）
2. MA2BT                      ← MA 响应式层 → Direct BlendTree（必须在 MA 之后）
3. Avatar Compressor (LAC)    ← 自动纹理压缩（先处理贴图）
4. Avatar Optimizer (AAO)     ← 合并 Mesh/Bone/PhysBone（在压缩后的贴图上工作）
5. TexTransTool               ← Atlas 化（最后）
6. Meshia / 其他减面工具      ← 减面（按需）
```

**为什么 MA2BT 在 LAC 之前**（2026 新增）:
- MA2BT 强制 `seq.AfterPlugin("nadena.dev.modular-avatar")`，必须在 MA 之后
- MA2BT 处理的是 Animator 层结构（与贴图无关），顺序可与 LAC 互换
- 推荐放在靠前位置，让后续工具基于"已合并的 Animator 层"工作

**为什么 LAC 在 AAO 之前**（2026 新增）:
- AAO 的 Trace & Optimize 涉及 Skinned Mesh 合并、PhysBone 合并
- 这些操作基于"贴图状态"做决策
- 先压缩贴图可以让 AAO 看到"最优贴图状态"
- 详见 `lac-avatar-compressor.md` 与 `avatar-optimizer.md`

### 思路

完全理解 MA 和 Fury 各自做了什么，避免让他们处理同一个区块。

---

## VPM Package 导入清单

初次导入需要安装以下 Package（导入后在 Settings > Packages 启用）：

- AAO: Avatar Optimizer
- anatawa12's gists pack
- **Avatar Compressor (LAC)** ⭐NEW 2026-06-17
- Continuous Avatar Uploader
- lilAvatarUtils
- **MA2BT** ⭐NEW 2026-06-17
- Meshia Mesh Simplification
- Modular Avatar
- Non-Destructive Modular Framework
- TexTransTool

> ❌ `lilNDMFMeshSimplifier` 已废弃，不再列入导入清单

---

## lilAvatarUtils 性能指标体系 ⭐NEW 2026-07-04

> 来源:lilAvatarUtils 官方文档 https://lilxyzw.github.io/lilAvatarUtils/en_US/docs/
> 补充意义:补充 5 个 PhysBones 性能指标 + 6 种 Lighting 环境模式(原 ndmf-tools.md 仅记录开启位置)

### PhysBones 性能指标(5 个)

> 来源:`PhysBonesGUI.html` - "useful for identifying PhysBone components that have a large impact on performance rank"

| # | 字段 | 含义 | 性能影响 |
|---|------|------|---------|
| 1 | **Bones** | PhysBone 计算的骨骼数 | ⭐⭐⭐ **数量越多,成本越高**(核心指标) |
| 2 | **Colliders** | 碰撞检测的碰撞器数 | ⭐⭐ 影响碰撞计算频率 |
| 3 | **Collision** | 碰撞计算总数 | ⭐⭐⭐ = bones × colliders,**值越大负载越高** |
| 4 | **Multi Child Type** | 多子骨骼方向计算方式 | ⭐ 算法选择影响性能(Polar 限制消耗大) |
| 5 | **Parent** | 父对象(可整合依据) | 💡 同一父级可考虑合并 PB |

**官方使用建议**:
> "useful for identifying PhysBone components that have a large impact on performance rank, and for discovering PhysBones that have similar root bones and settings and can be integrated."

**实践要点**:
- **Bones 数量**:直接决定 PhysBone 成本,优先控制
- **Collision = Bones × Colliders**:乘积决定碰撞开销
- **Multi Child Type**:避免 Polar(性能消耗大)
- **Parent 整合**:同一父级且设定相近的 PB 可合并(降低组件数)

**与 AAO 协作**:
- lilAvatarUtils 用于**发现和诊断**高成本 PB
- AAO Merge PhysBone 用于**自动合并**相同设定的 PB
- 推荐工作流:`lilAvatarUtils 诊断 → AAO 合并 → lilAvatarUtils 验证`

### Lighting 6 种环境模式

> 来源:`LightingTestGUI.html` - "check how your avatar looks in various environments"

| # | 模式 | 用途 | 适用场景 |
|---|------|------|---------|
| 1 | **No light** | 完全没有光(含环境光) | 测试 Avatar 的自发光/Emission 效果 |
| 2 | **Overexposure** | 过亮方向光照射 | 测试高光、曝光溢出、色彩饱和度 |
| 3 | **In Shadow** | 整个 Avatar 处于阴影中 | 测试阴影下的表现,避免"黑脸"问题 |
| 4 | **Spot Light** | 单个聚光灯照射 | 测试聚光下细节、轮廓光 |
| 5 | **3 Spot Lights** | 三个聚光灯照射 | 测试复杂舞台光、演出光效 |
| 6 | **Custom** | 用户自定义光照 | 复现特定 World 光照环境 |

### Safety 模式预览(原 Safty 拼写错误纠正)⭐CORRECTED

> **拼写纠正**:VRCD 原文"Safty" 应为 **"Safety"**(官方标准拼写)

- **功能**:预览 **VRCSDK Safety 模式下未应用 Shader** 的 Avatar 表现
- **官方原文**:"When you install VRCSDK, you can also check how your avatar looks when shaders are not applied due to safety."
- **意义**:
  - VRChat 客户端有"低性能模式"会关闭部分 Shader
  - Safety 模式可让创作者**提前预览低性能模式下的 Avatar 表现**
  - 避免对 Shader 效果的过度依赖
- **典型场景**:
  - 测试 Avatar 在低端客户端(Quest)的视觉表现
  - 避免"特定 Shader 关掉就崩坏"的设计

### 与其他工具的协作

| 工作流 | 工具组合 | 用途 |
|--------|---------|------|
| **改模诊断** | lilAvatarUtils → AAO | 诊断高成本 PB/Material → 自动合并 |
| **贴图优化** | lilAvatarUtils Textures → LAC | 查看贴图设置 → 自动压缩 |
| **光照验证** | lilAvatarUtils Lighting | 6 种环境预览 → 最终发布前 QA |
| **清理工具** | lilAvatarUtils Utils | 删除未用属性/Missing Component |

---

## 辅助窗口

### Actual Performance Window

由于 NDMF 是非破坏型最佳化，VRChat SDK 只显示编译前状态（通常是 Very Poor）。

**启用方法**:
```
1. 功能列 Tools > anatawa12's gist selector
2. 勾选 ActualPerformanceWindow 后，点击 Apply Changes
3. 功能列 Tools > anatawa12's gist
4. 勾选 Compute actual Performance on Play
```

进入 Play-mode 即可查看编译后真实效能。

### AvatarUtils 窗口

性能检测工具，可查看 Avatar 各部位数值。

**启用方法**: `Tools > AvatarUtils`

将 Avatar 拖入窗口即可查看各参数值。

#### UI 操作三按钮(2026-07-04 来自 lilAvatarUtils 官方文档)

> 来源:https://lilxyzw.github.io/lilAvatarUtils/en_US/docs/AvatarUtils.html
> 补充意义:补充 UI 操作规范(原 ndmf-tools.md 仅记录开启位置)

**核心规则**:在工具中修改属性后,必须按 `Apply` 才会一次性应用所有更改。

| 按钮 | 作用 | 注意事项 |
|------|------|----------|
| **Apply** | 批量应用 UI 上所有修改 | ⚠️ **不按 Apply 修改不生效** |
| **刷新** | 刷新界面显示最新状态 | Apply 后按下查看新状态 |
| **Revert** | 放弃未应用的修改 | 后悔药 |

**搜索与筛选**:
- 搜索区输入值后,**超过阈值的项变红**
- 勾选 `Filters` 复选框 → **只显示红色(超标)项**,便于快速定位优化目标

**Total 总计**:
- 多数页面最底层显示 `Total`(该类资源的总规模,如贴图总大小、材质总数等)

**典型工作流**:
```
1. 拖入 Avatar 到 AvatarUtils 窗口
2. 在各 Tab 中查看资源(超阈值会变红)
3. 勾选 Filters 只看红色项
4. 修改属性(或用 Replace 批量替换)
5. 按 Apply 一次性应用
6. 退出前按刷新确认状态
```

---

## 2026 版本增量：NDMF 1.14.0

- 新增 `VirtualAnimatorController.SetParameter`，为构建期虚拟 Animator 参数设置提供明确 API。
- 修复 Unity 6000.x 兼容性问题。
- 修复 `PropCache` 的 compute 或 refresh operator 抛出异常后可能永久 stale 的问题。
- Avatar root 设置为 `EditorOnly` 时增加显式错误提示。

来源：[FACT] [NDMF 1.14.0](https://github.com/bdunderscore/ndmf/releases/tag/1.14.0)

> **兼容性说明**：NDMF 版本变化可能影响多个 Avatar 构建插件的执行顺序和预览结果；升级后应重新执行一次 Avatar Build，并检查 MA、AAO、TexTransTool 等插件的构建日志。

## 相关文档

- `performance-rank.md` — Performance Rank 标准
- `optimization-guide.md` — 完整最佳化实操指南
- `self-mmd.md` — 自身跳 MMD 工作流
- `world-fixed-objects.md` — 固定世界物品结构

---

## VRC 工具箱(2026-07-04 来自 yexcadocs)

> 来源:yexcadocs §固定世界 MMD(P2)
> 本节为社区实操经验补充。

### 工具信息

| 字段 | 值 |
|------|-----|
| **名称** | 如梦 VRC 工具箱 |
| **版本** | v1.2.5 |
| **来源** | https://gitee.com/cmoyuer/vrchat-avatar-toolkit/releases |
| **作者** | 如梦Nya(B 站) |
| **状态** | 较旧,新版可能步骤略变 |

### MMD 转换特性

- **可不带 MMD 模型**转换动作
  - 但有些动作会变怪
  - **建议带上 MMD 模型**(防止动作失真)
- 转换后**复制原模型**再拖入动画
  - 在层级中 `CTRL+D` 复制
  - 隐藏原模型
  - 将动画拖到新模型
  - 点击播放测试

### 完整工作流(自身跳 MMD)

```
1. 打开 VRC 工具箱
2. 顶部选择 "VRC工具箱 - MMD动作转换"
3. 拖入 MMD 动作文件(vmd)
4. 拖入 MMD 模型(pmx)
5. 点击 "生成配置文件"
6. 调整动画方向: 跟变换旋转 → 原始
7. 在 Avatar Animator 的 Action 层导入动画
8. 配置 Locomotion Control 配对
9. 在 FX 层配置音乐开关
```

详细完整流程见 `self-mmd.md`。

---

## 相关页面

[[performance-rank.md]] · [[optimization-guide.md]] · [[ma-component-cards.md]] · [[ma2bt.md]] · [[modular-avatar.md]] · [[vrcfury-reference.md]] · [lilToon 官方](https://lilxyzw.github.io/lilToon/) · [[lilycal-inventory.md]] · [[dressing-tools.md]] · [[avatar-audit-methodology.md]]
