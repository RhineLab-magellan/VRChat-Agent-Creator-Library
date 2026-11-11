---
title: "Avatar 检查 / 审计方法论（Evidence & Authorization）"
category: avatar
knowledge_level: applied
status: active
source: 本地知识库整理 + VRChatEditorSkill playbook 本地化
source_type: community
version: 1.0
last_review: 2026-08-19
confidence: High
tags:
  - avatar
  - audit
  - evidence
  - authorization
  - toolchain
  - unity-mcp
aliases:
  - "Avatar 检查 / 审计方法论"
  - avatar-audit-methodology.md
related:
  - unity-mcp.md
  - playable-layers.md
  - expression-menu.md
  - modular-avatar.md
  - avatar-size-limits.md
  - blend-shape-troubleshooting.md
type: entity
created: 2026-08-19
sources: "本地知识库整理 + VRChatEditorSkill playbook 本地化"
updated: 2026-08-19
---
# Avatar 检查 / 审计方法论（Evidence & Authorization）

> 来源: 本地知识库整理 + VRChatEditorSkill playbook 本地化（核实日期 2026-08-19）
> 置信度: High（证据阶梯 / 授权边界为本库审计纪律；VRChat 官方数值见 §9 锚点）
> 定位: 本库 **Avatar 检查 / 诊断 / 对比 / 审计** 的唯一入口文档
> 适用范围: VRChat **Avatar** 项目（**不**覆盖 World / Udon）

---

## 1. 定位与适用

本文档是知识库的「检查 / 审计纪律」入口，适用于对 VRChat Avatar 项目做**检查、诊断、对比、审计**（不包括实现性修改）。核心主张一句话：

> **只声称你实际运行并验证过的那一层证据；低层不能证明高层，高层旧证据不能证明当前源码。**

适用边界：

| 适用 | 不适用 |
|------|--------|
| Avatar 的检查 / 诊断 / 对比 / 审计 | VRChat **World** / Udon / 世界场景 / 世界运行时 |
| 表达式、服装、道具、预览、优化相关检查 | 涉及上传/构建产物而不授权时的验证 |
| 菜单→参数→提供者→生成链路追踪 | 未经授权的任何写操作 |

衔接：本文档与 [[playable-layers.md]]（Animator 层级）、[[expression-menu.md]]（表情菜单链路）对接；Editor 级查询/变更须经 [[unity-mcp.md]] 的实例确认门（见 §8)。

> ⚠️ 若请求是修复/改动/上传，超出「检查 / 审计」只读语义，必须先单独授权——见 §6 授权矩阵。

---

## 2. 证据分层表（7 级）

按「能证明 / 不能证明」两列区分，使用**最窄**的、当前证据支持的那一层标签：

| 层 | 能证明 ✅ | 不能证明 ❌ |
|----|----------|------------|
| **1. `STATIC_SOURCE`** | 序列化文件、GUID/fileID 链、字面值、包元数据、声明的配置 | 导入语义、Prefab 实例解析、合并输出、运行时行为 |
| **2. `UNITY_RESOLVED`** | 经所选 Unity MCP 实例观察到的已导入资产、Prefab 实例、组件字段、active 状态、已解析层级 | NDMF 最终输出或 VRChat 运行时行为 |
| **3. `PROVIDER_PREVIEW`** | 工具源/预览态（provider 可见的 inventory 或 virtual/preview 状态） | 之后还能被后续 pass 变换的最终 SDK 产物 |
| **4. `NDMF_BUILT`** | NDMF/MA/FaceEmo/AAO 等生成后的合并 Avatar、菜单、参数、控制器、网格、组件 | 未同时运行过的 SDK 包体限制或客户端行为 |
| **5. `SDK_BUILD`** | VRChat SDK 校验 + 确切构建产物/参数结果 | 桌面 / VR / 多人 / 上传行为 |
| **6. `CLIENT_RUNTIME`** | 具名客户端层的表现（Play Mode / Build & Test / 桌面 / VR / 多人——须点名是哪一层） | 其他客户端、硬件模式、网络、上传（除非单独测过） |
| **7. `UPLOAD_CONFIRMED`** | 用户**明确授权**的已上传 Avatar 及其观察结果 | 未观察到的客户端或后续编辑 |

核心纪律（两条）：
- **低层不能证明高层**：YAML/Provider 预览/旧构建缓存 ≠ 最终玩家行为；
- **高层旧证据不能证明当前源码**：来源变更后旧的高层结论作废。

