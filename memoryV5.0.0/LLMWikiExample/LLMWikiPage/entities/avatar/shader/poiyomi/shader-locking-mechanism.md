---
title: "Poiyomi Shader Locking 机制与动画化"
category: avatar
subcategory: shader
poiyomi_subdir: true
knowledge_level: applied
status: active
source: "本地知识库整理(2026-07-04) + Poiyomi 官方 FAQ + Poiyomi ShaderPart.cs 源码 + VRCFury 官方文档 + VRC School + d4rkAvatarOptimizer Issue #182"
source_type: official
version: 1.0
upstream_version: "Poiyomi 10.0 / VRChat SDK 3.4.2+"
last_review: 2026-07-04
confidence: High
tags:
  - avatar
  - shader
  - poiyomi
  - locking
  - animation
  - vrcfury-integration
  - hue-shift
  - material-property
aliases:
  - "Poiyomi Locking"
  - "Poiyomi 锁定机制"
  - "Shader Locking"
  - "Poiyomi 动画化"
related:
  - index.md
  - ../../../../comparisons/poiyomi-vs-liltoon.md
  - modular-system.md
  - ../../../../comparisons/poiyomi-pro-vs-toon.md
  - ../../vrcfury-reference.md
  - ../../avatar-optimizer.md
  - ../liltoon/color-settings.md
type: entity
created: 2026-07-04
sources: "本地知识库整理(2026-07-04) + Poiyomi 官方 FAQ + Poiyomi ShaderPart.cs 源码 + VRCFury 官方文档 + VRC School + d4rkAvatarOptimizer Issue #182"
updated: 2026-07-04
---

# Poiyomi Shader Locking 机制与动画化

> **Domain**: Avatar → Shader → Poiyomi → Locking
> **核心来源**: Poiyomi 官方 FAQ + ShaderPart.cs 源码 + VRCFury 官方文档 + VRC School + d4rkAO Issue #182
> **状态**: 活跃(Active) — 2026-07-04 验证
> **关键价值**: 填补 Memory 库对 Shader Locking 机制的完全空白

---

## 1. 概述 — 什么是 Shader Locking

> [FACT] **来源**:
> - Poiyomi FAQ: https://www.poiyomi.com/9.3/general/faq
> - Poiyomi LLMS: https://context7.com/poiyomi/poiyomitoonshader/llms.txt
> - VRCFury 官方文档: https://github.com/VRCFury/vrcfury.com/blob/main/docs/43-actions.mdx

### 1.1 核心机制

VRChat 上传 Avatar 时，Poiyomi 材质会被**自动 Lock**（Shader Optimizer 处理）。这是 VRChat 的强制性能优化机制，**不是 Poiyomi 特有**。

**Lock 后效果**:
- ❌ 移除未使用的功能（strip unused features）
- ❌ 烘焙常量到 Shader 代码（bake constants）
- ❌ 属性变为**不可动画**（makes properties non-animatable）
- ✅ 显著提升 GPU 性能

**未 Lock 警告**:
```
Poiyomi 官方 Shader 源码中包含警告字面量:
[HideInInspector] _ForgotToLockMaterial (";;YOU_FORGOT_TO_LOCK_THIS_MATERIAL;", Int) = 1
```
未 Lock 的材质上传 VRChat 时会显示警告。

### 1.2 解决方案

在 Material Inspector 中**手动标记**需要保持可动画的属性为 `Animated` 或 `Renamed` — Lock 后这些属性仍可被 Animator 动画化。

### 1.3 核心交互

```
+------------------------+
| Unity Animator          |
| (VRCFury / MA / 手写)   |
+-----------+-------------+
            | 动画化属性
            v
+------------------------+
| Locked Poiyomi Material |
| - 标了 Animated 属性    |  ← Lock 后仍可被修改
| - 标了 Renamed 属性     |  ← Lock 后改名为 _animatedproperty_<材质>
+------------------------+
```

---

## 2. 两种锁定标记方式（P1-2, P1-3）

