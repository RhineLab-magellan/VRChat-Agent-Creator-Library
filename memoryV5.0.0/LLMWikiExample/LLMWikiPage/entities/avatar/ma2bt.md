---
title: "MA2BT (Modular Avatar to BlendTree)"
category: avatar
knowledge_level: applied
status: active
source: "github.com/Null-K/MA2BT"
source_type: community
version: 1.1
last_review: 2026-10-24
confidence: Medium
tags:
  - avatar
  - animator
  - physbone
  - modular-avatar
aliases:
  - "MA2BT (Modular Avatar to BlendTree)"
  - ma2bt.md
related:
  - ndmf-tools.md
  - modular-avatar.md
  - avatar-optimizer.md
  - optimization-guide.md
  - performance-rank.md
  - avatar-modding-guide.md
type: entity
created: 2026-06-20
sources: "github.com/Null-K/MA2BT | https://github.com/zhuozhi233/MA2BT-Pro"
updated: 2026-10-24
---
# MA2BT (Modular Avatar to BlendTree)

> 工具名: MA2BT | 来源: github.com/Null-K/MA2BT
> 置信度: High (官方仓库)
> 语言占比: C# 100.0%
> VPM 包名: `com.puddingkc.ma2bt`

---

## 1. 工具定位

**MA2BT** 是 NDMF 插件，专门将 Modular Avatar 生成的**响应式 Animator 层**（`MA Responsive: <Name>`）合并到一个**Direct BlendTree**，从而减少 FX 层数量并提升 Avatar 运行性能。

| 维度 | 说明 |
|------|------|
| **设计目标** | 把多个 MA 响应式层压缩为单一 Direct BlendTree，减少 Animator 层数和空动画 |
| **核心场景** | Avatar 开关多物件（Object Toggle、Material Setter、Shape Changer 等）的层合并优化 |
| **非目标** | Mesh / Bone / PhysBone / Texture 优化（这些归 AAO / LAC / Meshia） |
| **互补关系** | 与 AAO Trace & Optimize 互补使用；可与 AAO Animator Optimizer 串联工作 |
| **平台限制** | 仅作用于 **VRChat Avatar 3.0**（`RunsOnPlatforms(WellKnownPlatforms.VRChatAvatar30)`） |

> 💡 **设计哲学**：MA2BT 不修改 MA 原始层结构，而是在 NDMF 优化阶段后期将符合条件的层"重组"为 Direct BlendTree，**复用已存在的 `MA_To_BlendTree_Layer` 而非新建**，避免多次构建产生重复层。

---

## 2. 工作原理

Modular Avatar 会为每一个响应式属性（Object Toggle、Material Setter、Shape Changer 等）生成一个独立的 Animator 层，这些层在运行时带来额外开销。MA2BT 在构建完成后分析这些生成的层，并将其中**简单结构**转换为共享层中的 BlendTree 节点。

### 2.1 转换前后对比

**优化前**：
```
MA Responsive: Hat        (Layer)
MA Responsive: Glasses    (Layer)
MA Responsive: Jacket     (Layer)
MA Responsive: Shoes      (Layer)
...每个层独立运作
```

**优化后**：
```
MA_To_BlendTree_Layer     (1 Layer, 1 Direct BlendTree)
  ├── hat_param     → 1D BlendTree
  ├── glasses_param → 1D BlendTree
  ├── jacket_param  → 1D BlendTree
  └── shoes_param   → 1D BlendTree
...所有参数共用一个根直混层
```

### 2.2 执行阶段

```
1. Modular Avatar 生成 MA Responsive:* 层
   ↓
2. seq.AfterPlugin("nadena.dev.modular-avatar")  ← MA2BT 强制在 MA 之后
   ↓
3. BuildPhase.Optimizing 阶段
   ↓
4. AnalyzeAllLayers() → 过滤可转换层
   ↓
5. GroupByParameter() 按参数分组
   ↓
6. BuildRootBlendTree() 构建 Direct BlendTree
   ↓
7. InjectBlendTreeLayer() 注入新层（复用同名层）
   ↓
8. RemoveLayers() 删除已被转换的源层
   ↓
9. DestroyImmediate(MAToBlendTree 组件)
```

### 2.3 关键算法要素