> ⚠️ 未运行层一律写 `NOT_RUN` / `BUILD_REQUIRED` / `MCP_REQUIRED`，**绝不写 `PASS`**。

---

## 3. 工具链确认（TOOLCHAIN_PROFILE）

做完整基线审计前，先以只读方式确认工具链（包检测是只读证据；工具选择是用户决策，两者互不蕴含授权）。

**最小 schema（文本块）**：

```text
target_avatar: <exact hierarchy 或 prefab 路径>
unity_mcp: <server/toolset 与确切 instance，或 unavailable>
clothing: <选定工具或 manual>
expression: <选定工具或自定义 owner>
props: <选定 provider 规则>
preview: <工具或 none>
optimization: <工具或 none>
installed_evidence: <manifest/component/version 来源>
missing_tools: <已选定但未安装的工具>
shared_consumers: <已知根对象或 prefabs>
authorized_actions: <inspect/import/configure/preview/build/runtime/upload>
status: CONFIRMED | NEEDS_USER_CHOICE | MISSING_TOOL | MCP_REQUIRED | STALE
```

> ⚠️ **检测 ≠ 已安装 ≠ 被该 Avatar 使用**。一个包可能已安装但对所选 Avatar 未使用；不要仅凭包名/文件夹名推断 provider 归属。

**默认角色映射（推荐，仍以用户选择为准）**：

| 类别 | 默认推荐 | 保留替代 |
|------|----------|----------|
| 衣装 / 衣橱 | **lilycalInventory** | 服装声明了其他受支持 provider |
| 表情 | **FaceEmo** | Avatar 有刻意的自定义表情 owner |
| 道具 / 通用 feature | 资产声明的 provider，**否则 MA** | Prefab 已有受支持 provider 配置 |
| 预览 | **Gesture Manager**（仅按需） | 无需编辑器模拟 |
| 优化 | **Avatar Optimizer**（仅按需 / 超出硬上限时） | 无优化目标或硬上限问题 |

---

## 4. 审计工作流

### 4.1 玩家菜单优先（player-menu-first）

以**玩家菜单**为主特征目录，从菜单追溯到源文件：

```
menu path → control/value → parameter → owner/provider →
controller/driver/生成源 → clip/binding → 确切 model 绑定 → 源文件 & GUID/fileID
```

被动系统**不虚构菜单项**：PhysBones / 自动 Contact / 内部生成参数不需要玩家菜单条目，除非它们暴露了玩家控制。

### 4.2 菜单所有权纪律

- 保留 provider 菜单所有权：一个插件的完整菜单视为一个所有权单元；**不跨插件合并叶子**来凑页容量；
- 若保留 provider 边界导致硬溢出，接受清晰的翻页/子菜单，而非跨 provider 合并；
- **接近上限 ≠ 问题**：只要未超硬上限，不主动提示优化。

### 4.3 尺寸与优化

> 对接近硬平台上限的值视为正常范围内的结果，不视作缺陷；仅当超限、当前功能无法在不超限下添加、或用户明确要求优化时才提优化。

```text
WITHIN_LIMIT | AT_LIMIT | OVER_LIMIT | NOT_MEASURED | BUILD_REQUIRED | AMBIGUOUS_TARGET
```

### 4.4 参数 / 驱动路由

最小读链：

```text
参数输入 → Expression Parameters 声明（type/default/synced/saved，见 [[expression-menu.md]]）→
controller/layer/state/driver → clip 绑定 → 具名模型效果 → 源文件 & GUID/fileID
```

要点与停止条件：
- **未在 Expression Parameters 中注册**的参数不占同步预算、远端不可见（本地 Animator 仍可用）——不要把它误判为「已同步」；
- 参数冲突时做**双向索引**：`参数 → 消费者` 与 `功能 → 参数` 都查；同一个 synced 位被多个 provider 占用时，先升级到 `PROVIDER_PREVIEW`/`NDMF_BUILT` 看 dedupe/重映射结果，仍无法唯一归主则标 `AMBIGUOUS_TARGET` 并停止，不猜 owner；
- **升级时机**：参数**成本**（位预算）只有 `NDMF_BUILT`/`SDK_BUILD` 层可证；「远端正同步/不同步」须 `CLIENT_RUNTIME`（具名多人层），单机状态机不能证明。

### 4.5 衣装 / Provider 迁移路由

最小读链：

