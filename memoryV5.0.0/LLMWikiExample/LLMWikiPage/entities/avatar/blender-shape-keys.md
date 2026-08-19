---
title: Blender 形态键 (Shape Keys) → Unity BlendShapes 工作流
category: avatar
subcategory: workflow
knowledge_level: applied
status: active
source: 本地知识库整理 + Unity 官方手册 + Blender 社区
source_type: community
version: 1.0
last_review: 2026-07-04
confidence: High
tags:
  - avatar
  - blender
  - shapes-keys
  - blendshapes
  - workflow
  - fbx
  - modding
aliases:
  - "Blender Shape Keys"
  - "Blender BlendShapes"
  - "形态键"
  - "Blender 形态键"
  - "Blender → Unity BlendShapes"
related:
  - avatar-modding-guide.md
  - modular-avatar.md
  - ../platform/unity-recorder.md
source_origin: 从 07-16790.md §Q "blender里的形态键修改后在Unity里不显示" 提取 (P1)
type: entity
created: 2026-07-04
sources: 本地知识库整理 + Unity 官方手册 + Blender 社区
updated: 2026-07-04
---

# Blender 形态键 (Shape Keys) → Unity BlendShapes 工作流

> 将 Blender 中的形态键 (Shape Keys / Morph Targets) 正确导出到 Unity SkinnedMeshRenderer 的 BlendShapes。
>
> **源 QA**: 07-16790.md §"blender 里的形态键修改后在 Unity 里不显示"

---

## 0. 速查表

| 症状 | 根本原因 |
|------|---------|
| Unity 中看不到 BlendShapes | 用了 `MeshRenderer` 而非 `SkinnedMeshRenderer` |
| 形态键没导入 | FBX 导出未勾选 "Apply Modifiers" / "Animation" |
| 修改后不显示 | 未重新 Import/Reimport FBX |
| Unity 6 中丢失 | 可能是 Mesh 实例缓存问题(删除并重新拖入) |

---

## 1. 关键认知(必读)

> ⚠️ **原始 QA 答案的误区**
>
> 源文档原始回答是 "Blender 中对形态键的使用不会保存,要在 Unity 中使用"。
> **这不准确**。Blender 中形态键的 **值** 在导出时确实不会保留(0-100 范围的具体值),但形态键的 **定义** 会通过 FBX 的 "BlendShapes" 通道传入 Unity。
>
> 真正的问题是 **FBX 导出设置** 和 **Unity 导入设置** 不当,导致形态键根本没进 FBX。

### 1.1 名词对应关系

| Blender | FBX 格式 | Unity |
|---------|---------|-------|
| Shape Keys (形态键) | BlendShapes (BlendShapeChannel) | BlendShapes (SkinnedMeshRenderer.blendShape*) |
| Shape Key 值 (0-1) | BlendShape 权重 (0-100) | BlendShape Weight (0-100) |

---

## 2. 完整工作流

### 步骤 1:Blender 中准备形态键

1. 在 Blender 中选择 Mesh 对象
2. 进入 `Object Data Properties` (绿色三角图标)
3. 展开 `Shape Keys` 面板
4. 添加基础键 `Basis` (自动存在)
5. 添加其他形态键(如 `Smile`、`Blink` 等)
6. 通过调节 `Value` 滑块 (0-1) 编辑形态

> 💡 **建议**: 在编辑前先点 `Basis` 的小锁图标 🔒,防止误编辑基础形态

### 步骤 2:Blender FBX 导出设置 ⚠️ 关键

**菜单**: `File` → `Export` → `FBX (.fbx)`

#### 2.1 必勾选项

| 路径 | 选项 | 说明 |
|------|------|------|
| `Main` | ☑ Apply Scalings | FBX 单位缩放 |
| `Main` | ☑ Apply Modifiers | ⚠️ **关键**: 应用修改器,否则形态键不导出 |
| `Geometry` | ☑ Apply Transform | 应用变换 |
| `Geometry` | ☑ **Smoothing** | "Face" 或 "Edge" |
| `Geometry` | ☑ **Blend Shapes** | ⚠️ **关键**: 导出形态键 |
| `Armature` | ☑ Add Leaf Bones | 视情况 |

#### 2.2 容易踩的坑

- ❌ **不要**勾选 "Add Leaf Bones"(会导致 SkinnedMesh 多出空骨骼)
- ❌ **不要**在形态键未确认前就 Export Basis
- ✅ 如果有多个 Object,需要逐个设置(没有"全局"应用)

### 步骤 3:Unity FBX 导入设置 ⚠️ 关键