| 算法点 | 实现细节 |
|--------|----------|
| **根直混参数** | 硬编码字符串常量 `zhz/1`（继承自原"浊鸷"插件命名） |
| **注入层名** | `MA_To_BlendTree_Layer`（已存在则复用而非新建） |
| **注入状态名** | `RootBlendTree`，`writeDefaults=true` |
| **层权重** | `DefaultWeight = 1`，`BlendingMode = Override` |
| **阈值算法（Compact）** | 按"实际有动画的阈值 + 间距中点"动态生成 1D BlendTree 阈值 |
| **阈值算法（Standard）** | 补充 `0..Max` 所有整数 + 现有值 |
| **副参数支持** | 仅接受单边 Greater/Less 阈值（简单布尔条件） |

---

## 3. 系统要求

| 项目 | 版本 |
|------|------|
| **Unity** | 2022.3 及以上（VRChat 指定 2022.3.22f1） |
| **VRChat Avatars SDK** | 3.x |
| **Modular Avatar** | ≥ 1.10.0 |
| **NDMF** | ≥ 1.4.0（随 MA 自动安装） |
| **运行平台** | 仅 VRChat Avatar 3.0 |

> ⚠️ package.json 中 `unity` 字段声明为 `2019.4`，但 README 要求 `2022.3+`，**两者不一致，以 README 为准**。

---

## 4. 安装方式

### 方式 1: VCC / ALCOM（推荐）

```
VPM 仓库链接: https://null-k.github.io/vpm-listing/index

1. 打开 VCC / ALCOM
2. 添加仓库链接
3. 在项目中安装 MA2BT
4. 确保已安装 Modular Avatar（会带入 NDMF）
```

### 方式 2: GitHub Release 手动安装

```
1. 访问 https://github.com/Null-K/MA2BT/releases
2. 下载最新 .zip 包（v2.0.0+ 为 VPM 格式）
3. 通过 VPM CLI 或 ALCOM 导入
```

### 方式 3: 旧版 .unitypackage（不推荐）

> v1.x 提供 `MA2BT.unitypackage`，v2.x 已切换为 VPM 格式。**建议直接使用 v2.x**。

---

## 5. 使用方法

### 5.1 基本流程

```
1. 选中你的 Avatar 根节点（Avatar root，含 VRC Avatar Descriptor）
2. Add Component > MA2BT > MA2BT
3. 配置组件选项（Compact Mode / Multi-State Layers / Scan All Layers）
4. 正常构建 Avatar（MA2BT 会在优化阶段自动运行）
5. 在 Console 中查看 [MA2BT] 日志了解转换结果
```

### 5.2 Inspector 界面

组件位于 `Add Component > MA2BT > MA2BT`，包含：

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `compactMode` | bool | true | 仅在实际存在动画的数值上生成 BlendTree 阈值 |
| `convertMultiState` | bool | false | 尝试转换包含多个条件状态的层（默认关闭以保证安全） |
| `scanAllLayers` | bool | false | 不仅扫描 MA 生成的层，也会扫描所有符合模式的 FX 层 |

自定义 Inspector 使用紫色主题（`Color(0.55, 0.20, 0.85)`），版本号 `MA2BT v2.0.2` 硬编码。

### 5.3 日志格式

```
[MA2BT] No convertible MA Responsive layers found, skipping.
[MA2BT] Keeping layer "<layer>": <reject reason>
[MA2BT] Found N convertible layers (including K non-MA layers), M kept layers.
[MA2BT] Done: merged N layers into K BlendTree nodes.
[MA2BT] Parameter "<name>": L layers > T thresholds [v1, v2, ...]
```

---

## 6. 选项详解

### 6.1 Compact Mode（默认开启 ✅）

**作用**：仅在实际存在动画的数值上生成 BlendTree 阈值，减少空动画。

| 模式 | 阈值生成策略 | 适用场景 |
|------|--------------|----------|
| **Compact（开启）** | 按"实际有动画的阈值 + 间距中点"动态生成 | 大多数情况（推荐） |
| **Standard（关闭）** | 补充 `0..Max` 所有整数 + 现有值 | 需要枚举所有整数值的特殊场景 |

> 💡 **建议**：保持默认开启。Standard 模式会产生大量空动画 Clip。

### 6.2 Multi-State Layers（默认关闭 ❌）

**作用**：尝试转换包含多个条件状态的层（如多值 int 参数）。