```text
衣装源/wearable → provider 组件（[[lilycal-inventory.md]] / [[dressing-tools.md]] / [[modular-avatar.md]]）→
菜单与参数（默认/保存/同步/remap）→ Animator/骨架/渲染器/BlendShape 责任 → 生成输出（NDMF_BUILT）→ 具名运行时
```

要点与停止条件：
- **换 Provider 前先跑 §7.1 迁移责任清单**——每一份旧责任必须有新 owner，验证新 provider/NDMF 层后才移除旧组件；
- 每次装新 wearable 都要检查**同名 BlendShape 同步**（易配错，见 [[dressing-tools.md]]）；
- **inactive 对象在 provider 组件启用时仍可能参与构建**——不能把「源 inactive」当失败结论，要看构建结果；
- 骨架装配/合并通常是骨骼工具或手动工作，不属 LI/DT 自身职责（[INFERENCE]）；
- 停止：菜单源无法归一 owner、依赖生成菜单但未授权 preview/build、源链可定位但生成态未构建 → `BUILD_REQUIRED`/`NOT_RUN`。

### 4.6 PhysBone / Contact 路由

最小读链：

```text
PhysBone Root/组件 或 Contact Sender/Receiver → 允许的参数（类型/默认）→ 碰撞器/过滤器/tag → 运行时消费者
```

要点与停止条件：
- 被动系统不虚构菜单项；`PhysBone/Contact 参数`可能被 OSC 读取——**未在 Animator/Expression 定义也可能 active**，`[[avatar-optimizer.md]]` 等自动清理须复核这类参数；
- 指标锚点：Contact 上限（Avatar 口径）**256 组件**（官方硬上限；本文仅适用 Avatar，见 §1）。注意区分：**世界**运行时上限为 1024（见 [[contact.md]]），不可混用。PhysBone 相关指标引用 [[performance-rank.md]]/[[avatar-dynamic-bone-limits.md]]；
- **升级时机**：「物理运动/可交互」须 `CLIENT_RUNTIME`（具名层）；「参数是否同步」须生成层 + 具名多人层。

### 4.7 材质 / 纹理 / 网格 / BlendShape 路由

最小读链：

```text
渲染器路径 + 材质槽 → 材质（shader 属性）→ 纹理/导入器 或 动画绑定 → 共享消费者与平台覆盖
```

要点与停止条件：
- **导入设置或包含视觉资产改变后**：旧的体积/构建证据自动作废，标 `STALE`；不得在无确切构建下推断最终 bundle 变化；
- **共享材质/贴图/网格**改动 = 多消费者改动，先映射全部消费方再做（§7）；
- BlendShape 故障走 [[blend-shape-troubleshooting.md]] 的 4 步法；对照表参考 [[expression-menu.md]]/[[playable-layers.md]]；
- **升级时机**：「材质在玩家客户端的样子」须 `CLIENT_RUNTIME`；「尺寸影响」须 `SDK_BUILD`。

### 4.8 体积 / 上传就绪路由

- **压缩下载大小 与 未压缩 AssetBundle 大小都要来自当前的确切构建**（`SDK_BUILD`），不能用场景/FBX/纹理/项目目录尺寸或相似 Avatar 的结果转借；
- **Build & Test 不强制上传限制，也不等于「上传通过」**——上传是单独授权层（§6）；
- 平台未知 / Pipeline·cache 命中多个 Avatar / 构建过旧或源指纹过期 → `AMBIGUOUS_TARGET`/`STALE`/`NOT_MEASURED`，不渲染体积结论；
- 官方锚点 PC/Android 数值见 §9；Quest 实操目标 5–8MB、Crunch 只减下载不减内存（见 [[avatar-size-limits.md]] 与 §9）。

### 4.9 路由升级决策表（问题类型 → 最低证据层）

| 问题类型 | 最低证据层 | 理由 |
|----------|-----------|------|
| 仅序列化/字面值/包元数据 | `STATIC_SOURCE` | 无 Editor 依赖 |
| 菜单/参数/组件的**实例**配置 | `UNITY_RESOLVED` | 需经 Unity MCP 解析实例 |
| 「生成后」的菜单/参数/FX 是否包含 | `PROVIDER_PREVIEW` 或 `NDMF_BUILT` | 依赖 provider 生成态 |
| 参数成本（位预算/synced） | `NDMF_BUILT` 或 `SDK_BUILD` | 取决于合并/dedupe 后的最终结果 |
| 体积 < 上限 | `SDK_BUILD`（当前确切构建） | 打包后产物才可靠 |
| 远端同步/多玩家行为 | `CLIENT_RUNTIME`（具名多人层） | 运行时网络证据 |
| 已上传产物的表现 | `UPLOAD_CONFIRMED` | 单独授权层 |