1. 将 FBX 拖入 Unity `Assets/`
2. 在 Project 窗口中**点击** FBX 文件
3. Inspector 中展开 `Model` 选项卡

#### 3.1 必勾选项

| 选项 | 推荐值 | 说明 |
|------|--------|------|
| **Import BlendShapes** | ☑️ | ⚠️ **关键**: 不勾则 Unity 不导入形态键 |
| **Normals** | Import | 法线计算 |
| **Smoothing Angle** | 60° | 光滑角度 |
| **Read/Write Enabled** | ☑️ (运行时修改) | 视需求 |
| **Optimize Mesh** | ☑️ | 优化 |
| **Import Visibility** | ☑️ | 视需求 |
| **Import Cameras** | ☐ | 一般不需要 |
| **Import Lights** | ☐ | 一般不需要 |

#### 3.2 Rig 选项卡

| 选项 | 推荐值 |
|------|--------|
| **Animation Type** | `Generic` / `Humanoid` / `Legacy` (视情况) |
| **Rig** | 不要选 `None` ❌ |

> ⚠️ **关键**: 如果 Rig 选 `None`,即使勾了 Import BlendShapes 也可能不显示

### 步骤 4:Avatar 应用 ⚠️ 关键

#### 4.1 组件类型

- ❌ **不要**用 `MeshRenderer` (普通渲染器,不支持 BlendShapes)
- ✅ **必须**用 `SkinnedMeshRenderer` (蒙皮渲染器)

#### 4.2 检查 BlendShapes 是否存在

1. 在 Hierarchy 中选择带 SkinnedMeshRenderer 的对象
2. Inspector → `SkinnedMeshRenderer` → 展开 `BlendShapes` 面板
3. 看到所有形态键列表 (如 `Smile`, `Blink`) → **成功**
4. 没有列表 → 检查上述所有步骤

#### 4.3 在 Animation 中使用

1. `Window` → `Animation` → `Animation`
2. 选择要添加动画的对象
3. `Add Property` → `[对象]` → `Skinned Mesh Renderer` → 选择形态键
4. 调节权重,设置关键帧

---

## 3. 代码访问 (C#)

```csharp
using UnityEngine;

public class BlendShapeController : MonoBehaviour
{
    [SerializeField] private SkinnedMeshRenderer skinnedMesh;
    
    void Start()
    {
        // 运行时设置
        if (skinnedMesh != null)
        {
            // 设置 "Smile" 形态键为 50% 权重
            int smileIndex = skinnedMesh.sharedMesh.GetBlendShapeIndex("Smile");
            if (smileIndex >= 0)
            {
                skinnedMesh.SetBlendShapeWeight(smileIndex, 50f);
            }
        }
    }
    
    // 获取所有 BlendShape 数量
    public int GetBlendShapeCount()
    {
        return skinnedMesh != null ? skinnedMesh.sharedMesh.blendShapeCount : 0;
    }
}
```

