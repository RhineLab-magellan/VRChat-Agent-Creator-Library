---
title: Avatar 最佳化实操指南
category: avatar

knowledge_level: applied
status: active

tags:
  - avatar
  - animator
  - physbone

aliases:
  - "Avatar Optimization"
  - "Avatar 优化"
  - 性能优化
  - "Avatar Performance"

related:
  - performance-rank.md
  - ndmf-tools.md
  - playable-layers.md
  - vrc-constraints.md
  - vrcfury-reference.md
  - modular-avatar.md
  - ../world/performance-guide.md
  - avatar-size-limits.md
  - avatar-audio-optimization.md

source: 本地知识库整理
source_type: community
version: 1.3
last_review: 2026-07-04
confidence: High
type: entity
created: 2026-07-04
sources: 本地知识库整理
updated: 2026-07-04
---
# Avatar 最佳化实操指南

> 来源: Kuriko Avatar 最佳化笔记（HackMD）
> 置信度: High | 更新: 2026-06-05

---

## 事前选择 > 事后优化（2026-07-04 新增）

> **来源**: VRCD 中文文档 https://docs.vrcd.org.cn/books/vrchat-pc-android
> **核心思想**: 在优化之前先选择合适的资源，事半功倍

### 适合 Android 的衣物 5 大特征

| # | 特征 | 原因 |
|---|------|------|
| 1 | **外观简洁** | 低面数，减少减面负担 |
| 2 | **较少 PhysBone** | 原始 PB 多（如裙子）→ Android 难保留 |
| 3 | **大面积覆盖上半身** | 女性模型胸部面数最多 → 被覆盖后可移除 |
| 4 | **未使用特殊渲染属性** | 切换 Shader 后能保持外观 |
| 5 | **只使用一个材质球和基础贴图** | 越少材质球/贴图 → 越易达到好评级 |

### 实践策略

- ✅ 选择一件**最适合**的衣物
- ❌ **移除**其他多余衣物
- ⚠️ 多件衣物**面数叠加**必然超出 Android 面数限制（Poor < 20,000）

### 与现有优化的关系

```
[事前选择] → 减少优化负担
   ├─ 衣物选择（本文档）
   ├─ 插件选择（见 android-development.md §Android 插件兼容性原则）
   └─ Shader 选择（见 android-development.md §Avatar 着色器限制）
      ↓
[事后优化] → 进一步精修
   ├─ AAO Trace And Optimize
   ├─ Avatar Compressor (LAC)
   └─ Meshia 减面
```

**核心原则**: 事前选对资源，比事后优化 10 次更有效

---

## 速查懒人包

```
1. 复制 Avatar 的 FX Layer、Expression Menu、Expression Parameter
2. 删除所有跟开关物件相关的设定（保留表情 Blend Shape）
3. 复制 Avatar 本身
4. 导入 **AAO + Avatar Compressor (LAC) + Meshia** Mesh Simplification
5. 运行 AAO Trace And Optimize（自动 + LAC 自动纹理压缩 + Meshia 减面）
6. 面数还太多 → Remove Mesh By BlendShape / Meshia 减面 / 脱掉
7. 把 FX、参数、选单塞回复制出来的 Avatar
```

> 💡 **2026 推荐做法**：在第 4 步加入 Avatar Compressor (LAC) 自动处理纹理压缩，详见 `lac-avatar-compressor.md`

---

## AAO Trace And Optimize（自动最佳化）

> 📚 **完整 AAO 组件知识库**: `avatar-optimizer.md` — 12 个组件详解 + API + 优化方向矩阵
> 文档: https://vpm.anatawa12.com/avatar-optimizer/en/docs/reference/trace-and-optimize/

### 自动处理项目

- ✅ 合并 Skinned Mesh
- ✅ 固定目前状态的 Blend Shape 并删除
- ✅ 删除未使用物件
- ✅ 删除没有实质作用的 end 骨
- ✅ 删除没有作用的 PhysBone
- ✅ Animator 最佳化

### 使用方法

```
1. 选择 Avatar 物件本身
2. Inspector > Add Component > AAO Trace And Optimize
3. 完成
```

### 注意事项

> ⚠️ 若 Animator 中有开关物件的 Animation，会跳过 Skinned Mesh 合并

---

## Light（光源）

**规则：没事不要放。**

| Excellent | Good | Medium | Poor |
|-----------|------|--------|------|
| 0 | 0 | 0 | 1 |

如果嫌 Avatar 太暗，调整 Shader，而非加灯。

---

## Particle System（粒子系统）

**无法最佳化，只能控制数量。**

| 指标 | Excellent | Good | Medium | Poor |
|------|-----------|------|--------|------|
| 粒子系统数 | 0 | 4 | 8 | 16 |
| 粒子总数量 | 0 | 300 | 1,000 | 2,500 |
| 粒子系统面数 | 0 | 1,000 | 2,000 | 5,000 |

**控制要点**：
- Max Particles 调整粒子最大数量
- Poor 极限是 2,500，超出会 Very Poor
- 粒子轨迹/碰撞根据需求开关
- Trail Renderer 会吃掉 2 个材质球