| 模式 | 安全性 | 适用场景 |
|------|--------|----------|
| **关闭（默认）** | ✅ 高 — 仅支持 2 状态（Default + 1 Conditional） | 大多数情况（推荐） |
| **开启** | ⚠️ 中 — 支持多状态转换 | 已知 MA 层结构简单且需要多值参数 |

> ⚠️ **建议**：除非确认所有层都是简单结构，否则保持关闭。

### 6.3 Scan All Layers（默认关闭 ❌）

**作用**：不仅扫描 MA 生成的层，也会扫描所有符合模式的 FX 层。

| 模式 | 扫描范围 | 适用场景 |
|------|----------|----------|
| **关闭（默认）** | 仅 `MA Responsive:` 前缀的层 | 标准 MA 用户（推荐） |
| **开启** | 所有符合模式的 FX 层 | 混合 MA + 手写层的高级用户 |

> ⚠️ **风险**：开启后可能转换非 MA 生成的层，需自行确保层结构兼容性。

---

## 7. 可转换条件

当且仅当满足**所有**以下条件时，层才会被转换：

| # | 条件 | 说明 |
|---|------|------|
| 1 | **层名前缀** | 名称以 `MA Responsive:` 开头（开启 `Scan All Layers` 时不限制） |
| 2 | **状态数** | 状态机包含 2 个状态（1 个 Default + 1 个 Conditional） |
| 3 | **Multi-State** | 或开启 `Multi-State Layers` 时可支持更多状态 |
| 4 | **Entry Transition** | 所有 Entry Transition 条件仅使用单一参数 |
| 5 | **瞬时过渡** | 所有 Transition 均为瞬时（`Duration = 0` 且无 Exit Time） |

### 7.1 支持的 Condition Mode

| Mode | 是否支持 | 说明 |
|------|----------|------|
| **Greater** | ✅ | 支持 |
| **Less** | ✅ | 支持 |
| **If** | ✅ | 支持（隐含 ≥ 0.5） |
| **IfNot** | ✅ | 支持（隐含 < 0.5） |
| **Equals** | ❌ | 不支持（直接 Fail） |
| **NotEqual** | ❌ | 不支持（直接 Fail） |

### 7.2 主参数选择优先级

```
1. 有限 Lo+Hi 且不以 "__ActiveSelfProxy" 开头的 → 按 (Hi-Lo) 降序
2. 否则有限 Lo+Hi 的 → 再按 "__ActiveSelfProxy" 排序
3. 再退化为至少有 Lo 或 Hi 有限的
```

### 7.3 副参数要求

副参数必须为**简单布尔条件**（仅有 Lo 或仅有 Hi），记录为 `SecondaryCondition`。

---

## 8. 跳过原因（完整枚举）

以下情况**不会**被转换，Console 会输出 `Keeping layer "<layer>": <reject reason>`。

### 8.1 层结构问题

| 原因 | 触发条件 | 解决建议 |
|------|----------|----------|
| `No state machine` | 该层没有状态机 | 检查层结构 |
| `Insufficient state count (N)` | 状态数 < 2 | 检查状态机设计 |
| `No default state` | 没有默认状态 | 设置 Default State |
| `Multi-state layer (N conditional states), enable convertMultiState` | 多状态层但未开启 Multi-State | 开启 `convertMultiState` 选项 |
| `No Entry Transition` | 没有 Entry Transition | 检查状态机转换 |
| `State "<name>" has no corresponding Entry Transition` | 状态没有对应的 Entry Transition | 检查状态机设计 |

### 8.2 条件参数问题

| 原因 | 触发条件 | 解决建议 |
|------|----------|----------|
| `Multiple parameters detected: "A" and "B"` | 检测到多参数 | 拆分层，每层单参数 |
| `In inverted mode, Entry Transition has N conditions (expected 1)` | 反相模式多条件 | 简化 Entry Transition |
| `Multiple parameters in inverted mode: ...` | 反相模式多参数 | 同上 |
| `Inverted multi-parameter AND conditions are not supported` | 反相多参数 AND 条件 | 同上 |
| `No conditions` | 没有条件 | 添加条件 |
| `Unsupported condition mode: Equals/NotEqual` | 使用了 Equals/NotEqual | 改用 Greater/Less/If/IfNot |
| `Invalid condition range for parameter "...": lo..hi` | 无效的阈值范围 | 检查阈值设置 |
| `Failed to identify main parameter` | 无法识别主参数 | 检查参数条件 |
| `Failed to extract parameter name` | 无法提取参数名 | 检查参数名是否合法 |
| `Secondary parameter "..." must be a simple boolean condition` | 副参数非简单布尔 | 简化副参数条件 |