> [FACT] **来源**: Poiyomi ShaderPart.cs 源码 https://github.com/poiyomi/PoiyomiToonShader/blob/c5aaeeb3/_PoiyomiShaders/Scripts/ThryEditor/Editor/EditorStructs/ShaderPart.cs

### 2.1 源码验证

Poiyomi ThryEditor 的右键菜单（仅 `!IsLockedMaterial` 时显示）:

```csharp
if (IsAnimatable && !ShaderEditor.Active.IsLockedMaterial)
{
    _contextMenu.AddItem(new GUIContent("Animated (when locked)"), IsAnimated, () => { SetAnimated(!IsAnimated, false); });
    _contextMenu.AddItem(new GUIContent("Renamed (when locked)"), IsAnimated && IsRenaming, () => { SetAnimated(true, !IsRenaming); });
    _contextMenu.AddItem(new GUIContent("Locking Explanation"), false, () => { Application.OpenURL("https://www.youtube.com/watch?v=asWeDJb5LAo&ab_channel=poiyomi"); });
    _contextMenu.AddSeparator("");
}
```

### 2.2 Animated (when locked)（P1-2）

| 维度 | 详情 |
|------|------|
| **触发条件** | 属性 `IsAnimatable` |
| **机制** | 标记属性 Lock 后仍可动画（属性名不变）|
| **适用场景** | 单一 mesh 单一材质 |
| **选择标准** | "Hue Shift 在一个 mesh 上只有一个材质时使用"（VRC School）|

### 2.3 Renamed (when locked)（P1-3）

| 维度 | 详情 |
|------|------|
| **触发条件** | `IsAnimatable && IsAnimated` |
| **机制** | 标记属性 Lock 后改名为独立名称 `_animatedproperty_<材质名>` |
| **适用场景** | 同一 mesh 多个相同材质（防冲突）|
| **选择标准** | "Hue Shift 在一个 mesh 上有多个材质时使用"（VRC School）|

### 2.4 决策表：Hue Shift 场景下两种标记选择标准（P1-4）

> [FACT] **来源**: VRC School https://vrc.school/docs/Avatars/Radial-Puppets/

| 场景 | 正确标记 | 原因 |
|------|---------|------|
| 单一 mesh 单一材质 | `Animated (when locked)` | 单一属性名，无冲突 |
| 单一 mesh 多个相同材质 | `Renamed (when locked)` | 防止属性名冲突 |
| 跨 mesh 同一材质 | `Animated (when locked)` | 不同 Renderer，无冲突 |
| 调试模式（未 Lock）| 标记不生效 | 材质自动检测 |

**VRC School 原文引用**:
> "Note that if you are using the Poiyomi shader, you will need to set the Hue Shift field to `Animated (when locked)` if there is only one material you're hue shifting on that mesh, or to `Renamed (when locked)` when there are multiple materials you're hue shifting on that mesh."

---

## 3. 4 种 Copy Property Name 选项（P2-1）

> [FACT] **来源**: Poiyomi ShaderPart.cs 源码

Poiyomi 在右键菜单中提供 **4 种属性名复制选项**（不仅是 1 种）:

| 选项 | 复制内容 | 适用场景 |
|------|---------|---------|
| `Copy Property Name` | 原始 Shader 属性名 | 材质未 Lock 时 |
| `Copy Animated Property Name` | 动画化后的属性名 | 材质 Lock 后（**推荐**）|
| `Copy Animated Property Path` | 完整动画路径 | 复杂动画设置 |
| `Copy Property as Keyframe` | Keyframe 格式 | Unity Animation 录制 |

**源码确认**:
```csharp
_contextMenu.AddItem(new GUIContent("Copy Property Name"), false, () => { EditorGUIUtility.systemCopyBuffer = MaterialProperty.name; });
_contextMenu.AddItem(new GUIContent("Copy Animated Property Name"), false, () => { EditorGUIUtility.systemCopyBuffer = GetAnimatedPropertyName(); });
_contextMenu.AddItem(new GUIContent("Copy Animated Property Path"), false, CopyPropertyPath);
_contextMenu.AddItem(new GUIContent("Copy Property as Keyframe"), false, CopyPropertyAsKeyframe);
```