> 💡 **用户端 Particle System Limits 系统**(2026-06-30 新增)
>
> 上述指标是**上传时** Performance Rank 评估,而 VRChat 客户端还有**独立的**运行时限制系统:
> - 配置: `config.json` 在 `%AppData%\..\LocalLow\VRChat\vrchat\`
> - 11 个变量(数量、网格、碰撞、trail penalty)
> - **Quest 端默认启用且不可禁用**
> - Penalty 计算公式**【推断】** 来自官方变量描述,实际实现可能略有差异
> - 详见 [[entities/avatar/avatar-particle-system-limits]] §4.2 Penalty 系统

---

## Texture Memory（材质贴图内存）

### 关键原则

> 💡 无论原档是 PNG/JPG/PSD，解析度多高多低，**最终 VRAM 使用量只依 Unity 内设定决定**

### 降分辨率技巧

| 物件类型 | 推荐分辨率 |
|----------|-----------|
| 主贴图 | 2048 以下 |
| 鞋子等不明显物件 | 512 附近 |
| 遮罩贴图 | 128 也行 |
| MatCap 部位 | 32 级别 |

### 压缩建议

| 压缩方式 | 适用场景 | 注意 |
|----------|----------|------|
| Normal | 普通贴图 | 对 R 频道破坏大 |
| High Quality | 皮肤等贴图 | 无 A 频道的贴图用 High 会 VRAM 加倍 |
| Crunch | 下载大小优化 | VRAM 不变，不推荐 |

> ⚠️ Crunch Compression：下载变小，但贴图破坏 + 解压资源 + VRAM 不变

### 全自动纹理压缩方案（2026 推荐）⭐NEW

> 工具: **Avatar Compressor (LAC)** — 详见 `lac-avatar-compressor.md`
> VPM: `https://vpm.limitex.dev/`

**为什么不手动设置**:
- 每张贴图都要设 Max Size + Compression + 平台格式（繁琐）
- 容易把 Normal Map 当 Albedo 压缩（破坏法线）
- 共享贴图容易重复处理
- 无 VRAM 预估（盲调）

**LAC 自动化**:
- 4 种分析策略（Fast / HighAccuracy / Perceptual / Combined）
- 5 个内置预设（开箱即用）
- 自动平台格式选择（PC: BC7 / Quest: ASTC_4x4）
- 自动贴图类型识别（Normal/Emission/Metallic 等）
- 共享贴图识别
- 编译前 VRAM 预估

**与 AAO 协作顺序**: LAC → AAO（详见 `ndmf-tools.md`）

### VRAM 与分辨率关系

- 每降一级分辨率 → VRAM 变为 **1/4**
- 每升一级分辨率 → VRAM **×4**

**推荐目标**: 单 Avatar VRAM < 40MB（Excellent），活动用 < 40MB（Excellent）

> 💡 **2026 推荐做法**：使用 Avatar Compressor (LAC) 自动达成上述目标，无需手动逐张设置。详见 `lac-avatar-compressor.md`

### 6 个核心贴图导入设置 ⭐NEW 2026-07-04

> **来源**: VRCD 18-vrchat-CI4 §贴图方面 + VRChat 官方 Dev Update (2024-03-14)
> **背景**: 在 §Texture Memory 上方调整导入设置是减小贴图体积的最直接方法

#### Alpha Is Transparency（Alpha 是透明的）

- **作用**: 允许导入贴图中透明部分
- **配合**: 与非 Opaque（透明）渲染模式材质球配合使用
- **何时关闭**:
  - 物体未包含透明部分
  - 用 PhotoShop 将贴图透明部分填充为纯色（黑/白），从根源减小大小
- ⚠️ **重要警告**: 有些着色器（如 **liltoon**）使用 **Alpha 蒙版**制作半透明 → 遇到此情况应**修改贴图对应的材质球设置**而非贴图本身

#### Generate Mipmaps（生成 Mipmap） ⭐CORRECTED 2026-07-04

> **数据纠正**:原"+33% 导入大小" 来自 18-vrchat-CI4 测试数据(7.1MB → 9.5MB)。
> lilAvatarUtils 官方文档(lilxyzw 2025)明确指出 **"turning it off can reduce VRAM by 33%"**。
> 两者口径不同:CI4 测的是**导入大小**,lilAvatarUtils 测的是 **VRAM 占用**。
> 两者**不矛盾**但**应用场景不同**。

- **作用**: 生成一系列逐渐降低分辨率的贴图，根据距离选择合适分辨率
- **代价(VRAM 口径)**:关闭 MipMap 可减少 **约 33% VRAM 占用**(lilAvatarUtils 官方)
- **代价(导入大小口径)**:开启 MipMap 增加约 +33% 导入大小(7.1MB → 9.5MB,VRCD 18-vrchat-CI4 测试)
- **何时关闭**:
  - 贴图无复杂纹理/渐变
  - 不在意模型的远处视觉效果
  - Quest 平台性能要求严苛
- ⚠️ **例外**:Vertex Shader 中使用的贴图(如 outline 遮罩)需**保持 MipMap**

#### Read/Write Enabled（读写启用）⭐NEW 2026-07-04

> 来源:lilAvatarUtils 官方文档 + Unity 官方 Texture Import Settings
> 来源 URL:https://lilxyzw.github.io/lilAvatarUtils/en_US/docs/TexturesGUI.html