> 通用决策：**沿读链走到第一处「当前证据层无法证明」的分叉点就停下来**，标 `BUILD_REQUIRED`/`NOT_RUN`/`MCP_REQUIRED`，不越层补猜；需要 Editor 状态时先过 §8 的 Unity MCP 实例确认门。

---

## 5. 状态语言表

| 状态 | 含义 |
|------|------|
| `PASS` | 仅**已运行且达标**的层可用；未运行的层不得写 PASS |
| `NOT_RUN` | 该层未尝试执行 |
| `BUILD_REQUIRED` | 源证据无法回答生成态问题 |
| `MCP_REQUIRED` | 结论/动作需要 Unity Editor 状态，但无可用确切 Unity MCP 实例 |
| `BLOCKED` | 尝试过的层无法产出可靠证据 |
| `STALE` | 因刷新触发点而过期的旧证据 |
| `AMBIGUOUS_TARGET` | 构建/Cache/Pipeline Manager 标识/重复对象无法对应到**恰好一个** Avatar |
| `SOURCE_ONLY` | 仅由静态源证据支撑，未达更高层 |

**典型误用**（应避免）：

| 错误说法 | 原因 |
|----------|------|
| 「works in VRChat」来自 YAML/Animator clip | 那是 `STATIC_SOURCE`，不是运行时 |
| 「under size limit」来自源文件尺寸 | 束体限制须用**确切当前构建**，不能从场景/FBX/纹理/相似 Avatar 推断 |
| 「所有相似 Avatar 参数成本相同」基于层级相似性 | 每个目标 Avatar 独立计算 |
| 「Upload passed」仅跑了 Build & Test 或本地预览 | 上传是单独授权层 |

---

## 6. 授权边界矩阵

| 请求措辞 | 通常授权 | 未自动授权 |
|----------|----------|------------|
| 检查 / 解释 / 审计 / 对比 / 诊断 | 读文件、查 live 状态、非变更诊断 | 保存/Apply、导入/刷新、Play Mode、构建、上传 |
| 修复 / 修改 / 移除 / 迁移 | **窄来源编辑** + 受影响文档更新 + 成比例的导入/编译校验 | Build & Test、新尺寸 probe/构建、发布/上传 |
| 预览 / 测试 | 具名预览/运行时层及其可逆设置 | 上传、无关项目清理 |
| 构建 | **具名目标**的确切构建 | 上传；破坏性源改动；假设 Build & Test 约束上传限制 |
| 上传 / 发布 | 预检后**确切确认的 Avatar 与平台** | 凭猜测选目标或上传其他 active 描述符 |

> ⚠️ 授权边界核心：**修复 = 窄来源编辑 + 受影响文档更新**；**预览/测试/Build = 具名层**；**上传 = 单独授权**——构建许可 ≠ 上传许可。可覆盖未保存场景工作、会修改共享源、或影响超出所命目标的多消费者时，停下来补决策。

---

## 7. 共享与生成物所有权（一页小结）

| 类别 | 处理方式 |
|------|----------|
| **源（可编辑声明）** | 可编辑的源配置，是授权修复的目标 |
| **生成物** | **timestamped 输出 / NDMF 结果 / 构建克隆 / 缓存**——只读或可重生成，**不手工改**（除非工具明确声明其为源） |
| **共享资产** | 改**共享**资产（prefab/material/menu/parameter/controller/anim/model）前，**先映射全部消费方**；可能影响多于所选场景对象 |

> 分层归因：scene-instance override / 共享资产 / 工具自有源配置 / 生成输出 / 构建克隆缓存——改动前先判定目标属于哪一层。

### 7.1 Provider 迁移责任清单（勾选表）

换 Provider、把衣装/功能迁到另一工具、或重做生成管线前，逐项确认**每一份旧职责都有了新 owner**：