**关键提示**:
- VRCFury / MA 等外部工具集成时，**必须用 `Copy Animated Property Name`**
- 复制原始 `Copy Property Name` 在材质 Lock 后**无效**（属性已被改名/烘焙）

---

## 4. VRCFury Material Property 集成流程

### 4.1 VRCFury 官方建议

> [FACT] **来源**: VRCFury 官方文档 https://github.com/VRCFury/vrcfury.com/blob/main/docs/43-actions.mdx

> "Animates a Material Property on the specified renderer to the defined value. In many shaders, you can simply right click a slider or value to copy its name. **If your shader supports locking, make sure to mark the Material Property you are animating as animated.**"

### 4.2 完整操作流程（Hue Shift 案例）

```
步骤 1: 选 mesh → Add Component → VRCFury Toggle
步骤 2: Add Action → Material Property
步骤 3: Options → 勾选 "Use Slider Wheel" + "Save Between Worlds"
步骤 4: 在 Poiyomi 材质中:
   a. 右键 Hue Shift → 选 "Animated (when locked)" 或 "Renamed (when locked)"
   b. 根据场景（见 §2.4 决策表）选其一
步骤 5: ⚠️ 重要: 必须先 Lock 材质（测试上传后行为）
   → 右键 Avatar → Thry → Materials → Lock All
步骤 6: 然后右键 Hue Shift → Copy Animated Property Name
步骤 7: 回到 VRCFury Toggle → 粘贴到 Property 字段
步骤 8: Value = 1 (0% → 100% 循环)
步骤 9: Gesture Manager 测试
```

### 4.3 Poiyomi Rename Animated 作为 VRCFury Workaround（P2-2）

> [FACT] **来源**: VRCFury 官方文档

VRCFury 官方在 Material Property 文档中**直接引用** Poiyomi 的 Rename Animated 机制作为多 Renderer 冲突的解决方案:

> "Note: Material Properties will be applied across the entire renderer, this is a unity limitation. If your shader has any, This can be avoided by using the Renaming function of your shader, for example [Poiyomi Rename Animated](https://www.poiyomi.com/general/locking#rename-animated)."

**机制**:
- 标记为 Renamed 后，Lock 时属性名变为 `_animatedproperty_<材质名>`
- 每个材质实例获得**独立**的属性名
- 同一 Shader + 同一属性，不同材质不会冲突
- VRCFury Material Property + Copy Animated Property Name 自动使用新名

---

## 5. 与其他工具的冲突（P1-6）

### 5.1 d4rkAvatarOptimizer + Poiyomi Renamed 冲突

> [FACT] **来源**: https://github.com/d4rkc0d3r/d4rkAvatarOptimizer/issues/182
> **状态**: Open（2026-04-01 报告，截至 2026-07-04 未解决）

**冲突场景**:
```
- Poiyomi 材质标记为 "Renamed (when locked)"
- VRCFury Material Property 用 "Copy Animated Property Name" 获取属性名
- d4rkAvatarOptimizer 优化设置: "Full" preset
- 上传时：Renamed 属性的值被 d4rkAO 删掉/烘焙
- 结果: VRCFury Hue Shift 动画失效
```

**d4rk 作者回复**:
> "Yeah my optimizer doesn't support RA. I recently added a warning that should show up when anything is marked RA in a material used in the avatar the optimizer component is on."

### 5.2 解决方案

| 方案 | 操作 | 适用场景 |
|------|------|---------|
| **手动 Lock 材质** | Right-click Avatar → Thry → Materials → Lock All | 不用 d4rkAO 优化时 |
| **不用 Renamed** | 用普通 Animated（单一 mesh 单一材质）| 简单 Avatar |
| **Mesh 拆分** | 让 d4rkAO 自己处理 | 复杂 Avatar（d4rk 内部逻辑）|