- **作用**:允许脚本访问贴图数据(ReadPixels/SetPixels/GetPixels)
- **代价**:开启 = **RAM 占用翻倍**(lilAvatarUtils 官方:"Copying textures for script access doubles the RAM consumption")
- **何时关闭**:
  - 贴图不需要在脚本中读取
  - 99% 的 Avatar 贴图都应该关闭(纯显示用途)
- **何时开启**:
  - 需要运行时修改贴图(如 Paint、笔刷、截图)
  - 极少数特殊 Shader 需要 CPU 端采样

#### Mip Streaming（Mip 流式加载）⭐NEW 2026-07-04

> 来源:lilAvatarUtils 官方文档(实验性功能)

- **作用**:根据相机位置按需加载 Mipmap(仅加载当前可见的 mip 层级)
- **优势**:降低 VRAM 消耗(适合高分辨率贴图)
- **状态**:实验性,需 Unity 2021.2+ 支持

#### Max Size（最大尺寸）— 影响 90% 视觉效果

- **作用**: 将贴图分辨率限制在设定值内
- **行为**: 原始 < 设定值时按原始分辨率导入
- **量化数据**（Normal Quality 压缩）:

| 设定值 | 原始 | 压缩后 |
|--------|------|--------|
| 4096 | 7.8MB | **8.0MB** |
| 2048 | 7.8MB | **2.0MB** |
| 1024 | 7.8MB | **0.5MB** |
| 512  | 7.8MB | **128KB** |

> 💡 **关键**: VRChat 官方明确 "**Each step of texture resolution increases memory/filesize by 400%!**"

#### Format（格式）

- **DXT5 | BC3**: 适用于 **99%** 的贴图，**导入透明度**
- **DXT1 | BC1**: 不需要透明度时使用，体积比 DXT5 小一半
- ⚠️ **法线贴图（Normal）** 不应更改格式，保持 Automatic（**会破坏 R 通道**）

#### Compression（压缩质量）

| 选项 | 体积 | 画质 | 备注 |
|------|------|------|------|
| **None** | 最大（甚至 > 原图） | 无损 | "完全不进行压缩" |
| **Low Quality** | 极小 | 较低 | 进一步压缩可选 |
| **Normal Quality** | 中等 | 良好 | 默认推荐 |
| **High Quality** | 较大 | 最佳 | 皮肤贴图等关键部位 |

⚠️ **翻译陷阱**: 中文 Unity 中 "Normal Quality" 会被错误翻译为"**法线质量**"，实际是"**普通质量**"

#### Crunch Compression（使用 Crunch 压缩）

- ⚠️ **任何情况下都应避免使用**！
- 只会减小**下载大小**，**不减小 VRAM 占用**
- 增加 CPU 负担 + 贴图破坏

> [FACT] **VRChat 官方明确立场 (2024-03-14 Dev Update)**:
>
> "Crunching textures may or may not help. **Avoid relying on it.** Crunch reduces quality and doesn't affect uncompressed size at all. The AssetBundle compressor does a decent job at compression anyways, and finally, un-Crunching costs CPU time. As such, **you shouldn't use Crunch unless you know you need it.**"

### 推荐分辨率矩阵（按贴图类型）

> **来源**: VRCD 18-vrchat-CI4 §Max Size 表格

| 分辨率 | 主色 / Albedo | 法线 / Normal | 材质捕捉 / MatCap | 各种 Mask |
|--------|---------------|----------------|--------------------|-----------|
| 128    |               |                | ✓                  |          |
| 256    |               |                | ✓                  | ✓        |
| 512    |               | ✓              |                    | ✓        |
| 1024   | ✓             | ✓              |                    |          |
| 2048   | ✓             |                |                    |          |
| 4096   | （不推荐）    | （不推荐）     | （不推荐）         | （不推荐）|

> **选型原则**:
> - **Albedo**: 1024 或 2048 即可
> - **Normal**: 512 或 1024
> - **MatCap**: 128 或 256
> - **Mask**: 256 或 512

### 批量化工具

> 详细文档: `ndmf-tools.md §1.4`

- **lilAvatarUtils** (lilxyzw): 批量调整贴图导入设置（原生不支持中文，需汉化）
  - GitHub: https://github.com/lilxyzw/lilAvatarUtils
  - 汉化: https://hrenact.booth.pm/items/5843998
- **Avatar Compressor (LAC)**: 全自动压缩（推荐，详见 `lac-avatar-compressor.md`）
- **AAO Trace And Optimize**: 自动 + 全面优化

---

## Skinned Mesh Renderer（蒙皮网格）

### 合并条件

能配合身体骨架变形的物件可以合并。

### 合并好处

- CPU 处理负担降低
- Material Slots 同步合并（同一材质球的多 Mesh 算一个 Slot）

### 代价

> ⚠️ 无法单独开关其中某个物件（不能穿脱衣服）

### Face（头）特殊处理

Face 的 Blend Shape 数量通常最多，全部合并反而增加负担。

**建议**：头髮、身体等可合并，Face 保持独立。

### Excellent 挑战

1. 把 `Body` 也扔进 MergeMesh
2. 使用 `Freeze BlendShape` 固定外观并删除 Blend Shape

### AAO Merge Skinned Mesh 步骤