> **Udon 限制**: 在 Udon (U#) 中访问 BlendShapes 受限,需检查 API 白名单

---

## 4. 已知问题与解决方案

### 4.1 Unity 6 中丢失部分形态键

**症状**: Unity 6 重新导入后,某些形态键(尤其是 Skinned Mesh 上的)丢失

**原因** (来自 Unity 论坛):
- 场景中的旧实例未完全更新
- FBX 重新导入后,场景实例缓存未刷新

**解决方案**:
1. 从场景中**完全删除**该模型
2. 重新从 Project 窗口拖入场景
3. 或在空项目中测试,确认 FBX 本身正确

### 4.2 多个 Object 中只有一个显示

**症状**: 多个 Mesh 对象,只有一个显示 BlendShapes

**原因**: 其他对象的 FBX 导出未勾选 "Blend Shapes"

**解决方案**: 重新导出所有对象的 FBX

### 4.3 形态键值不对(不是 0-1)

**症状**: 导入后 BlendShape Weight 显示异常

**原因**: Blender 形态键的 0-1 值在 FBX 中转换为 0-100 权重。Blender 中的 0.5 → Unity 中的 50

**注意**: 形态键的**值**通常不会从 Blender 传到 Unity(只传定义);具体值需在 Unity 中或代码中设置

### 4.4 推荐使用专用导出插件

> 💡 **建议**: 原生 Blender FBX 导出器对游戏引擎优化不佳
>
> 推荐插件:
> - **Auto Rig Pro** (Blender 插件,商业) — 优化 FBX 导出
> - **Better FBX Importer/Exporter** (Blender 插件,免费) — 更可控

---

## 4.5 VRChat Humanoid 约束 ⚠️ 关键(2026-07-04 来自 22-7f6ff)

> [FACT] **来源**: https://creators.vrchat.com/avatars/rig-requirements/ + http://vrchat.wikidot.com/tutorial:avatars:unity-upload-tutorial
> **优先级**: 🔴 高(直接决定 Avatar 是否可上传)
> **P 等级**: P1

### VRChat 必需骨骼清单

| 骨骼 | Body 选项卡位置 | 必需性 | 备注 |
|------|---------------|-------|------|
| **Pelvis** | Body | ✅ 必需 | - |
| **Spine** | Body | ✅ 必需 | - |
| **Chest** | Body | ✅ **VRChat 必需**(Mecanim 可选) | ⚠️ |
| **Neck** | Head | ✅ **VRChat 必需**(Mecanim 可选) | ⚠️ |
| **Left/Right Shoulders** | Left/Right Arm | ✅ 必需 | - |
| **Head** | Head | ✅ 必需 | - |
| **Left/Right Hands** | Left/Right Arm | ✅ 必需 | - |
| **Left/Right Feet** | Left/Right Leg | ✅ 必需 | - |
| **Upper Chest** | Body | ❌ **不要映射** | ⚠️ 会导致 IK 问题 |
| **Toes** | Left/Right Leg | ⚪ 可选 | - |
| **Jaw** | Head | ⚪ 可选 / **不应分配** | ⚠️ 常见错误 |

### 父级关系(IK 工作前提)

> [FACT] **官方明确**: "your shoulder bones... must be direct children of your chest bone... Also, the neck bone... must also be a direct child of the Chest."

- **Shoulders** 必须是 **Chest** 的直接子级
- **Neck** 必须是 **Chest** 的直接子级

### Jaw 不应分配 ⚠️ 关键

> [FACT] **VRChat Wiki 明确**: "Assigning a eye bone to the jaw bone... The first problem can easily be fixed by clearing the jaw bone assignment."

- 许多模型**会自动**将错误骨骼分配给 Jaw(常见错误!)
- **Jaw 在 VRChat 是可选的**(viseme 可用 jaw-flap bone 或 blendshapes)
- 如分配错误: **必须清除** Jaw 分配
- 清除方法: 点击 "jaw" 旁边的绿色圆圈 → 按 **Delete** 键

### 验证步骤

1. 检查所有必需骨骼已映射
2. 检查 Upper Chest **未映射**
3. 检查 Jaw **未分配**
4. 检查 Neck/Shoulders 是 Chest 的直接子级
5. **Done** → **Apply**

详见: `[[entities/avatar/avatar-upload-getting-started]] §4`

---

## 5. Avatar Creator 速查清单

```
□ FBX 导出时勾选 Apply Modifiers?
□ FBX 导出时勾选 Blend Shapes?
□ Unity 导入时勾选 Import BlendShapes?
□ Rig 不是 None?
□ 使用 SkinnedMeshRenderer 而非 MeshRenderer?
□ 重新导入后从 Project 重新拖入场景(Unity 6)?
```

---

## 6. 相关文档

- [[entities/avatar/avatar-modding-guide]] — Avatar 改模指南
- [[entities/avatar/modular-avatar]] — Modular Avatar (MA)
- [[entities/platform/unity-recorder]] — Unity Recorder(用于截图 BlendShape 效果)
- `（见源库 sources/ 目录）vrcfury-reference.md` — VRCFury(可自动处理 Merge SkinnedMesh)

---

## 7. 引用

- **源 QA**: https://docs.vrcd.org.cn/books/16790
- **Unity 官方手册 (BlendShapes)**: https://docs.unity3d.com/6000.4/Documentation/Manual/BlendShapes.html
- **Unity 官方 (Scripting BlendShapes)**: https://docs.unity3d.com/ScriptReference/SkinnedMeshRenderer.html
- **Blender 手册 (Shape Keys)**: https://docs.blender.org/manual/en/latest/animation/shape_keys/index.html
- **Unity Forum (Shape Keys Not Loading)**: https://discussions.unity.com/t/shape-keys-are-not-importing-in-unity-from-blender/1527440
- **Blender Artists (Shape Key Animation to FBX)**: https://blenderartists.org/t/exporting-shapekey-animation-to-fbx-for-use-in-unity/1626417

---

## 相关页面

[[avatar-modding-guide.md]] · [[modular-avatar.md]] · [[../platform/unity-recorder.md]]