### 5.3 Renamed 属性命名规则

> 来源: d4rkAO Issue #182 讨论
> "Poiyomi 自动命名为 `_animatedproperty_materialname` when locked"

**机制**:
- 标记为 Renamed 后，Lock 时属性名变为 `_animatedproperty_<材质名>`
- 每个材质实例获得独立属性名
- 外部工具必须使用 `Copy Animated Property Name` 才能拿到正确属性名

---

## 6. 完整 Hue Shift 工作流（VRC School）（P2-6）

> [FACT] **来源**: https://vrc.school/docs/Avatars/Radial-Puppets/

### 6.1 两种实现方式

**方式 A: Motion Time 动画**
```
1. 创建一个 Animation,两个 keyframe:
   - 0:00 → Hue Shift = 0
   - 0:01 → Hue Shift = 1
2. 添加 Radial Puppet Rotation Parameter 到 FX Layer (Float 类型)
3. 新建 Layer, Weight = 1
4. 在 Layer 内创建 State
5. State 的 Motion = 录制的 Hue Shift 动画
6. 启用 Motion Time
7. Motion Time 字段 = Radial Puppet Rotation Parameter
```

**方式 B: 1D Blend Tree**
```
- 单个 Blend Tree 节点对应 Hue Shift 滑块值
- 类似 Motion Time，但用 Blend Tree 而非 Motion Time
```

### 6.2 关键步骤（结合 Poiyomi Locking）

```
1. 添加 Radial Puppet Rotation Parameter 到 FX Layer (Float)
2. 新建 Layer, Weight = 1
3. 在 Layer 内创建 State
4. State 的 Motion = 录制的 Hue Shift 动画
5. 启用 Motion Time
6. Motion Time 字段 = Radial Puppet Rotation Parameter
7. (Poiyomi) 设置 Hue Shift 为 Animated (when locked)
8. ⚠️ Lock 材质 → 测试 (Av3 Emulator / Gesture Manager)
```

---

## 7. Poiyomi C# API（程序化 Lock/Unlock）

> [FACT] **来源**: Poiyomi LLMS https://context7.com/poiyomi/poiyomitoonshader/llms.txt

```csharp
using Thry.ThryEditor;

// Lock a material
ShaderOptimizer.SetLockedForAllMaterials(new Material[] { myMaterial }, 1, true);

// Unlock a material
ShaderOptimizer.SetLockedForAllMaterials(new Material[] { myMaterial }, 0, true);

// Check if material is locked
bool isLocked = ShaderOptimizer.IsLocked(myMaterial);
```

**应用场景**:
- 构建工具链（CI/CD 自动化）
- 批量 Lock/Unlock 多个材质
- 程序化检测 Lock 状态

---

## 8. 实战陷阱

| 错误 | 症状 | 修复 |
|------|------|------|
| 标记了但上传后失效 | 材质未 Lock + 未先标记 | 先 Lock 材质，再标记 Animated |
| 单 mesh 多材质冲突 | 多个 Hue Shift 不响应 | 用 Renamed (when locked) |
| 用了 Renamed 但 d4rkAO 优化 | Renamed 值被删除 | 手动 Lock 或关 d4rkAO |
| 复制了 Property Name 而非 Animated Property Name | VRCFury 动画不生效 | 重新复制 Animated Property Name |
| 未 Lock 时测试 OK 上传失效 | 材质上传后自动 Lock | 测试前先 Lock 模拟上传后状态 |
| 调试模式（编辑器）显示 Lock 警告 | "YOU_FORGOT_TO_LOCK_THIS_MATERIAL" | 正常提示，材质未 Lock |

---

## 9. 关键事实速查