```
1. 在 Avatar 物件上 Add Component > AAO Merge Skinned Mesh
2. 绑定 Root Transform（通常是 Hips）
3. 设置 Anchor Override（建议用 Body）
4. 将要合并的物件拖入「添加元素」
5. 取消勾选 Optimize BlendShape（如需保留）
```

---

## Material Slots（材质栏位）

### Material vs Material Slots

| 概念 | 定义 | 影响 |
|------|------|------|
| Material（材质球） | 以文件为单位 | 无 |
| Material Slots（材质栏位） | 编译后实际数量 | 影响 Performance Rank |

### 减少方式

1. **合并 Skinned Mesh** — 同材质球的多 Mesh 合并后算一个 Slot
2. **合并材质球（Atlas 化）** — 使用 TexTransTool

### TexTransTool Atlas 化

> 工具: https://ttt.rs64.net/ — **完整文档: `tex-trans-tool.md`**
> 当前版本: v1.0.1（2025-12-26）

#### ⚠️ 能力边界（重要）

> TexTransTool 可以压**材质球** + **VRAM**，但**不直接压材质槽**（Material Slot）
> 想真正减少 Material Slot → **必须配合 AvatarOptimizer.MergeSkinnedMesh**

#### 注意事项

> ⚠️ 必须按材质性质分组（皮肤/头髮/衣服/半透明/玻璃/金属），否则透明材质会混合错误

#### 分组方法（v1.0.1 当前）

```
AtlasTexture GameObject:
├ Material Selector: 勾选要合并的材质球
├ Material Merge Group: 按性质分组（实验性→稳定）
├ All Material Merge Reference: 指定所有材质合并目标
├ Atlas Texture Size: 输出纹理大小（2 的幂）
└ Force Set Texture: 强制覆盖纹理（Quest 适配用）
```

> 💡 v0.10.0 后：`MergeMaterial` → `AllMaterialMergeReference`（语义更清晰）

#### Quest 适配关键步骤

```
1. Material Selector 全选所有材质
2. All Material Merge Reference → Quest 专用材质
3. Force Set Texture → True
4. Build → Quest 端：1 个材质 + 1 张图集
```

#### 烘焙技巧

复杂材质合并出问题？尝试烘焙：

```
1. 备份该材质球
2. 选择适合的烘焙选项
3. 完成
```

感谢 Touma 分享。

#### 与 AvatarOptimizer 协作（v0.9.0+）

```
执行顺序（关键）：
1. TTT AtlasTexture（图集化、合并材质球）
2. AAO MergeSkinnedMesh（合并 Mesh + Material Slot）
3. AAO TraceAndOptimize（最终优化）
```

v0.9.0 起 TTT 与 AAO 通过 API 深度协作：
- AAO RemoveMeshBy* 删除的区域不会分配图集空间
- AAO UV 写入 UV1 时 TTT 通过 API 报告改写位置

#### 故障排查速查

| 问题 | 原因 | 解决 |
|------|------|------|
| 上传不生效 | 组件未 Active | 激活组件 |
| 上传不生效 | NDMF 未安装 | 安装 NDMF v1.7.0+ |
| GTX10/9 + DX11 故障 | 已知 GPU 兼容问题 | 升级 v0.10.0+ 或用 Vulkan |
| Gizmo 不显示 | SceneView 开关 | 启用 Gizmo 总开关 |
| 旧资产不工作 | SaveData 不兼容 | Tools > TexTransTool > Migrator |

---

## Render Queue 规则 ⭐NEW 2026-07-04

> 来源:lilAvatarUtils 官方文档 https://lilxyzw.github.io/lilAvatarUtils/en_US/docs/MaterialsGUI.html
> 补充意义:Render Queue 配置错误是 Avatar 渲染异常的常见根因(memory 库原无此规则)

### 三个关键阈值

| Render Queue 范围 | 行为 | 风险等级 | 典型症状 |
|------------------|------|---------|---------|
| **< 2500**(含透明材质) | 与天空盒渲染顺序冲突 | 🟡 中 | 透明物体在天空盒后面、显示异常 |
| **≥ 2501** | 失去焦点 + 无法接收阴影 | 🟡 中 | 半透明物体模糊,投影丢失 |
| **≤ 2450**(透明材质过低) | ⚠️ **擦除其他材质** | 🔴 高 | 整个 Avatar 的部分材质被擦除 |

### 详细规则

#### 1. 透明材质 < 2500

- **官方原文**:"If a material that includes transparency is set to less than 2500, rendering problems may occur when it overlaps with the skybox."
- **根因**:Render Queue < 2500 会被 Unity 当作"不透明"队列,在天空盒**之后**渲染
- **症状**:在户外场景中,透明部分在天空盒"后面"显示
- **解决方案**:将透明材质的 Render Queue 设为 2501(Unity Transparent 标准)

#### 2. Render Queue ≥ 2501

- **官方原文**:"If it is set to 2501 or more, the lens effect will cause the material to lose focus and it will not be able to receive shadows."
- **根因**:Transparent 队列的材质不参与深度写入
- **症状**:
  - 失去焦点(虚化效果失效)
  - 无法接收其他物体的阴影
- **影响**:通常这是**正确的**,除非需要透明材质接收阴影

#### 3. Render Queue ≤ 2450(严重 Bug)