### 8.3 转换问题

| 原因 | 触发条件 | 解决建议 |
|------|----------|----------|
| `Non-instant Transition` | 存在混合时间 | 设置 Duration = 0 |
| `Complex state machine structure` | 包含子状态机或 AnyState 过渡 | 简化状态机结构 |

---

## 9. 与其他优化器关系

### 9.1 推荐执行顺序

```
1. Modular Avatar（生成 MA Responsive 层）
   ↓
2. MA2BT（合并 MA Responsive 层 → Direct BlendTree）  ← 在 MA 之后
   ↓
3. AAO Trace And Optimize（合并 Skinned Mesh / Bone / PhysBone）
   ↓
4. AAO Animator Optimizer（可进一步合并层）
```

> 💡 MA2BT 与 AAO 是**互补而非竞争**关系：
> - **MA2BT** 处理 FX 层合并（响应式层 → Direct BlendTree）
> - **AAO** 处理 Skinned Mesh / Bone / PhysBone 合并
> - 两者可叠加工作，效果累加

### 9.2 测试建议

> ⚠️ 如果安装了 AAO 或其他会合并动画层的插件，生成的 `MA_To_BlendTree_Layer` 层会被这些插件**进一步合并**。可以先移除其他优化插件来测试合并的数量和效果。

### 9.3 与 VRCFury 共存

参考 `ndmf-tools.md` 中的执行顺序：

```
1. MA + 其他新增内容 NDMF 工具
2. Fury
3. 所有 NDMF 最佳化工具（包括 MA2BT 和 AAO）
```

---

## 10. 限制与已知问题

### 10.1 功能限制

| 限制 | 说明 |
|------|------|
| **平台限制** | 仅 VRChat Avatar 3.0（不支持 2.0 / 3.0 以外的平台） |
| **层类型限制** | 仅处理 FX 层（不影响 Base / Additive / Gesture / Action 层） |
| **参数类型限制** | 强制 Float（若同名参数存在但类型不是 Float，会保留默认值并强制转换） |
| **不支持 Equals/NotEqual** | 仅支持 Greater/Less/If/IfNot |
| **Multi-State 默认关闭** | 多状态层需手动开启 `convertMultiState` |

### 10.2 已知边界

| 边界情况 | 行为 |
|----------|------|
| `__ActiveSelfProxy` 前缀参数 | 作为次要优先级处理（避免错误识别为主参数） |
| 反相模式（双 Entry Transition） | 要求每个 Entry 仅 1 个条件、参数相同 |
| 多参数 AND 条件 | 不支持，跳过 |
| 子状态机 / AnyState 过渡 | 视为复杂结构，跳过 |

---

## 11. 反馈渠道

| 语言 | 渠道 |
|------|------|
| **简体中文** | QQ 群 `1047423396` |
| **English** | GitHub Issues: https://github.com/Null-K/MA2BT/issues |
| **日本語** | GitHub Issues: https://github.com/Null-K/MA2BT/issues |

---

## 12. Release 历史

| Tag | 发布时间 | 资产 | 大小 | 下载次数 |
|-----|----------|------|------|----------|
| v2.0.2 | 2026-05-09 | `com.puddingkc.ma2bt-2.0.2.zip` + `package.json` | 16.9 KB | 186 |
| v2.0.1 | 2026-05-05 | `com.puddingkc.ma2bt-2.0.1.zip` + `package.json` | 11.1 KB | 63 |
| v2.0.0 | 2026-05-05 | `com.puddingkc.ma2bt-2.0.0.zip` + `package.json` | 10.2 KB | 17 |
| v1.0.1 | 2026-05-04 | `MA2BT.unitypackage` | 6.8 KB | 2 |
| v1.0.0 | 2026-05-04 | `MA2BT.unitypackage` | 6.8 KB | 0 |

> v1.x 为 `.unitypackage` 格式；v2.x 切换为 VPM zip 包。

---

## 13. 架构与依赖（开发者参考）

### 13.1 项目结构