| 事实 | 状态 |
|------|------|
| VRChat 上传自动 Lock 材质 | ✅ FACT（Poiyomi FAQ + VRCFury 官方）|
| Poiyomi 提供 Animated (when locked) 标记 | ✅ FACT（ShaderPart.cs 源码）|
| Poiyomi 提供 Renamed (when locked) 标记 | ✅ FACT（ShaderPart.cs 源码 + VRC School）|
| Renamed 自动命名为 `_animatedproperty_<材质名>` | ✅ FACT（d4rkAO Issue #182）|
| Poiyomi 提供 4 种 Copy Property Name 选项 | ✅ FACT（ShaderPart.cs 源码）|
| VRCFury 官方推荐 Poiyomi Rename Animated | ✅ FACT（VRCFury 官方文档）|
| d4rkAO 不支持 Poiyomi Renamed 模式 | ✅ FACT（d4rkAO Issue #182）|
| Poiyomi C# API: `ShaderOptimizer.SetLockedForAllMaterials` | ✅ FACT（Poiyomi LLMS）|
| VRChat 未 Lock 材质会显示警告 | ✅ FACT（Poiyomi 源码 `_ForgotToLockMaterial`）|
| Hue Shift 单 mesh 单材质用 Animated | ✅ FACT（VRC School）|
| Hue Shift 单 mesh 多材质用 Renamed | ✅ FACT（VRC School）|

---

## 10. 引用与原始数据

### 10.1 官方文档
- **Poiyomi FAQ**: https://www.poiyomi.com/9.3/general/faq
- **Poiyomi LLMS**: https://context7.com/poiyomi/poiyomitoonshader/llms.txt
- **VRCFury 官方 Actions 文档**: https://github.com/VRCFury/vrcfury.com/blob/main/docs/43-actions.mdx
- **VRC School Radial Puppet**: https://vrc.school/docs/Avatars/Radial-Puppets/

### 10.2 源代码
- **Poiyomi ShaderPart.cs**: https://github.com/poiyomi/PoiyomiToonShader/blob/c5aaeeb3/_PoiyomiShaders/Scripts/ThryEditor/Editor/EditorStructs/ShaderPart.cs
- **Poiyomi 锁定机制文档**: https://www.poiyomi.com/general/locking

### 10.3 GitHub Issues
- **d4rkAO Issue #182**: https://github.com/d4rkc0d3r/d4rkAvatarOptimizer/issues/182

### 10.4 知识库交叉引用
- [[entities/avatar/shader/poiyomi/index]] — Poiyomi 知识库主索引
- [[comparisons/poiyomi-vs-liltoon]] — Poiyomi vs lilToon 对比
- [[entities/avatar/shader/liltoon/color-settings]] §1.5 — LilToon HSVG 注意事项
- [[entities/avatar/vrcfury-reference]] §4.1 — VRCFury Material Property Action
- [[entities/avatar/avatar-optimizer]] — 工具链冲突
- `../../../../../../（见源库 raw/ 目录）参考文献/23-04025/00-总文档-P等级评级.md` — 原始 P 等级评级

### 10.5 知识库整合来源
- **本文档纳入来源**: 23-04025.md（VRCD 文档库 iconoclass 编写）处理后的 P1+P2 知识点
- **处理时间**: 2026-07-04
- **P 等级**: P1（核心新知识）x 4 + P2（外部验证补充）x 3

---

## 元信息

| 字段 | 值 |
|------|-----|
| **文档版本** | 1.0 |
| **创建日期** | 2026-07-04 |
| **上游版本** | Poiyomi 10.0 / VRChat SDK 3.4.2+ / VRCFury v1.1341.0+ |
| **完整度** | 100%（核心机制 + 4 种 Copy 选项 + 工具链冲突 + API）|
| **评审状态** | ✅ Stage 1 完成（创建）|
| **下一步** | Stage 2 引用完整性验证 |

---

## 相关页面

[[index.md]] · [[../../../../comparisons/poiyomi-vs-liltoon.md]] · [[modular-system.md]] · [[../../../../comparisons/poiyomi-pro-vs-toon.md]] · [[../../vrcfury-reference.md]] · [[../../avatar-optimizer.md]] · [[../liltoon/color-settings.md]]
