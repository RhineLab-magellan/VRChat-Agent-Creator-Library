---
title: "Avatar Animator System — Complete Guide"
category: avatar
knowledge_level: applied
status: active
source: 本地知识库整理
source_type: community
version: 1.4
last_review: 2026-07-04
confidence: High
tags:
  - avatar
  - animator
  - rig-requirements
  - armature
  - write-defaults
  - direct-blend-tree
  - vrc-school
aliases:
  - 动画器
  - "Avatar Animator System — Complete Guide"
related:
  - avatar-modding-guide.md
  - lac-avatar-compressor.md
  - ma-component-cards.md
  - ma2bt.md
  - modular-avatar-tutorials-detailed.md
  - playable-layers.md
type: entity
created: 2026-07-04
sources: 本地知识库整理
updated: 2026-07-04
---
# Avatar Animator System — Complete Guide

> 来源: VRCLibrary - Miscellaneous Avatar Knowledge (翻译版)
> 原文: https://notes.sleightly.dev/ | https://vrclibrary.com/wiki/books/miscellaneous-avatar-knowledge
> 译者: LIII Works / VRCD | 截止: 2024-09-05
>
> **补充来源**: VRC School（vrc.school）+ VRChat 官方文档（creators.vrchat.com）
> 补充日期: 2026-07-04

---

## 0. Animator 组件基础（VRC School 补充）⭐

> 来源：VRC School + Unity 官方文档
> 重要性：理解 Animator 组件是 Avatar 开发的基础

### 0.1 Animator 组件的创建

- Unity 自动为含 Humanoid/General 骨架的 FBX 添加 Animator 组件
- 手动添加：Inspector → Add Component → Animator
- 路径：模型 Inspector → "Create From This Model" 或 "Copy From Other Avatar"

### 0.2 Animator 组件的五个核心字段

| 字段 | 用途 |
|------|------|
| **Controller** | 动画控制器，指挥如何播放动画片段 |
| **Avatar** | 骨骼映射与人形肌肉系统（4种类型：Humanoid/Generic/Legacy/None）|
| **Apply Root Motion** | 根运动应用模式 |
| **Update Mode** | 更新模式（Normal/Physics/Unscaled Time）|
| **Culling Mode** | 剔除模式（Always Animate/Cull Update Transforms/Cull Completely）|

### 0.3 路径解析规则

- 动画片段的目标路径 = 相对于**最近的父级 Animator 组件**
- 根部 Animator 下的子对象：路径为 "子对象名"
- 嵌套 Animator 下的子对象：路径为 "父对象/子对象"

### 0.4 VRChat 特定行为

| 设置 | VRChat 本地 Avatar | VRChat 远程 Avatar | 子游戏对象 Animator |
|------|-------------------|-------------------|-------------------|
| Apply Root Motion | 默认 "Normal" | 默认 "Normal" | 不受影响 |
| Update Mode | 默认 "Normal" | 默认 "Normal" | 不受影响 |
| Culling Mode | **Always Animate** | **Cull Update Transforms** | 不受影响 |
| Controller 字段 | **不使用**，由 VRC Avatar Descriptor 替换 | 同上 | **使用**分配的 Controller |

**关键含义**：
- VRChat Avatar 的 Animator Controller 字段被 VRC Avatar Descriptor 的 Playable Layers 组合所取代
- 但子 GameObject 上的 Animator 不受影响，正常使用自己的 Controller

### 0.5 动画片段属性路径示例

- 子对象相对于角色上的 Animator：路径为 `父对象/子对象`
- 子对象相对于父级 Animator：路径直接为 `子对象`
- 向动画窗口添加属性时，路径根据最近的父级 Animator 组件添加

### 0.6 动画窗口行为

根据 Animator 组件状态，动画窗口内容变化：
1. **未检测到 Animator/Controller/Clip** → 窗口可能为空或无功能
2. **从项目窗口选择 Clip** → 显示该 Clip，但 Preview/Record 功能不可用（除非选择带 Animator 的 GameObject）
3. **拥有 Animator + Controller + ≥1 Clip** → 完全启用所有功能

### 0.6.1 初学者最常混淆的两组概念（教学补充）
- **Record vs Preview**
  - `Preview`：预览当前动画效果，不代表你正在写入关键帧。
  - `Record`：进入录制状态后，属性改动会被记录到当前 Clip。
  - 教学上应明确提醒：**看到动画在动 ≠ 你已经在写关键帧**。
- **Dopesheet vs Curves**
  - `Dopesheet` 更适合看关键帧时间点与整体结构。
  - `Curves` 更适合微调插值与连续变化趋势。
  - 初学者常先在 Dopesheet 建结构，再到 Curves 修手感。

### 0.6.2 Clip 制作的最小教学链路
- 最小可讲清流程：`Animator → Clip → Controller → Layer`
- 实操中建议提醒：
  - 至少有两个关键帧，才更容易让初学者理解“发生了变化”；
  - 静态片段通常用于开关 / 默认状态，动态片段更适合连续运动；
  - 常见动画属性大致分为 `GameObject`、`Component`、`Material` 三类。

---

## 0.7 Hips 作为根骨骼的硬性要求（VRCD 21 补充）⭐