```
MA2BT/
├── Editor/
│   ├── MAToBlendTreePlugin.cs         # 核心 NDMF 插件
│   └── com.puddingkc.ma2bt.Editor.asmdef
├── Runtime/
│   ├── MAToBlendTree.cs               # MonoBehaviour 设置组件 + 自定义 Inspector
│   └── com.puddingkc.ma2bt.Runtime.asmdef
├── .github/workflows/release.yml      # 自动打包 + 发布
├── LICENSE                            # Unlicense (公共领域)
├── README.md / README_EN.md / README_JP.md
└── package.json                       # VPM 包描述
```

### 13.2 核心 NDMF 插件注册

```csharp
[assembly: ExportsPlugin(typeof(MAToBlendTreePlugin))]

[RunsOnPlatforms(WellKnownPlatforms.VRChatAvatar30)]
internal class MAToBlendTreePlugin : Plugin
{
    public override string QualifiedName => "com.puddingkc.ma2bt";
    public override string DisplayName => "MA2BT";
    public override Color? ThemeColor => new Color(0.55f, 0.2f, 0.85f, 1);

    protected override void Configure()
    {
        Sequence seq = InPhase(BuildPhase.Optimizing);
        seq.AfterPlugin("nadena.dev.modular-avatar");  // 强制在 MA 之后
        seq.WithRequiredExtension(typeof(AnimatorServicesContext), s =>
        {
            s.Run("MA2BT Optimize", ctx =>
            {
                var settings = ctx.AvatarRootObject.GetComponent<MAToBlendTree>();
                if (settings == null) return;
                var optimizer = new LayerToBlendTreeOptimizer(ctx, settings);
                optimizer.Process();
                Object.DestroyImmediate(settings, true);  // 销毁组件
            });
        });
    }
}
```

### 13.3 数据模型

```csharp
internal class AnalyzedLayer
{
    public VirtualLayer Layer;
    public bool IsConvertible;
    public string RejectReason;
    public string ParameterName;
    public bool IsInverted;
    public List<StateInfo> States = new();
    public int OriginalIndex;
    public bool IsExternalLayer;
}

internal class StateInfo
{
    public bool IsDefault;
    public float ThresholdLo = float.NaN;
    public float ThresholdHi = float.NaN;
    public VirtualMotion Motion;
    public List<SecondaryCondition> SecondaryConditions = new();
}

internal class ResolvedState
{
    public StateInfo ActiveState;
    public StateInfo InactiveState;
    public List<SecondaryCondition> SecondaryConditions = new();
}

internal class ParameterGroup
{
    public string ParameterName;
    public List<AnalyzedLayer> Layers = new();
    public List<float> Thresholds = new();
}
```

### 13.4 核心常量

```csharp
const string ROOT_PARAM            = "zhz/1";
const string BLEND_TREE_LAYER_NAME = "MA_To_BlendTree_Layer";
const string MA_RESPONSIVE_PREFIX  = "MA Responsive: ";
```

> ⚠️ 根直混参数 `zhz/1` 是硬编码字符串，继承自原作者"浊鸷"的命名（保留部分命名是为了致敬原始作者）。

### 13.5 asmdef 引用

```json
// Editor/com.puddingkc.ma2bt.Editor.asmdef
{
  "references": [
    "com.puddingkc.ma2bt.Runtime",
    "nadena.dev.ndmf",
    "nadena.dev.ndmf.runtime",
    "nadena.dev.modular-avatar.core",
    "VRC.SDK3A",
    "VRC.SDKBase"
  ],
  "includePlatforms": ["Editor"]
}
```

---

## 14. 鸣谢