- **官方原文**:"If a transparent material is set to an excessively low value (such as 2450 or less), it is very likely to cause problems with other materials being erased."
- **根因**:过低的 Render Queue 会导致 Z-test 异常
- **症状**:其他材质被"擦除"(从视觉上消失)
- **影响**:🔴 **破坏性问题**,必须立即修复
- **解决方案**:将 Render Queue 调整到 2501(标准透明)或 3000+(特殊效果)

### 实践建议

| 材质类型 | 推荐 Render Queue | 说明 |
|---------|------------------|------|
| **不透明(Opaque)** | 2000-2449 | 默认(Standard) |
| **透明(Transparent)** | 2501-2999 | 标准透明(Geometry+ queue) |
| **粒子 / 特效** | 3000+ | Overlay 队列,最顶层渲染 |
| **Cutout 镂空** | 2450-2499 | Alpha Test 队列(Unity 标准) |

### 诊断工作流

```
1. 发现 Avatar 材质异常(消失/重叠/失焦)
2. 选中异常材质 → Inspector → Shader Properties
3. 检查 Render Queue 字段
4. 与上述三档规则对照
5. 修正 Render Queue 到推荐范围
6. 重新构建测试
```

### 常见误用

| 误用 | 后果 | 正确做法 |
|------|------|---------|
| 把所有材质都设为 2450 强制前渲 | 擦除其他材质 | 透明用 2501,镂空用 2450 |
| 半透明用 Render Queue = 2000 | 与天空盒冲突 | 改为 2501 |
| 在 Particle System 用 Render Queue = 2450 | 遮挡问题 | 改用 3000+ |
| 复制 Shader 时不修改 Render Queue | 渲染顺序错乱 | 按材质类型手动调整 |

### 工具辅助

- **lilAvatarUtils Materials Tab**:可一次性查看所有材质的 Render Queue
- **修改方式**:lilAvatarUtils 中可批量修改 + Apply 一次性应用
- **验证方式**:改完后用 lilAvatarUtils Lighting Tab 的 In Shadow / No Light 模式验证

---

## PhysBone 最佳化