> **来源**: [Avatar 文档（vrcd.org.cn）](https://docs.vrcd.org.cn/books/avatar) §第二部分「为什么一切都基于 Hips」 + VRChat 官方 Rig Requirements 文档
> **本地化日期**: 2026-07-04
> **重要程度**: 🔴 硬性要求（不符合会导致 IK 异常）

### Avatar 骨架的核心结构

```
Nebbia (根)
├── Armature (骨架,概念本身)
│   ├── Hips (髋部,最大父)
│   ├── Head, Hand, Spine 等
│   └── ...
├── Belt (Mesh)
├── Body (Mesh)
└── Watch (Mesh)
```

### 四个核心概念

| 概念 | 中文 | 定义 |
|------|------|------|
| **Root** | 根物体 | Avatar 整体,所有其他部分是它的一部分 |
| **Armature** | 骨架 | 包含所有骨骼的特殊 Object,**不实际控制 Mesh** |
| **Hips** | 髋部 | 骨架中最大的父,大腿骨和骨盆相连的骨骼部位 |
| **Mesh** | 网格 | 实际看到的东西（头发、脸、身体、服装等） |

### 🔴 硬性要求：Hips 必须是所有 Humanoid 骨骼的 ancestor

**VRChat 官方警告（原文）**:
> "This avatar has a split heirarchy (Hips bone is not the ancestor of all humanoid bones). IK may not work correctly."
> "Be very careful with these kinds of rigs! Many of these rigs are unsuitable for use with VRChat and need to be re-rigged to work properly."

**含义**:
- Hips 骨骼必须是所有其他 Humanoid 骨骼的**父级或更高级**
- 若骨架被拆分为上半身/下半身两个部分 → 放在 Body > Hips 槽位的骨骼必须是其余 Humanoid 骨骼的祖先
- 许多"下半身独立骨架"的 Avatar **不符合 VRChat 要求**,需要重新骨架

### 为什么 Hips 是最大的父（教学性解释）

1. **3D 建模惯例**: 髋骨在人体骨骼中相当于**中心轴点**
2. **动画稳定性**: 髋骨作为根骨骼提供稳定基础,确保其他骨骼按正确层级运动
3. **自然动作模拟**: 围绕髋骨的层级结构可更自然地模拟行走、跳跃等

### Armature 的本质（重要）

> 🔴 **Armature 是"骨架"这个概念本身,不实际控制 Mesh**

类比（VRCD 21 原文）:
- 人群（概念）= Armature
- 张三、李四、王五（实际人）= 骨骼
- 驱散"人群" → 操作具体的人（骨骼）
- Armature 是"人群"概念,真正做事的是里面的"人"

**进一步警告**:
- 即使骨头不在 Armature 内,也可以正常操控 Mesh
- 但**骨头所属的 Armature 必须存在,不能被移除**

### 引用

- [L1] https://creators.vrchat.com/avatars/rig-requirements/ - "General Hierarchy" 章节
- [L2] https://docs.vrcd.org.cn/books/avatar - VRCD 21「Avatar 与你」§为什么一切都基于 Hips

---

## 1. Write Defaults（写入默认）

### 核心概念

Write Defaults 决定动画属性在**离开 Animator 状态后**是否保持更改。

| 设置 | 行为 |
|------|------|
| **WD On** | 离开状态时，属性返回到上传时的默认状态 |
| **WD Off** | 离开状态时，更改保持不变 |

**默认状态**：Avatar 上传时物体所处的状态（如缩放、启用状态）

### 1.0.1 具体示例：帽子开关（来源：VRCLibrary）

> 来源：VRCLibrary - Miscellaneous Avatar Knowledge（VRCD 中文社区翻译版）
> 原文：https://notes.sleightly.dev/write-defaults/
> 纳入日期：2026-07-04（VRCLibrary 翻译版 P3 纳入）

**场景**：上传时帽子关闭（默认状态 = 关闭）

**WD On（启用）**：
- 当切换到 `Hat On` 状态时：帽子会被打开
- 当切换回 `Empty State` 状态时：帽子会被关闭（因为帽子被写回了默认状态）

**WD Off（禁用）**：
- 当切换到 `Hat On` 状态时：帽子会被打开
- 当切换到 `Empty State` 状态时：帽子会保持开启状态（因为没有动画将其关闭，也没有写回默认状态）

**教学价值**：这个例子直观说明了 WD On/Off 的核心差异——"WD On 会重置，WD Off 会保持"。

---

### 核心规则（必须遵守）

> ⚠️ 违反这些规则会导致动画表现异常

1. **禁止同一控制器中混用 WD On/Off**
   - 例外：单 State 的 Direct Blend Tree 可单独使用任意 WD
   - 原因：播放单个 WD Off 状态会导致所有 WD On 状态无法写入默认值

2. **禁止 WD Off 的 Direct Blend Tree**
   - 会产生奇怪行为：影响其他 BT、输出被平滑化
   - 解决：使用 WD On，单 State 场景下安全

3. **WD Off 状态必须有填充动画**
   - 建议：空属性 + 至少 2 帧（约 0.0167 秒）
   - 原因：空状态可能用之前层的动画默认值覆盖之前动画

4. **Transform 动画会覆盖 Gesture 层**
   - FX 层用 WD Off 控制 Transform → 覆盖 Gesture 层所有 Transform
   - 解决：配合 Avatar Mask 使用

### 优点/缺点对比

| | Write Defaults On | Write Defaults Off |
|---|---|---|
| **优点** | 简单（可用空状态作 Buffer）<br>不覆盖 Gesture 层 Transform<br>2 状态切换帧时间少 33%（Unity 2019）<br>易修改默认状态 | 不依赖上传状态<br>可混用 WD（通常不受影响）<br>可使用序列操作 |
| **缺点** | 依赖上传状态<br>混用时出问题<br>无法使用序列操作 | 需要填充动画<br>覆盖 Gesture 层 Transform（无遮罩时）<br>2 状态切换帧时间多 50%（Unity 2019） |

### Blendshape 值翻三倍问题

**触发条件**：
- Additive 层为 WD Off
- 从空 WD On 状态过渡到 Blendshape WD On 状态

**解决方案**：
1. 每个状态都明确动画化 Blendshape 值（正确方法）
2. 将空闲层状态设为 WD On（快捷但不完善）

---

## 1.1 Write Defaults 深度细节（VRC School 补充）⭐

> 来源：VRC School（vrc.school）+ VRChat 官方文档交叉验证
> 补充点：数学模型、性能数据、特殊触发条件
> 验证状态：2026-07-04

### Direct Blend Tree + WD Off 的数学模型

**关键公式**（直接混合树中子节点的最终权重）：

```
最终权重 = 当前子节点权重 × (x / (1 - yx))
```

其中：
- `x` = 当前子节点的权重
- `y` = 其他子节点权重之和 / 当前子节点权重

**陷阱**：
- 若分母 `1 - yx ≤ 0` → 权重趋于无穷大，**永远叠加**
- 若 n 个子节点各设 `1/n` 权重 → 乘数 = 1，行为正常（但仍不推荐）

**建议**：使用 **WD On** + 简单直接

### Unity 2019 性能数据（实测）

| 场景 | WD On | WD Off |
|------|-------|--------|
| 2 状态切换帧时间 | **基线** | 多 50% |
| 2 状态切换帧时间（反向） | **少 33%** | 基线 |

> 含义：创作者应**根据场景选择** WD 设置
> 简单 Toggle 场景 → WD On 性能更好
> 复杂序列操作场景 → WD Off 功能更强大

### Blendshape 三倍化 Bug 详细触发条件（VRC School 经验）

**精确触发**：

1. 存在一个**空 WD On 状态**（无任何动画属性的关键帧）
2. 该状态过渡到另一个**动画化 Blendshape 的 WD On 状态**
3. Additive 层设置为 **WD Off**
4. 过渡时 Blendshape 默认值被乘以 3

**避免方法**：
1. **正确方法**：每个状态都**显式动画化** Blendshape 值
2. **快速但不完美**：将空闲层中的状态设为 WD On

**已知事实**：
- 0×3=0，100×3=100 → 只用 0 和 100 时问题不显
- VRChat 启用了 `Clamp Blendshapes`（项目设置 → 玩家 → 其他设置，已弃用）
- 该问题与**任何遮罩**无关，也与动画是否可达无关

### 跨 WD 兼容预制件的设计指南

> 出处：VRC School + VRLabs Avatar 3.0 Manager 工具

- 每个层只动画化**一组属性**（层中每个状态都动画相同的属性集）
- 多层动画同一属性时，确保每个状态都有动画
- 顶部有带默认状态的**重置层**（Reset Layer）

**遵循这些指南 → 你的系统将同时适用于 WD On 和 WD Off**

### 工具辅助

| 工具 | 用途 |
|------|------|
| **VRLabs Avatar 3.0 Manager** | 检查 WD 混用、批量设置 WD |
| 工具限制 | 不处理 Direct Blend Tree 例外 → Direct BT 状态名需带 `(WD On)` 后缀 |
| **VRChat SDK** | 检测到混用 WD 时给出警告（Mixed Write Defaults）|

### 创作者选择建议

1. **简单 Avatar**：建议 **WD On**，利用性能优势（少 33%）
2. **复杂 Avatar**：建议 **WD Off** + 显式动画化所有属性
3. **跨 Avatar 复用的预制件**：遵循上述兼容指南
4. **Direct Blend Tree**：必须 WD On
5. **Additive 层**：VRChat 社区推荐使用 WD On（即使其他层是 Off）

> [FACT] **VRChat 官方说明**：
> 无论选择哪个选项，我们都建议在整个 Avatar 中**保持使用 Write Defaults 的一致性**——也就是说，所有状态都使用 Write Defaults Off，或者都使用 On。同一 Avatar 上同时使用 Off 和 On 状态**会导致意外的属性值**，这通常被称为"混合 Write Defaults"。SDK 会在检测到这种情况时**给出警告**。

---

## 2. Avatar Mask（人形遮罩）

### Unity 行为

Avatar Mask 基于启用/禁用的选项，控制 Humanoid Muscle、材质球替换、Transform 动画：

| 遮罩设置 | 效果 |
|---|---|
| 启用 Humanoid Muscle | 该层可修改该 Humanoid 值 |
| 禁用 Humanoid Muscle | 该层不能修改该 Humanoid 值 |
| 启用/未指定 Transform | 该 Transform 可被动画修改位置/旋转/缩放 + 替换第一材质球 |
| 禁用 Transform | 该 Transform 不能被动画修改 |

**关键约束**：
- 有 Avatar Mask 的层 → 不能控制除第一个外的材质球
- 有 Avatar Mask 的层 → 不能控制 Animator 的根 Transform

### 为什么需要 Avatar Mask

- 动画层按顺序播放（下方层覆盖上方层）
- 如果你在 Gesture 层控制手部，FX 层也控制手部 → FX 覆盖 Gesture
- Transform 覆盖只在第二个动画使用 WD Off 且在稍后应用的层时发生

**示例**：FX 层控制面部表情（同时控制 Humanoid Muscle）→ 手部动作被覆盖

### Gesture 层特殊行为

VRChat 将第一层的 Avatar Mask 应用到其下所有动画层。如果某层遮罩禁用了 Humanoid 或 Transform，该属性无法被动画控制。

---

## 3. VRChat Playable Layers 行为

### 播放顺序（从上到下）

```
1. Base
2. Additive
3. Gesture
4. Action
5. FX（最后播放，优先级最高）
```

### 关键约束

> ⚠️ 前四层只在本地 Avatar 克隆体播放，VRChat 只复制 Transform 和 GameObject 开关状态到镜像/阴影克隆体

因此：
- Blendshape、材质替换、着色器动画等 → **必须在 FX 层**
- Base/Additive/Gesture/Action → **只控制 Transform 和物体开关**

### 各层详细说明

| 层 | 用途 | 内容限制 | 推荐 |
|---|---|---|---|
| **Base** | 运动（Humanoid Muscle） | Transform + GameObject 开关 | 仅 Humanoid Muscle |
| **Additive** | 运动调整（如呼吸） | Transform + GameObject 开关 | 仅 Humanoid Muscle，叠加方式 |
| **Gesture** | 手势 + 身体部分动画 | Transform + GameObject 开关 | Transform + 特定 Humanoid |
| **Action** | 覆盖前面 Humanoid（如表情） | Transform + GameObject 开关 | 权重从 0 开始，使用后归零 |
| **FX** | 所有其他动画 | 无限制 | 一切非 Transform 内容 |

### Action 层特殊说明

- 默认权重为 0
- 使用 `Playable Layer Control State Behavior` 将权重混合到有效值
- 完成后必须将权重调回 0，否则覆盖其他层

### FX 层特殊行为

- 第一层无 Avatar Mask → 创建默认遮罩（禁用所有 Humanoid + 启用所有 Transform）
- 第一层有 Avatar Mask → 应用到所有层
- 想要在 FX 控制 Humanoid → 制作专门遮罩并在最顶层设置
- VRChat 默认 FX 层有 WD Off 混合树，可能导致 Transform 动画问题

### 特殊层（独立，不受 Avatar Mask 影响）

| 层 | 用途 |
|---|---|
| **T-Pose** | 确定视点/视角球放置 |
| **IK Pose** | 确定关节弯曲方向（应略微弯曲） |
| **Sitting Pose** | 控制坐下姿势（覆盖 Humanoid 动画） |

---

## 4. Expression Parameter Mismatch（参数类型不一致）

### 兼容表格

#### Animator Bool →

| Expression | 结果 |
|---|---|
| Bool | Bool = False / True |
| Int | Int = 0 / 1 |
| Float | Float = 0.0 / 1.0 |

#### Animator Int →

| Expression | 结果 |
|---|---|
| Bool | Int > 0 → True |
| Int | 直接传递 |
| Float | 直接转换（如 2 → 2.0） |

#### Animator Float →

| Expression | 结果 |
|---|---|
| Bool | Float > 0 → True |
| Int | 四舍五入（≥0.5 → 1, <0.5 → 0） |
| Float | 直接传递 |

### 底层原理

- Unity Animator 后端使用 **Float** 处理参数
- VRChat 使用 **SBytes**（signed byte）代替 bool/int/float
- 参数不是"转换"，而是"不一致的兼容"

---

## 4.1 表达式参数官方同步规格（VRChat 官方文档）⭐

> 来源：VRChat 官方 Animator Parameters 文档（creators.vrchat.com/avatars/animator-parameters/）
> 重要性：⭐ 创作者必须理解同步精度限制，避免设计超出范围的参数
> 验证状态：2026-07-04

### 参数内存使用

| 参数类型 | 同步范围 | 内存使用 | 编码方式 |
|---------|---------|---------|---------|
| `Int` | 0 - 255 | **8 bits** | Unsigned 8-bit int |
| `Float` | -1.0 到 1.0 | **8 bits** | Signed 8-bit **fixed-point decimal** |
| `Bool` | True / False | **1 bit** | 单 bit |

### 同步精度关键事实

> ⚠️ **远程同步的 Float 值有 255 个可能值**（即 1/127 精度）
> 能精确存储 -1.0、0.0、1.0
> 范围：`-1, -126/127, -125/127, ..., 126/127, 1`

**本地 vs 远程**：
- **本地**（OSC 等修改）：存储为原生 32-bit float
- **远程**（同步后）：8 bits fixed-point，255 个可能值

### 参数总限制

```
✅ VRChat 最多同步 256 bits 的自定义参数
✅ Avatar 最多 8192 个总自定义参数（同步+非同步）
✅ 内置参数不计入上述限制
```

### 内置参数示例

| 名称 | 描述 | 类型 | 同步 |
|------|------|------|------|
| `IsLocal` | 本地/远程 | Bool | None |
| `Viseme` | Oculus 唇形索引 (0-14) | Int | Speech |
| `Voice` | 麦克风音量 (0.0-1.0) | Float | Speech |
| `GestureLeft/Right` | 手势 (0-7) | Int | IK |
| `GestureLeftWeight/RightWeight` | 扳机值 (0.0-1.0) | Float | Playable |
| `VelocityMagnitude` | 速度大小 | Float | IK |
| `VelocityX/Y` | 速度分量 | Float | IK |

### 同步模式

| 模式 | 用途 | 更新频率 |
|------|------|---------|
| **Slow sync** | 较长运行动画状态 | 每 0.1-1 秒（按需）|
| **Continuous sync** | 频繁变化值 | 持续每 0.1 秒（10/秒），远程插值 |

### 创作者含义

1. **避免设计大范围 Float 参数**（如 0-1000）→ 同步后只剩 255 档
2. **滑块/转盘等连续值控制** → 提前量化到 [-1, 1] 范围
3. **Bool/Trigger 优势**：1 bit vs 8 bits → 节省同步带宽
4. **记住非同步参数**：不被同步，但可被 Contact/PhysBone/Parameter Driver 设置

### Default AV3 Aliasing

- **Action 层默认使用 aliasing** → 参数名 `VRCEmote`（Int 1-16）
- **FX 层默认使用 aliasing** → `VRCFaceBlendH`（-1, 1）+ `VRCFaceBlendV`（-1, 1）
- 默认 FX 层要求 Avatar 有 `Body` 蒙皮网格 + 4 个 Blendshape（`mood_happy/sad/surprised/angry`）

---

## 4.5 为什么要用 Direct Blend Tree?设计哲学 ⭐

> 来源补充:VRCD 中文社区《VRChat Avatar 开发工具箱》+ 6 个权威源交叉验证(2026-07-04)
> 关联:`§5 Direct Blend Tree`(本节提供"为什么"层,§5 提供"怎么用"层)

### 核心论点

Blend Tree 在 VRChat 中被"过度使用"承担了本不属于它的设计责任,根因有 3 个:

#### 约束 1:Animator Layer 性能成本高

| 平台/等级 | Animator 数量上限 | 性能影响 |
|----------|-----------------|---------|
| Excellent | 1 (仅 Root Animator) | 几乎无开销 |
| Good | 4 | 约 +50% 帧时间/层 |
| Medium | 16 | 显著 |
| Poor | 32 | 严重 |

**VRChat Wiki 量化结论**(Community:VRChat Performance Benchmarks):
> "When the surprisingly large cost of layers was first discovered, people called for Direct Blend Trees as the one magical solution that would cut frame time by orders of magnitude."
> "For a basic Direct Blend Tree setup... These results shown a 3/4ths cut to our frame time."

> **结论**:Direct Blend Tree 相对普通 Animator Layer **可减少约 75% 帧时间**。
> **推论**:Layer 数量越多,性能越差 → 把多个开关/转轮合并到单层是"有利可图"的设计选择。

#### 约束 2:VRChat Avatar SDK 缺乏直接运行程序能力

- VRChat Avatar SDK **不能**像 Udon 那样直接执行任意代码
- 功能实现**必须**通过 Animator + State Behaviour 间接完成
- 主要工具:VRC Avatar Parameter Driver(详见 §4.6 限制)

#### 约束 3:VRC Avatar Parameter Driver 的能力边界

- ❌ **不能**修改 Built-in VRC Parameters
- ❌ **不能**修改 AAP(Animated Animator Parameters)
- ✅ 只能修改自定义的 Expression Parameters / Animator Controller Parameters

> **关键推论**:对于 Parameter Driver 不能修改的参数,创作者**必须**用 Animator 动画(Blend Tree 嵌套)来间接修改 —— 这就是 AAP 技术存在的核心原因。

### Unity Animator 的"自修改"特性

- Unity Animator 允许**动画控制器使用动画修改动画器自身的参数值**
- 这是 Unity 的设计特性(不是 VRChat 专有)
- 在 VRChat 场景下被"发掘"并广泛使用 → Blend Tree 承担 Parameter Driver 的部分职责

### 设计哲学总结

| 设计问题 | 创作者选择 |
|---------|-----------|
| 多个 Toggle/Radial | → 单层 Direct Blend Tree(避免多 Layer 性能开销) |
| 需要修改 AAP | → Blend Tree 嵌套(因为 Parameter Driver 不能改) |
| 复杂逻辑(加/乘/平滑) | → VRC School 高级 Blend Tree 技巧 |
| 帧时间独立效果 | → Frame Time Detection 模式 |

### 与 §5 Direct Blend Tree 的关系

| 本节(§4.5) | §5 Direct Blend Tree |
|------------|---------------------|
| **为什么**需要 Direct Blend Tree | **怎么用**Direct Blend Tree |
| 设计哲学层 | 操作层 |
| 量化数据(75% 帧时间收益) | 创建步骤 + 注意事项 |
| 约束分析 | 嵌套技术 + 限制 |

> 💡 **建议**:阅读 §5 之前先理解本节,可更好掌握 Direct Blend Tree 的工程价值。

### 引用

- VRCLibrary: <https://vrclibrary.com/wiki/books/miscellaneous-avatar-knowledge/page/combining-layers-using-direct-blend-trees>
- VRChat Wiki: <https://wiki.vrchat.com/wiki/Community:VRChat_Performance_Benchmarks>
- VRChat 官方 Playable Layers: <https://creators.vrchat.com/avatars/playable-layers/>
- VRChat 官方 Avatars: <https://creators.vrchat.com/avatars/>
- VRC School Advanced BlendTrees: <https://vrc.school/docs/Other/Advanced-BlendTrees/>

---

## 4.6 VRC Avatar Parameter Driver 的能力边界 ⭐

> 来源补充:VRChat 官方 `creators.vrchat.com/avatars/state-behaviors/`(2026-07-04)
> 关联:`§4.5 设计哲学`(解释 AAP 为何重要)

### 核心限制

VRC Avatar Parameter Driver **不能**修改以下两类参数:

| 参数类型 | 是否可改 | 影响 |
|---------|---------|------|
| 自定义 Expression Parameters(非 syncd) | ✅ 可改 | 常规用法 |
| 自定义 Animator Controller Parameters(local) | ✅ 可改 | 常规用法 |
| 自定义 Synced Expression Parameters | ✅ 可改(只本地改 + 自动 sync) | 同步参数 |
| **Built-in VRC Parameters** | ❌ **不能** | GestureLeft/Right 等 |
| **AAP (Animated Animator Parameters)** | ❌ **不能** | Blend Tree 修改的参数 |

### VRChat 官方原文

> 来源:<https://creators.vrchat.com/avatars/state-behaviors/>
>
> **"You can't use this to copy from and/or change [Built-In VRC Parameters] and [Animated Animator Parameters (AAP)]"**

### AAP 是什么

AAP = Animated Animator Parameters(动画化的 Animator 参数)

- 由 Unity Animator 的**动画**修改,而非由 State Behaviour 驱动
- 典型实现:Blend Tree 嵌套(Direct BT 内的 1D BT,或 1D BT 内的 BT)
- VRChat 官方(隐含):AAP 是创作者**必须**掌握的高级技术

### 为什么这是关键发现

| 场景 | 不用 AAP | 用 AAP |
|------|---------|--------|
| 多 Toggle 切换 | Parameter Driver 每改一个参数 = 1 帧开销 | Direct Blend Tree 1 帧搞定多个 |
| 加法/乘法/平滑 | Parameter Driver 不支持 | Blend Tree 嵌套可实现 |
| 修改内置参数 | ❌ Parameter Driver 不能改 | ✅ AAP 可改(BT 动画内置参数) |

### AAP 创建方法(简要)

参考 VRC School Advanced Blend Tree Techniques 教程:

```
Direct Blend Tree (Weight=1)
└── 1D Blend Tree (Parameter=OutputAAP)
    ├── Threshold -1 → Animation "Output=-1"
    └── Threshold +1 → Animation "Output=+1"
```

写入 `OutputAAP` 参数时:
- 写入 -1 → 播放 "Output=-1" 动画 → 修改目标属性
- 写入 +1 → 播放 "Output=+1" 动画 → 修改目标属性
- 利用 1D BT 的"插值"特性,可在两动画间实现平滑/加法/乘法

### 引用

- VRChat 官方: <https://creators.vrchat.com/avatars/state-behaviors/>
- VRLabs VRC School: <https://vrc.school/docs/Other/Advanced-BlendTrees/>

---

## 5. Direct Blend Tree（合并多动画层）

### 用途

将多个切换开关和转轮开关合并到**单层**，减少 Animator Layers 性能开销。

### 创建步骤

1. **创建 Blend Tree State**
   - 新建 Animator Layer → 右键创建 Blend Tree State
   - 设置 Write Defaults = **ON**（必要步骤）

2. **改为 Direct 类型**
   - 双击打开 → 检查器设置 Blend Type 为 `Direct`

3. **添加子项**
   - 添加新的 Blend Tree 子项
   - 创建 float 参数（如 `Weight`）设置为 1
   - 子项参数设为该 float（所有子项可共用）
   - **禁止勾选** `Normalized Blend Values`

4. **创建开关**
   - 子项 Blend Type 设为 `1D`
   - 创建 float 参数控制该子项
   - 添加动画（必须是独立动画，不能用 Motion Time 单动画）

5. **添加到 Expression Parameters**
   - 转轮开关 → `Float`
   - 切换开关 → `Bool`

### Write Defaults 处理

- 单 State 的 Direct BT 可安全使用 WD On
- 不受 Animator Controller 其他部分 WD Off 影响
- 名称前加 `WD On` 前缀可防止 VRLabs Avatar 3.0 Manager 批量修改时影响

### 嵌套技术

- 嵌套 Direct BT 模拟文件夹结构
- 每个文件夹是独立的 Direct BT
- 可在独立窗口中打开（大规模时性能友好）

### 当前限制

- 不支持动画切换（如溶解切换、Motion Time）
- 不支持驱动其他参数（排他性切换需 Parameter Driver）
- 首次运行时动画属性值可能在一帧内为 0

---

## 5.1 五种混合树类型详细算法（VRC School 补充）⭐

> 来源：VRC School + Advanced Blend Tree Techniques
> 重要性：创作者必须理解 5 种类型的算法差异
> 验证状态：2026-07-04

### 1D（一维）

- 基于**单个参数**混合运动
- **线性混合**
- 权重由参数值在阈值之间的位置决定
- 例：参数在两个阈值之间 80% 位置 → 左侧权重 0.2，右侧权重 0.8
- **唯一可用于 Blend Tree 的参数类型**
- 公式：`f(t) = a + (b - a) × fraction`

### 2D Simple Directional（2D简单方向）

- 基于**二维参数空间**
- 运动围绕 `(0,0)` 中心形成多边形

**权重计算步骤**：
1. 找到参数值所在的两个点
2. 检查值是否比连接线离 (0,0) 更远
3. 是 → 用到两点的距离作为权重（归一化）
4. 否 → 用到两点和原点的距离作为权重（归一化）
5. 若 (0,0) 处有运动 → 获得第三个权重

**适用**：角色控制器、瞄准控制器
**不适用**：基于距离的直接动画混合

### 2D Freeform Directional（2D自由形式方向）

- 需在 (0,0) 处有运动
- 适合每个方向需要多个运动的场景
- 基于方向工作
- 参考论文：https://runevision.com/thesis/rune_skovbo_johansen_thesis.pdf §6.3

### 2D Freeform Cartesian（2D自由形式笛卡尔）

- "距离分割 = 权重分割" 函数
- 提供更直接的运动混合方法

### Direct Blend Tree（直接混合树）

**核心特性**：
- 为每个动画分配**专用参数**控制其权重
- 权重和**不受 1 限制**（可同时播放多个动画）
- 可优化 VRChat Toggle 性能（合并多个 Toggle 到单层）

**关键参数**：
- `Normalize Blend Values`：勾选后权重归一化（除以总权重，保持和为 1）
  - 例：动画 1 权重 2，动画 2 权重 3 → 归一化为 0.4 和 0.6
- 建议以权重 1 播放 1D Blend Tree 而非直接播放动画（避免依赖默认值）
- 关闭 WD 时行为差异大，应避免

**片段时间长度公式**：
```
clipLength = (ownLength + sum(otherLengths/theirSpeeds)) / ownSpeed
```

**示例**：
- 片段 A：长度 1，速度 2
- 片段 B：长度 5，速度 4
- 直接混合树中 A 的长度 = (1 + 5/4) / 2 = 0.875

### 性能基准（VRC School 实测）

> [FACT] **Direct Blend Trees 是减少 Animator 层开销的关键技术**

| 方案 | 性能 | 复杂度 |
|------|------|--------|
| 简单 2 状态 Toggle | 基线 | 简单 |
| 1D Blend Tree + 子项 | 帧时间**减少约 3/4** | 中等 |
| 2 层 Direct Blend Tree | 帧时间**减少约 4/5** | 复杂 |

**结论**：
- 大量 Toggle 场景（10+）→ Direct Blend Tree 显著提升
- 常切换场景（面部追踪、Voice 参数）→ 收益最大
- 否则建议保留 2 状态 setup

### 创建 Direct Blend Tree Toggle 的标准步骤

1. 创建新 Animator Layer → 命名（如 "Blendtree"）→ Weight 设为 1
2. 右键创建 Blend Tree State
3. 启用 **Write Defaults**（否则 Toggle 行为异常）
4. Blend Type 改为 **Direct**
5. 创建 `Weight` Float 参数（值 1），置于参数列表顶部
6. 每个 Toggle 创建一个子 Blend Tree：
   - 1D 类型
   - Parameter = Toggle 专用 Float 参数
   - 添加 2 个 Motion Field（Off / On 动画）
7. 在 Expression Parameters 中：
   - 转盘/滑块 → `Float` 类型
   - 切换开关 → `Bool` 类型
8. **重要**：VRChat 会自动将 `bool` 转换为 `float`（节省 1 个同步参数槽）

> [FACT] **Advanced Blend Tree Techniques 进阶**：
> - 通过 AAP（Animated Animator Parameters）实现加减乘除四则运算
> - 加法：在同一 DBT 中动画化 AAP 两次（效果叠加）
> - 乘法：嵌套 Blend Tree（子 BT 权重 = 父 BT 权重 × 子 BT 权重）
> - 帧时间检测 + 平滑（指数/线性）可实现无 DBT 替代品

### 5.1.1 进阶应用：逻辑门（AND / OR / XOR / NOT）

> 来源：VRCLibrary + VRC School Advanced BlendTrees
> 纳入日期：2026-07-04（VRCLibrary 翻译版 P3 纳入）

通过 Blend Tree 嵌套可以模拟逻辑门，用于复杂的复合条件控制：

| 逻辑门 | 实现方式 | 用途 |
|--------|---------|------|
| **AND 门** | 两个子项权重同时 > 0 才输出 | 双条件同时满足才激活 |
| **OR 门** | 多个子项权重之和 > 0 即输出 | 任一条件满足就激活 |
| **XOR 门** | 通过标准化确保互斥 | 只允许一个条件激活 |
| **NOT 门** | 1 - 权重（通过反向动画）| 条件不满足时激活 |

**数学基础**：

- **加法**：在同一 DBT 中动画化 AAP 两次（效果叠加）
- **乘法**：嵌套 Blend Tree（子 BT 权重 = 父 BT 权重 × 子 BT 权重）
- **减法**：通过 NOT 门 + 加法组合实现
- **标准化**：通过 `Normalize Blend Values` 选项实现互斥（XOR 效果）

**典型应用场景**：

- **多条件解锁**：例如"VIP 用户" AND "特殊事件期间"才显示某个物品
- **互斥菜单**：通过 XOR 实现"单选"效果（如服装分类）
- **反向动画**：通过 NOT 门实现"按住反向键"等复杂交互

**参考实现**：

- VRC School Advanced BlendTrees: https://vrc.school/docs/Other/Advanced-BlendTrees/
- 包含完整的 AND/OR/XOR 门实现示例

---

## 6. Gesture Toggle 行为：Avatars 2.0 vs 3.0 ⭐

> [FACT] **Gesture Toggle** 是 Action Menu 中的开关，**默认开启**。
> 创作者需理解 2.0 和 3.0 Avatar 中**截然不同的行为**。

### 6.0 基础行为（2.0 / 3.0 通用）

> [FACT] **Gesture Toggle 启用时**，VRChat 都会**尝试**:
> - 将玩家的**当前手指姿势**与 [标准 VRChat 手姿](https://docs.vrchat.com/docs/valve-index#section-vrchat-standard-hand-poses) 匹配
> - 匹配结果作为输入驱动后续行为（2.0 触发 Override / 3.0 更新 Animator Parameter）
>
> **设置位置**:
> - **Action Menu**（默认）
> - **SteamVR Controller Bindings** 中可设为**自定义操作**

### 6.1 Avatars 2.0 行为

> [FACT] **Avatars 2.0 中，Gesture Toggle 控制 Gesture Overrides**:
>
> - **启用时**: 任何应用的 Gesture Override **会播放**
> - **禁用时**: VRChat **不尝试**匹配手势
> - **特殊行为（Valve Index）**: 启用时手部姿势**不变化**（即使定义了 Override）

> [FACT] **Avatars 2.0 中**:
> - **禁用时**正在播放的 Gesture Override **继续播放**
> - **重新启用** Gesture Toggle 后恢复手势匹配

### 6.2 Avatars 3.0 行为

> [FACT] **Avatars 3.0 中，Gesture Toggle 控制 Gesture Animator Parameters**:
>
> - **启用时**: Gesture Animator Parameters **正常更新**（如 GestureLeft/Right 0-7）
> - **禁用时**: Gesture Animator Parameters **"卡住"**在**禁用时的值**，无论输入如何

> [FACT] **创作者关键影响**:
> - **Avatars 3.0 中**，玩家禁用 Gesture Toggle = 你的 Gesture Layer Animator **不再更新**
> - **任何依赖 Gesture 参数的动画都会冻结**
> - **Fallback 动画** 应放在其他 Layer（如 Action）以保证 Gesture Toggle 禁用时仍工作

### 6.3 创作者设计建议

> [FACT] **创作者应**:
>
> 1. **设计容错**: 即使 Gesture Toggle 关闭，Avatar 应仍有**合理表现**
> 2. **关键动画放在非 Gesture Layer**: 例如 Default 姿势在 Base / Additive
> 3. **考虑玩家**: 玩家可能为了截图/录像禁用 Gesture Toggle
> 4. **测试两种状态**: 启用 + 禁用 Gesture Toggle 都应测试

> **典型场景**:
> - 玩家禁用 Gesture Toggle 截图 = Avatar 手指停在禁用瞬间的姿势
> - 创作者应确保此姿势**可接受**（非尴尬姿势）

> **开关位置详见** §6.0 + `avatar/expression-menu.md` §1.1 (Action Menu 根菜单)

---

## 工具推荐

| 工具 | 用途 |
|---|---|
| [VRLabs Avatar 3.0 Manager](https://github.com/VRLabs/Avatars-3.0-Manager) | 检查 WD 混用、批量设置 WD |
| [AV3 Emulator](https://github.com/knob到来/VRC_Avatars_3.0_Emulator) | 模拟参数类型兼容 |
| [GestureManager](https://github.com/Black-Horse-Inc/GestureManager) | 模拟参数类型兼容 |

---

## 来源

- Write Defaults: https://notes.sleightly.dev/write-defaults/
- Avatar Mask: https://notes.sleightly.dev/animator-masks/
- Parameter Mismatch: https://notes.sleightly.dev/parameter-mismatching/
- Direct Blend Trees: https://notes.sleightly.dev/dbt-combining/
- 完整来源: https://vrclibrary.com/wiki/books/miscellaneous-avatar-knowledge

### 补充来源（2026-07-04 纳入 VRC School + VRChat 官方）

- VRC School: https://vrc.school/（Avatar System + Write Defaults + Blend Trees）
- VRC School GitHub: https://github.com/VRLabs/VRCSchool
- VRChat 官方 Animator Parameters: https://creators.vrchat.com/avatars/animator-parameters/
- VRChat 官方 Playable Layers: https://creators.vrchat.com/avatars/playable-layers/
- VRChat 官方 Avatars: https://creators.vrchat.com/avatars/
- VRC School Advanced Blend Trees: https://vrc.school/docs/Other/Advanced-BlendTrees/
- VRC School Network Sync: https://vrc.school/docs/Other/Network-Sync/

---

## 7. Animator Controllers 与混合机制（VRC School 补充）⭐

> 来源：VRC School + Unity 官方文档
> 重要性：理解混合公式是设计复杂动画层的前提
> 验证状态：2026-07-04

### 7.1 动画层基本结构

每个动画层包含：
- 一组动画状态
- 一个初始动画状态（默认状态，橙色显示）
- 状态之间的过渡
- 始终处于：播放某状态 / 在两个状态间过渡（混合）

### 7.2 混合权重与混合模式

**核心机制**：将第一个值作为基础，迭代混合其下方的每一层。

#### Override 模式（覆盖）

```
NextValue = CurrentValue + NewWeight × (NewValue - CurrentValue)
```

#### Additive 模式（叠加）

```
NextValue = CurrentValue + NewWeight × NewValue
```

**计算示例**：
- 第一层播放混合形状 = 50（权重 1）
- 第二层（Override 模式）播放混合形状 = 100（权重 0.5）
- 最终值 = 50 + 0.5 × (100 - 50) = **75**

### 7.3 混合权重默认设置

- **默认权重 = 0**（即层处于非活动状态）
- **激活层**：将权重设为更高值（通常为 1）
- **第一层权重**：不可更改（始终为 1）

### 7.4 虚拟形象遮罩 (Avatar Mask)

- 创建：右键 → Create → Avatar Mask
- 启用/禁用人形肌肉和变换的动画
- 屏蔽动画的某些部分，使动画可在多处复用

**Import Skeleton 流程**：
- 将 FBX 中的 Avatar 放入 Avatar 插槽
- 按下 Import Skeleton 按钮
- 显示 FBX 上的所有变换，可启用/禁用

### 7.5 同步层（Sync Layer）

**Sync 复选框**：同步两个层（除动画外），状态机完全相同

**Timing 复选框**：
- 关闭：同步层动画被拉伸/压缩以与源层时间相同
- 打开：动画时间根据层权重组合

### 7.6 IK Pass

- 启用时：脚本接收 `OnAnimatorIK(int LayerIndex)` 回调
- 可根据 Animator 动画对象的位置更新 IK 目标
- **VRChat 中无用**：VRChat 不使用 Animator 组件的 Controller 进行 IK

### 7.7 Trigger 参数在 VRChat 中的限制

> Trigger 类似于 Bool，但触发后自动重置为 False
> **VRChat 没有内置的等效功能**，所以 Trigger 在 VRChat 中没有太多用途

### 7.8 Float 参数比较限制

- **唯一**可用于 Blend Tree
- **只能使用大于或小于**进行比较
- **不能使用等于**（因为它代表数轴上的一个点）

---

## 8. Animator States 与 Transitions 字段详解（VRC School 补充）⭐

> 来源：VRC School + Unity 官方 Animation States 文档
> 重要性：理解状态和过渡字段是设计复杂状态机的前提
> 验证状态：2026-07-04

### 8.1 Animator State 核心字段

| 字段 | 用途 |
|------|------|
| **Motion** | 状态播放的 Animation Clip 或 Blend Tree |
| **Speed** | 播放速度（正/负/零/浮点），可乘以 Parameter |
| **Motion Time** | 通过参数滚动到指定时间点（基于归一化时间）|
| **Write Defaults** | 决定是否在离开状态时恢复默认值 |
| **Mirror** | 镜像人形动画（可参数化）|
| **Cycle Offset** | 循环起点偏移（可参数化）|
| **Foot IK** | 启用脚部 IK（仅在层 IK Pass 同时启用时有效）|

**Motion Time 行为**：
- 例：5秒动画，Motion Time 参数 = 0.6 → 播放到 3 秒处
- 优先 Loop Time：Loop Time 启用时，参数 1 = 0%，1.5 = 50%

**Cycle Offset 叠加规则**：
- 动作时间 0.5 + Cycle Offset 0.2 = 70% 位置
- Loop Time 启用 + 动作时间 0.9 + Cycle Offset 0.4 = 30% 位置（= 0.9 + 0.4 - 1）

### 8.2 Animator State 内部三种状态

- **Any State（任意状态）**：象征性代表所有状态
- **Entry（进入）**：进入层的入口
- **Exit（退出）**：退出层（评估来自 Entry 的过渡）

**默认状态**：橙色显示，可通过右键 → Set as Layer Default State 修改

### 8.3 Animator Transitions 核心字段

| 字段 | 用途 |
|------|------|
| **Transitions List** | 多个过渡（用三箭头表示），可拖动重排 |
| **Conditions List** | 触发条件（AND 逻辑）|
| **Exit Time + Has Exit Time** | 退出时间（默认 1 = 100% 播放完）|
| **Transition Duration + Fixed Duration** | 过渡持续时间（秒 vs 比例）|
| **Transition Offset** | 目标状态播放起点偏移 |
| **Interruption Source** | 中断源（5 个选项）|
| **Ordered Interruption** | 有序中断（仅列表中更高的过渡可中断）|
| **Mute / Solo** | 调试用（不常用）|

**关键公式**：
- 过渡触发时间 = `Exit Time × Clip Length`
- 过渡持续时间：`Fixed Duration` 勾选 → 秒数；未勾选 → `Duration × Clip Length`

### 8.4 条件类型

| 类型 | 目标值 | 备注 |
|------|--------|------|
| **Float** | 任何浮点数 | 可用 > < 比较，不可用 = |
| **Int** | 任何整数 | 整数比较 |
| **Bool** | True / False | 布尔 |
| **Trigger** | 无目标 | 触发后自动重置为 False |

**AND 逻辑**：一个过渡上放置多个条件
**OR 逻辑**：使用两个独立过渡，每个一个条件

### 8.5 Any State 特殊行为

- 创建：右键蓝色 Any State → Make Transition → 选目标状态
- 行为：表现得像从所有状态发出
- 独特能力：可**中断任何过渡**
- 特殊选项 `Can Transition To Self`：是否允许目标状态过渡到自身

### 8.6 Interruption Source 五选项

| 选项 | 允许的中断源 |
|------|------------|
| **None** | 仅 Any State 过渡 |
| **Current State** | Any State + 源到目标 |
| **Next State** | Any State + 目标到任何状态 |
| **Current State Then Next State** | 顺序：Any State → 源到目标 → 目标到任何 |
| **Next State Then Current State** | 顺序：Any State → 目标到任何 → 源到目标 |

**顺序很重要**：选择找到的第一个允许的过渡

### 8.7 Visual Transition Editor

可视化编辑器，可拖动：
- 顶部蓝色开始/结束标记（修改 Exit Time / Duration）
- 中间过渡（调整开始结束 + 持续时间）
- 底部动画片段（修改 Transition Offset）

**白线含义**：左右脚之间的**枢轴权重**（0=左脚，1=右脚）
- 受禁用 "Bake Into Pose" 的根变换位置动画影响
- "实际上没什么用 :)"

### 8.8 退出时间陷阱

> ⚠️ 单关键帧动画片段的退出时间被视为 **1 秒**
> 建议创建两个关键帧的片段，每个关键帧相隔 1 帧，以避免混淆

### 8.9 Animator IK 在 VRChat 中的限制

> VRChat 不使用 Animator 组件的 Controller 进行 IK
> 因此：
> - Foot IK 对 VRChat 动画器**无用**
> - IK Pass + OnAnimatorIK 对 VRChat 内容**完全无用**（需要脚本回调）

---

## 9. Animation Clips 切线与片段类型（VRC School 补充）⭐

> 来源：VRC School + Unity 官方 Animation Editor 文档
> 重要性：切线类型决定动画曲线行为
> 验证状态：2026-07-04

### 9.1 动画片段创建

扩展名 `.anim`，三种创建方式：
1. 项目窗口右键 → Create → Animation Clip
2. 动画面板 → Create 按钮
3. 动画面板 → 片段名称下拉菜单 → Create New Clip

### 9.2 打开动画面板

四种方式：
- 顶部工具栏：Window → Animation → Animation
- 右键其他窗口 → Add Tab → Animation
- 快捷键：`Ctrl + 6`
- 双击任何 Animation Clip

### 9.3 录制模式 vs 预览模式

| 模式 | 行为 | 时间线颜色 |
|------|------|-----------|
| **Record Mode** | 修改任何可动画属性时，Unity 自动在播放头位置创建关键帧 | 红色 |
| **Preview Mode** | 修改游戏对象不会自动创建关键帧 | 蓝色 |

注意：Record 模式下 Preview 仍有效（同时记录新关键帧 + 预览现有动画）

### 9.4 关键帧切线类型

| 切线类型 | 行为 |
|---------|------|
| **Clamped Auto** | 默认：自动设置曲线使其平滑通过 |
| **Auto** | 向后兼容的 Clamped Auto，建议使用 Clamped Auto |
| **Free Smooth** | 自由设置切线，左右切线共线保证平滑 |
| **Flat** | 切线为水平 |
| **Broken - Free** | 自由设置单独的左右切线 |
| **Broken - Linear** | 线性过渡到下一个关键帧 |
| **Broken - Constant** | 保留值直到下一个关键帧 |

操作：右键关键帧显示切线选项，`Ctrl + A` 一次性选择所有关键帧

### 9.5 动画片段设置

| 设置 | 用途 |
|------|------|
| **Loop Time** | 是否在时间线末尾循环 |
| **Loop Pose** | 循环时添加关键帧以使首尾无缝连接 |
| **Loop Offset** | 动画片段从何处开始播放（0.5 = 一半处）|

### 9.6 静态 vs 动态片段

| 类型 | 行为 | 用途 |
|------|------|------|
| **静态** | 整个片段值不变 | 快速/响应性更改（GameObject 开关、组件开关）|
| **动态** | 整个片段值变化 | 在 Clip 中值之间混合（溶解切换、色调偏移）|

> ⚠️ 即使静态片段的关键帧完全相同，也建议每个片段**至少包含两个关键帧**，以确保片段具有实际持续时间

### 9.7 动画类型分类

#### GameObject 动画
- **GameObject Toggle**：VRChat 中最常见，灵活切换子节点分支（可能不是最高效）
- **Transform Animations**：X/Y/Z 方向可分别动画

#### Component 动画
- **Component Toggle**：某些组件可独立切换
- **Component Properties**：包括
  - 面部表情 Mesh 混合形变值
  - 约束权重、源权重、偏移、冻结轴
  - 物理 Rigidbody/Joint 弹簧/阻力/质量/驱动/限制
  - 粒子系统发射率、寿命、速度

#### Material 动画
- **Material Properties**：
  - 颜色/色调/饱和度/亮度
  - Decals 动画纹理
  - 发光/闪烁/边缘照明
  - 溶解/细分效果
  - **Poiyomi Shader 用户**：需为 Animation 标记属性
- **Material Swaps**：
  - 完全替换网格的材质槽
  - 是**对象引用曲线**（针对特定 GUID 资产）
  - **不能混合**，总是恒定切线

---

## 相关页面

[[avatar-modding-guide.md]] · [[lac-avatar-compressor.md]] · [[ma-component-cards.md]] · [[ma2bt.md]] · [[modular-avatar-tutorials-detailed.md]] · [[playable-layers.md]]