| # | 责任项 | 待检查内容 |
|---|--------|-----------|
| 1 | **菜单入口** | 完整菜单路径、控件类型/值、图标、放置 owner |
| 2 | **参数** | 名称、类型、默认值、保存/同步意图、remap、最终消费者 |
| 3 | **Animator** | 层/状态/过渡归属、控制器 owner、Merge/生成来源 |
| 4 | **骨骼与 armature** | 映射/根、Bone Proxy 或骨骼重映射、约束、缩放 |
| 5 | **网格** | 渲染器、网格、材质槽、BlendShape + 跨衣装同名同步 |
| 6 | **对象开关** | 源 active 态、生成 toggle、平台过滤、默认态 |
| 7 | **PhysBone / Contact** | 根、碰撞器、过滤器/tag、参数、运行时依赖 |
| 8 | **材质 / 纹理 / 着色器** | 材质槽、shader 属性、纹理/导入器、平台覆盖 |
| 9 | **预置 / 默认态** | 包含项、互斥项、默认值、与单项开关的交互 |
| 10 | **生成结果** | 最终菜单/参数/Animator/层级、NDMF 所有权 |
| 11 | **非视觉效果** | 音频、粒子、事件、其他运行期消费者 |
| 12 | **旧组件处置** | 每份旧责任先有新 owner、验证新 provider/NDMF 层，**再**移除旧组件 |

> 迁移护栏：**不改生成物反推源**（重生成会抹掉）；旧职责未全部转移前不移除旧组件；迁移后按 §4.9 决策表重跑一遍最低证据层验证。

---

## 8. 与 Unity MCP 的关系

- 任何 **Editor 级查询/变更**（`UNITY_RESOLVED` 即第 2 层）都必须经 [[unity-mcp.md]] 的**实例确认门**：每次操作用前重新发现并证实 instance（项目路径、Unity 版本、active scene、dirty/compile/play 状态），不沿用旧连接、不以 recency 选实例；
- **无可用 Unity MCP 实例**时：只能声明 `STATIC_SOURCE`（第 1 层）；不得声称 `UNITY_RESOLVED`、不得以临时 Editor 脚本或 batch-mode Unity 静默替代；缺失层标 `MCP_REQUIRED` 或 `BLOCKED` 并说明未验证项；
- 干净 Console ≠ 通过；成功静态解析 ≠ Editor-resolved 证据（衔接 §2/§5）。

---

## 9. VRChat 官方锚点引用

> 以下数值为**官方来源 + 检索日期 2026-08-19**，引用时随文标注锚点 URL；未核对前不扩写。

| 主题 | 官方数值 | 官方 URL（核实日期 2026-08-19） |
|------|----------|----------------------------------|
| 参数同步 | 参数 **synced 256 bits**（同步位预算） | https://creators.vrchat.com/avatars/animator-parameters/ |
| 表达式参数 | 最多 **8192 expression params** | https://creators.vrchat.com/avatars/animator-parameters/ |
| PC 下载体积 | **200MB** 下载 / **500MB** 未压缩包体 | https://creators.vrchat.com/avatars/avatar-size-limits/ |
| Android 体积 | **10MB** 下载 / **40MB** 未压缩 | https://creators.vrchat.com/avatars/avatar-size-limits/ |
| Quest 实操目标 | Quest 实操目标 **5–8MB**；**Crunch 只减下载不减内存** | https://creators.vrchat.com/platforms/android/quest-content-optimization/ |

> ⚠️ 这些是「已被实测的层」才可引用的上限锚点；断 **体积/参数上限** 结论时须与你自己的第 5 层（确切构建）证据配套，见 §5 防误用。

---

## 10. 相关文档

- [[unity-mcp.md]] — Editor 级证据的授权通道与实例确认门（本文档的运行时配套）
- [[playable-layers.md]] — Animator 层级体系（审计链路中的层级解析）
- [[expression-menu.md]] — 表情菜单与参数链路（player-menu-first 追踪目标）
- [[modular-avatar.md]] — 生成层输出（NDMF_BUILT 证据来源）
- [[avatar-size-limits.md]] — 体积指标边界（SDK 构建层证据锚点）
- [[blend-shape-troubleshooting.md]] — Blend Shape 系统性排障（审计常见具体场景，见 §4.7 路由）
- [[lilycal-inventory.md]] / [[dressing-tools.md]] — 衣装/衣橱工具（§4.5 路由、§7.1 迁移清单）
- [[faceemo.md]] — 表情生成工具（表达式链路）
- [[physbones.md]] / [[contact.md]] — 被动系统锚点（§4.6 路由）
- [[performance-rank.md]] / [[avatar-dynamic-bone-limits.md]] / [[avatar-size-limits.md]] — 指标与体积锚点（§4.8、§9）

> 💡 本文档是审计纪律入口；各证据层对应的工具/数值条目请跳转到上表对应条目进一步细读。