> 简称 PB
>
> 💡 **用户端 Dynamic Bone Limits 系统**(2026-06-30 新增)
>
> 上述指标是**上传时** Performance Rank 评估,而 VRChat 客户端还有**独立的**运行时限制系统:
> - 默认: Max Affected Transforms = 32, Max Collision Checks = 8
> - 超出 → 所有 Dynamic Bone + Collider 组件被禁用(非部分禁用)
> - 配置: `config.json` 在 `%AppData%\..\LocalLow\VRChat\vrchat\`
> - 详见 [[entities/avatar/avatar-dynamic-bone-limits]]

### 两种实装方式

| 方式 | 优点 | 缺点 |
|------|------|------|
| 直接建立在 Transform | 安装简单，不需指定 Root | 管理困难，散布各处 |
| 建立在空白物件内 | 管理方便，可批量启用/停用 | 需手动指定 Root Transform |

### 合并 PhysBone

#### 合并条件

1. 不同 PB 的上一级必须有共同 Root Transform（不可超过两级）
2. 设定数值必须**完全一致**

#### 限制

> ⚠️ 抓取只能抓一边（同时抓两边有一边没反应），摇晃与碰撞不受影响

#### 合并步骤（Transform 内）

```
1. 展开 Armature，找到有相同设定的 PB
2. 在共同 Root Transform 上新建空白物件（如 Hair_Side）
3. 将左右 PB 拉入该物件层级
4. Add Component > AAO Merge PhysBone
5. 将左右 PB 拖入添加元素
```

#### 合并步骤（空白物件内）

```
1. 展开物件，检查所有 PB 的 Root Transform 位置
2. 找到可合并的 PB（如 Ear_L 和 Ear_R）
3. 新建空白物件（如 Ear），将 PB 拉入
4. Add Component > AAO Merge PhysBone
5. 拖入目标 PB
```

### 停用/删除 PhysBone

**停用方向**：
- 会捏/拽但很少人用的 PB（脸颊等）
- 太小/移动幅度不明显的 PB

**终极做法**：如果 PB 影响物件关闭后不影响整体，直接删除。

### 碰撞器优化

> 💡 大多数 Avatar 本身已内建完善 Collider，衣服自带的可以替换

**替换步骤**：
```
1. 固定 PB 的 Inspector
2. 找到 Avatar 本体相同位置的 Collider
3. 替换衣服自带的 Collider
4. 删除衣服自带 Collider 元件
```

---

## Bones（骨頭）合并

> 工具: AAO Merge Bone

### 作用

将目标骨融合至上一级父骨，减少一根骨。

**一举三得**：
1. 减少 Bones 数量
2. 减少 PB 影响骨头数
3. 避免设定 PhysBone Ignore 时末端不能动的问题

### 合并技巧

> 💡 融合间隔选择（上半部移动幅度小的保留，下半部移动幅度大的也保留）

### 注意事项

> ⚠️ 融合后若有 PhysBone，动态效果可能变僵硬

**调整方法**：降低 Pull、Momentum、Immobile。

感谢 Touma 与夜嵐蝶 Alma 介绍。

---

## Polygons（面数）减面

### AAO Remove Mesh 系列

| 工具 | 用途 | 使用场景 |
|------|------|----------|
| **Remove Mesh By BlendShape** | 删除隐藏身体部位的面数 | 使用 Blend Shape 隐藏身体防止穿模时 |
| **Remove Mesh By Mask** | 用遮罩贴图移除 | 多物件做在同 Skinned Mesh 时 |
| **Remove Mesh By Box** | 用方块裁切 | 无法用 Blend Shape 或 UV 时 |
| **Remove Mesh By UV Tile** | 按 UV Tile 移除 | 移除特定材质球的所有面数 |

### Remove Mesh By BlendShape

> 💡 Blend Shape 隐藏身体后，隐藏部分面数不会消失，只是被挤成一坨看不见的东西

**使用方法**：
```
1. 选择有 Blend Shape 的物件
2. Add Component > Remove Mesh By BlendShape
3. 勾选用于隐藏身体的 Blend Shape
4. 完成
```

### Remove Mesh By Box

可以与 Mask 组合使用，砍得更精准。

### 减面工具对比

| 工具 | 优点 | 缺点 | 价格 | 状态 |
|------|------|------|------|------|
| Mantis LOD Editor | 模型不易破洞 | 调整时卡顿 | **$50** | ✅ 活跃 |
| **Meshia Mesh Simplification** | Burst 高速、BlendShape 保留、防破洞选项丰富 | 需配置 Options | 免费 | ✅ **推荐** |
| lilNDMFMeshSimplifier | 导入即见效 | 看不到面数、无防破洞 | 免费 | ❌ **已废弃** |

### Meshia Mesh Simplification 使用技巧

> 📖 **完整技术文档**: `meshia-mesh-simplification.md`
> 💡 该工具不会计算其他 NDMF 工具处理过的结果，实际面数可能低于目标

#### 单 Mesh 模式（`MeshiaMeshSimplifier`）

```
1. 选择目标 GameObject
2. Add Component → MeshiaMeshSimplifier
3. 配置 Target（MeshSimplificationTargetKind + Value）
4. EditMode 实时预览
```

#### Avatar 级联模式（`MeshiaCascadingAvatarMeshSimplifier`）

```
1. 在 Avatar 根节点下创建空 GameObject
2. Add Component → MeshiaCascadingAvatarMeshSimplifier
3. 选择目标 Performance Rank
4. 调整每个 Mesh 的面数分配 + Options
```

#### 三个关键修复选项

| 问题 | 启用选项 |
|------|----------|
| 指尖/边缘破洞 | `PreserveBorderEdges = true` |
| UV 接缝纹理扭曲 | `UseBarycentricCoordinateInterpolation = true` |
| 曲面失真 | `PreserveSurfaceCurvature = true` |

#### 其他技巧

- 使用 `Enable Auto Adjust` 批量调整
- 使用锁头固定特定物件面数
- v3.1+ 支持按骨配置 Border Edges（更精细）
- 目标面数设太低时调高预期
- **不要混用 lilNDMF + Meshia**（同类工具重复执行）

---

## 批次上传

> 工具: Continuous Avatar Uploader

**建议**：
- 分散到多个 Asset 管理（建议每个 5~10 只）
- 出错时从出错那组重新上传（类似存档点）
- 单个 Asset 建议不超过 10 只

---

## 绕过 VRCSDK 预构建硬限制检查

> 📚 **详细知识**: [[entities/avatar/avatar-optimizer]] §8.2

### 现象

使用 NDMF 非破坏性工具（AAO, TTT, Meshia 等）后：
- 模型**实际**已满足硬限制（AAO 处理后）
- 但 **VRCSDK Control Panel 上传按钮被禁用**
- 原因：VRCSDK 在编译前检查性能等级，NDMF 工具尚未运行

### 3 种解决方案

| 方法 | 来源 | 优势 | 风险 |
|------|------|------|------|
| **NDM Framework/Manual bake avatar** | NDMF 官方 | 最稳妥，原始模型不被修改 | 需要手动操作 |
| **Sayamame-beans/Upload-without-preCheck** | 社区 | 一键操作 | VRCSDK 3.8.1-beta.1 暂时不兼容 |
| **VRCQuestTools** (VQT Avatar Builder) | kurotu | Quest 转换 + 绕过预检查 | 主要是 Quest 工具 |

> ⚠️ **重要前提**：所有方法**不绕过构建后硬限制**。模型如果真超过限制，上传仍会失败。

---

## 布料系统(Cloth)(2026-07-04 来自 yexcadocs)

> 来源:yexcadocs §Cloth(P1)
> 本节为社区实操经验补充。

### Cloth vs PhysBones 区别

| 维度 | Cloth | PhysBones |
|------|-------|-----------|
| 物理模型 | 基于物理面料模拟(有限元) | 基于骨骼链(Verlet) |
| 性能 | 重(每帧布料计算) | 轻(骨骼链解算) |
| 必要条件 | 搭配 Skinned Mesh Renderer | 无 |
| 网格要求 | 必须单层/缝合/四边形 | 无特殊要求 |
| 适用场景 | 裙摆、长袍、披风 | 头发、尾巴、绳子 |

### 必要条件

- 必须搭配 **Skinned Mesh Renderer**(否则 Unity 自动转换)
- 网格必须**单层/缝合/四边形**

### 碰撞器配置

- 仅支持 **Capsule Collider** + **Sphere Collider**
- **两个 Sphere Collider 自动形成锥体碰撞**(隐藏功能)
- ⚠️ **必须勾选 Is Trigger**,否则会"穿模 + 转圈"

详细 PhysBones 警告与抓取参数 API 见 `avatar-dynamic-bone-limits.md §9`。

---

## Trail Renderer 实操要点(2026-07-04 来自 yexcadocs)

> 来源:yexcadocs §Trail Renderer(P2)
> 本节为社区实操经验补充。

| 要点 | 说明 |
|------|------|
| **AutoDestruct** | 启用后,Trail 在时间结束后自动销毁(适合一次性效果) |
| **Emitting** | 控制 Trail 是否持续发射(关闭时 Trail 不增长) |
| **SetTextureScale** | 通过 Shader 调整 Trail 纹理比例(性能优于修改材质) |

**注意**:Trail Renderer **会计入 2 个材质球**配额(见 §Particle System)。

---

## 上传流程(2026-07-04 来自 yexcadocs)

> 来源:yexcadocs §上传前准备 / 成品模型上传(P1)
> 本节为社区实操经验补充,适用于 VRChat 上传阶段。

### 前提条件

- ⚠️ **Steam 登录账号不能上传**,需官网注册
  - 判断方法:账号名字后有其他字符说明是 Steam 登录
- 账号等级需达到蓝色(**New User**)

### Unity 版本绑定

- ⚠️ 当前 VRChat SDK 支持 **Unity 2022.3.22f1**(与 SOUL.md 一致)
- 旧版 Unity 2019.4.31f1c1 已不适用

### 上传面板流程

```
1. 打开 VRChat SDK → Show Control Panel → Builder
2. 登录 VRChat 账号
3. 等待编译(显示 Build Available)
4. 点击 "Build & Publish for Windows"
```

**如果 Build & Publish 不可点击**:
- 上方应有六边形红色感叹号
- 每个警告后方都有 `Auto Fix`
- 逐项点击 Auto Fix 修复
- 修复后即可点击 Build & Publish

### 故障排除

| 错误信息 | 解决方案 |
|---------|---------|
| `A VRCAvatarDescriptor is required to build an Avatar` | 模型是 SDK2,需升级或用 SDK2 上传 |
| `BluePrint` 错误 | 选中角色 → 右方划到最底 → 点击 `Detach` 后重新上传 |
| Build & Publish 不可点击 | 点击 Auto Fix(可能多项) |
| `Sharing` 设置 | `Private` 防止他人复制,`Publish` 公开 |

### 封面创建(VRCCam)

> ⚠️ **时效性更新 (2026-07-04)**: VRCCam 已在 SDK 2025-08 后被替换为 `ThumbnailCapture` 组件。详见下方 `§上传前:Avatar 缩略图优化` 章节。

**步骤** (旧版参考,SDK < 2025-08):
```
1. 在场景中找到 VRCCam GameObject
2. 右击 → 3D Object → 四边形(Quad)
3. 设置 Transform:
   - 位置 z = 2.6
   - 缩放 (x=4, y=3, z=1)