| 贡献者 | 贡献 |
|--------|------|
| **丨丿・丶乛** (https://space.bilibili.com/299071021) | 提供视频灵感 |
| **浊鸷** | 原作者，本插件基于其插件修改，保留部分命名 |
| **Null-K / PuddingKC** | 当前维护者，v2.x 完全重构 |

---

## 17. 分叉对比：MA2BT-Pro（zhuozhi233 fork）

> `[PROJECT-SOURCE]` 来源：`zhuozhi233/MA2BT-Pro` 的 README 与 `package.json`（v1.3.0）
> URL: https://github.com/zhuozhi233/MA2BT-Pro | 访问日期 2026-10-24
>
> ⚠️ **这是与本页正文不同的另一个包**。上文的包名、VPM 源、选项默认值均指 **上游 `Null-K/MA2BT`**；
> 本节的数值只适用于 **MA2BT-Pro**，二者**不可混用**。

### 17.1 两者是什么关系

| 维度 | MA2BT（上游，本页正文） | MA2BT-Pro（分叉） |
|------|------------------------|-------------------|
| 仓库 | `Null-K/MA2BT` | `zhuozhi233/MA2BT-Pro`（**fork**，`fork: true`） |
| 维护者 | Null-K / PuddingKC | 浊鸷（zhuozhi233） |
| VPM 包名 | `com.puddingkc.ma2bt` | **`com.zhuozhi.ma2bt-pro`** |
| VPM 仓库 | `https://null-k.github.io/vpm-listing/index` | **`https://zhuozhi233.github.io/vpm-listing/index.json`** |
| 版本（查证时） | v2.0.2 | **v1.3.0** |
| 依赖 | Unity 2022.3 / SDK 3.x / MA ≥1.10 / NDMF ≥1.4 | Unity 2022.3 / SDK 3.x / MA **≥1.10** / NDMF **≥1.9.0** |
| 注入层名 | `MA_To_BlendTree_Layer` | `MA_To_BlendTree_Layer`（**同名**） |
| License | Unlicense | Unlicense |

> 💡 因两者**注入层同名**，二者**功能重叠且不应同时安装**——同时存在会把同一批 `MA Responsive:` 层重复处理。

**Pro 的自述定位**：保留上游"把 MA 响应式层转 BlendTree"的核心思路，增加面向复杂项目的扩展与保护逻辑；**无法安全转换时保留原图层，不强制处理**。

### 17.2 Pro 相对上游增加的扩展

| 扩展 | 说明 |
|------|------|
| 多状态图层支持增强 | 允许转换包含多个条件状态的图层 |
| Bool / Int 参数转换与保护 | 增强 Bool/Int 条件处理，并对无法安全迁移的参数加保护 |
| Bool / Int 条件重写保留 | 支持保留图层中的 Bool/Int 条件重写 |
| 合并相同结构的混合树与动画 | 多个图层的多状态参数结构完全一致时，合并为同一嵌套混合树并合并相同状态动画 |
| **前缀排除** | 图层名 / 状态名 / 参数名可配置前缀排除 |
| 详细 Console 日志 | `[MA2BT Pro]` 逐条输出跳过原因，便于排查 |
| 扫描所有 FX 图层（可选） | 默认关闭；开启后也会尝试处理部分手动制作的简单图层 |

### 17.3 Pro 的选项与默认值（与上游不同，注意区分）

| 选项（中文界面） | 默认值 | 说明 |
|------------------|--------|------|
| **紧凑模式** | **开启** | 只生成需要的阈值，减少多余节点 |
| **多状态图层** | **开启**（上游默认关闭） | 允许转换包含多个条件状态的图层 |
| **合并相同混合树 / 动画** | **开启** | 结构完全一致时合并嵌套混合树与同名动画 |
| **扫描所有图层** | 关闭 | 不仅扫描 MA 生成的层，也尝试扫描所有 FX 图层 |

**排除项默认值**：

| 排除项 | 默认值 |
|--------|--------|
| 图层名前缀排除 | `lilycalInventory`、`AutoDresser` |
| 状态名前缀排除 | `Root`、`root` |
| 参数名前缀排除 | （空） |

> ⚠️ 上游本页 §16 速查表记录的默认值是 `Multi-State Layers = false`、`Scan All Layers = false`。
> **Pro 的"多状态图层"默认是开启的**——切换工具时务必重新核对此项，否则转换范围会比预期更大。

### 17.4 Pro 的跳过原因（何时会保留原图层）

Pro 不会"强行转换"。以下条件会保留原图层并在 Console 输出原因：

| 类别 | 具体原因 |
|------|----------|
| 扫描范围 | 图层不符合扫描范围（默认只扫 `MA Responsive:` 前缀） |
| 前缀排除 | 图层名 / 状态名 / 参数名命中排除列表 |
| 结构缺失 | 状态数量不足、没有默认状态、没有可解析的 Entry Transition |
| 选项限制 | 多状态图层存在但该选项被关闭 |
| 不安全内容 | 存在 StateMachineBehaviour；存在复杂过渡 / 不支持的 AnyState 结构 |
| 过渡设置不安全 | Transition Duration ≠ 0、Offset ≠ 0、Exit Time 不符合要求 |
| 状态设置不安全 | 启用了 Speed Multiplier / Motion Time / Mirror / Cycle Offset / Foot IK |
| 条件无法解析 | 参数缺失、类型不支持、无法转成 BlendTree 可用范围 |
| Int 条件不合规 | 使用 `NotEqual`，或 `Equals` 阈值非整数 |
| 参数共享冲突 | Bool/Int 参数同时被其他保留图层使用且无法安全改写 |
| 版本过旧 | 缺少所需 API 时**跳过全部优化**并在 Inspector 显示错误 |

> 💡 这与上游本页 §2 记录的跳过原因基本同构，但 Pro 增加了**前缀排除**与**参数共享冲突**两类。

### 17.5 选型建议（经验性，标 `[INFERENCE]`）

| 场景 | 建议 |
|------|------|
| 项目简单、只需要"把 MA 层塞进一个 BlendTree" | 上游 **MA2BT** 足够，选项少、行为更可预测 |
| 项目复杂、图层包含多状态，或需要排除特定前缀（如 `lilycalInventory`） | **MA2BT-Pro** 的排除项与多状态支持更实用 |
| 需要排查"为什么这层没被转换" | Pro 的 `[MA2BT Pro]` 日志粒度更细 |
| **两者二选一** | ✅ 必须；因注入层同名，不应同时安装 |

---

## 18. 相关文档（Cross-Reference）

| 文档 | 关系 |
|------|------|
| **[ndmf-tools.md](ndmf-tools.md)** | NDMF 工具生态总览（含 MA2BT 执行顺序） |
| **[modular-avatar.md](modular-avatar.md)** | MA 响应式系统（MA2BT 的输入源） |
| **[avatar-optimizer.md](avatar-optimizer.md)** | AAO（互补工具，处理 Skinned Mesh / Bone / PhysBone） |
| **[optimization-guide.md](optimization-guide.md)** | Avatar 完整最佳化实操指南 |
| **[performance-rank.md](performance-rank.md)** | PC/Quest 性能等级标准 |
| **[avatar-modding-guide.md](avatar-modding-guide.md)** | Avatar 改模教学 |

---

## 16. 速查表（Cheat Sheet）

```
┌──────────────────────────────────────────────────────────────┐
│ MA2BT 速查                                                   │
├──────────────────────────────────────────────────────────────┤
│ 工具: MA2BT (Modular Avatar to BlendTree)                   │
│ 平台: 仅 VRChat Avatar 3.0                                  │
├──────────────────────────────────────────────────────────────┤
│ VPM: https://null-k.github.io/vpm-listing/index              │
│ GitHub: https://github.com/Null-K/MA2BT                      │
├──────────────────────────────────────────────────────────────┤
│ 依赖: Unity 2022.3+ / VRChat SDK 3.x / MA ≥1.10 / NDMF ≥1.4 │
├──────────────────────────────────────────────────────────────┤
│ 默认选项:                                                    │
│   Compact Mode = true (推荐保持)                              │
│   Multi-State Layers = false (保持关闭)                      │
│   Scan All Layers = false (保持关闭)                         │
├──────────────────────────────────────────────────────────────┤
│ 注入层: MA_To_BlendTree_Layer (同名复用)                     │
│ 根直混参数: zhz/1 (硬编码)                                    │
├──────────────────────────────────────────────────────────────┤
│ 不支持: Equals/NotEqual / 多参数 AND / 子状态机 / AnyState    │
├──────────────────────────────────────────────────────────────┤
│ ⚠️ 本速查表仅适用上游 MA2BT (com.puddingkc.ma2bt)             │
│    分叉 MA2BT-Pro 包名=com.zhuozhi.ma2bt-pro，默认值不同      │
│    Pro 的「多状态图层」默认=开启，详见 §17                    │
├──────────────────────────────────────────────────────────────┤
│ 反馈: 简体中文 QQ群 1047423396 / 其他 GitHub Issues          │
└──────────────────────────────────────────────────────────────┘
```
---

## 相关页面

[[ndmf-tools.md]] · [[modular-avatar.md]] · [[avatar-optimizer.md]] · [[optimization-guide.md]] · [[performance-rank.md]] · [[avatar-modding-guide.md]]