4. 准备图片材质:
   - 创建 Material → Shader 选择(如 VRChat-Mobile-Toon Lit)
   - 拖入图片到材质
5. 将材质拖到四边形
6. 隐藏模型(取消左侧复选框)
7. 进入 Game 视图预览
8. 再次点击 Upload
```

### 关键限制

| 限制 | 数值 | 来源 |
|------|------|------|
| **模型加载大小** | **200MB** | yexcadocs(P1,需查证当前值) |
| **共享设置** | Private 防止他人复制 | VRChat 官方 |

### "重启解决 90%" 法则

> 故障排除优先级(作者经验):
> 1. 重新上传
> 2. 重新打开 Unity
> 3. 新建工程上传
> 4. 重启电脑
> 5. 检查网络

---

## Animation Optimization（动画优化）⭐NEW 2026-07-04

> **来源**: VRCD 18-vrchat-CI4 §动画方面 + VRChat 官方 Dev Update (2024-03-14)
> **背景**: Animation Clip 是 Avatar 中常被忽视的大小来源

### 关键原则

> [FACT] **VRChat 官方明确 (2024-03-14 Dev Update)**:
>
> "Don't use **fully-keyframed animations**, these can get really large!"

- ❌ **避免**完全关键帧（fully-keyframed）动画
- ❌ **避免**添加过多 MMD 动画
- ❌ **避免**使用动作包（携带大量未使用动画）
- ✅ **降低**动画曲线精度

### 数据量原理

- 一帧 N 个信息 → N 空间占用
- **动作动画**（手臂从大腿举到头顶）: 浮点小数点精度决定空间占用
  - 15°、16.98°、78.24°... → 每帧都存储
- **完全关键帧+无补间** = 巨大占用

### Unity 内置压缩选项

> 完整 Unity Manual: https://docs.unity3d.com/2022.3/Documentation/Manual/class-AnimationClip.html

| 选项 | 作用 | 推荐度 |
|------|------|--------|
| **Keyframe Reduction** | 减少冗余关键帧 | ⭐⭐⭐ |
| **Optimal** | 根据阈值自动减少冗余关键帧 | ⭐⭐⭐⭐⭐ 推荐 |

### 推荐工具

- **AAO Trace And Optimize** — 自动 Animator 优化
- **VRCFury Actions 系统** — 模块化动画
- **Unity 内置** — AnimationClip 导入设置中的 Animation Compression

### Animation 与其他优化的关系

> ⚠️ Animator 中有开关物件的 Animation 时，**AAO 会跳过 Skinned Mesh 合并**
> - 在优化前先清理 / 简化 Animator

### 引用

- VRChat 官方: "Don't use fully-keyframed animations, these can get really large!"
- Unity 官方: https://docs.unity3d.com/2022.3/Documentation/Manual/class-AnimationClip.html

---

## Audio Optimization（音频优化）⭐NEW 2026-07-04

> **详细文档**: `avatar-audio-optimization.md` — 完整音频优化指南（4 压缩格式 + 3 加载类型 + 量化数据 + Load In Background）

### 速查要点

- **短音效**（脚步声、武器声）: ADPCM + Decompress On Load
- **长音频**（音乐、歌曲）: Vorbis + Compressed In Memory
- **绝不在 Avatar 中使用 Streaming**

### Quest 平台特殊

- Vorbis CPU 消耗高 → Android 平台使用 **MP3** 替代
- 详见 `avatar-audio-optimization.md §1.4 / §6.3`

---

## §上传前:Avatar 缩略图优化 ⚠️ 关键(2026-07-04 来自 22-7f6ff)

> [FACT] **来源**: https://creators.vrchat.com/avatars/creating-your-first-avatar/ + https://feedback.vrchat.com/sdk-bug-reports/p/sdk-382-unity-crash-upon-saving-scene-after-using-camera-for-thumbnail
> **优先级**: 🔴 高(影响 Avatar 浏览第一印象)
> **P 等级**: P1

### 时效性:VRCCam → ThumbnailCapture 替换

> [FACT] **VRChat 官方明确**: "this crash has been around since we moved away from the old VRCCam in the scene to make thumbnails."

**旧版(SDK < 2025-08)**:
- VRCCam 是场景中的对象, 上传前自动创建
- 拖动可改变位置
- 可调整 FOV、背景、纯色

**当前(2026 推荐): ThumbnailCapture 组件**
- SDK 已升级为 `ThumbnailCapture` 组件
- **标准: 4:3 宽高比**

### 简单方法(默认)

1. 上传窗口 → 选择图片文件(从 PC)
2. 或从 Unity 场景捕获

### 高级方法(自定义缩略图)

1. 创建 `GameObject > 3D Object > Plane`
2. 导入图片作为纹理
3. Plane Shader = **Unlit**
4. 调整 Plane 大小匹配 4:3 比例(例: 0.4 x 0.3)
5. 放置 Plane 在场景中(不与 Avatar 重叠)
6. 添加 `Sharky's VRC Avatar Utils > Align Upload Camera` 组件
7. Build → Upload 窗口中调整

> **工具 URL**: https://sharkys-vrc-avatar-utils.readthedocs.io/en/latest/HowTo/UsePictureAsThumbnail/

### 滴管工具取色技巧

- 选中背景 → **Color Picker** 工具
- 从 Avatar 拾取颜色 → 微调得到协调的纯色背景

### Build & Test 模式(无需上传测试)

> [FACT] **官方明确**: "switch the build type to 'Build & Test Your Avatar'. After a local build, your avatar will appear in the 'SDK Test Avatars' section of your VRChat Avatars menu. Test avatars can only be seen by you."

- 切换构建类型到 **Build & Test Your Avatar**
- 本地构建 → Avatar 出现在 "SDK Test Avatars"
- **仅自己可见**, 适合上传前测试

### Viseme 配置方式

> [FACT] **来源**: https://creators.vrchat.com/avatars/

VRChat 支持 2 种 viseme 方式:
1. **Jaw-flap bone** - 在 Rigging Configuration Screen 分配 jaw-flap 骨骼
2. **Blendshapes** - 通过 Blend Shape 定义 viseme(更灵活)
3. **Viseme type 选项**: `Jaw Flap`(1 个 BlendShape)/ `Viseme`(多个 BlendShape)

详见: `[[entities/avatar/avatar-upload-getting-started]] §6`
`[[entities/avatar/blender-shape-keys]] §4.5`

---

## 相关文档

- `avatar-optimizer.md` — AAO: Avatar Optimizer 完整组件知识库 (12 组件 + API)
- `performance-rank.md` — Performance Rank 标准
- `ndmf-tools.md` — NDMF 工具生态与获取方式
- `meshia-mesh-simplification.md` — Meshia 减面工具完整技术文档
- `avatar-size-limits.md` — Avatar 大小限制与定义（K01+K02）⭐NEW 2026-07-04
- `avatar-audio-optimization.md` — Avatar 音频优化（K05+K06+K07+K12）⭐NEW 2026-07-04

---

## 相关页面

[[performance-rank.md]] · [[ndmf-tools.md]] · [[playable-layers.md]] · [[vrc-constraints.md]] · [[vrcfury-reference.md]] · [[modular-avatar.md]] · [[../world/performance-guide.md]] · [[avatar-size-limits.md]] · [[avatar-audio-optimization.md]] · [lilToon 官方](https://lilxyzw.github.io/lilToon/)
