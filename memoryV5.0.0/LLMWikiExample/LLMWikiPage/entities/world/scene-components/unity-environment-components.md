---
title: "Unity 环境/地形/文本杂项合并专页（Terrain / Tilemap / Skybox / TextMesh）"
category: world
subcategory: scene-components
knowledge_level: applied
status: active
source: "Unity Manual 2022.3（Terrain / TerrainCollider / Tilemap / TilemapRenderer / Skybox / TextMesh 各页）+ VRChat 官方白名单"
source_type: official
version: 1.0
last_review: 2026-08-19
confidence: high
tags:
  - world
  - unity
  - rendering
  - physics
  - mesh
  - reference
aliases:
  - Terrain
  - Tilemap
  - Skybox
  - TextMesh
  - TerrainCollider
  - TilemapRenderer
  - 环境/地形/文本合并页
related:
  - ../whitelisted-world-components.md
  - textmeshpro.md
  - ../../api/ui.md
  - index.md
type: entity
created: 2026-08-19
sources: "https://creators.vrchat.com/worlds/whitelisted-world-components/ | https://docs.unity3d.com/2022.3/Documentation/Manual/script-Terrain.html | https://docs.unity3d.com/2022.3/Documentation/Manual/class-TerrainCollider.html | https://docs.unity3d.com/2022.3/Documentation/Manual/class-Tilemap.html | https://docs.unity3d.com/2022.3/Documentation/Manual/class-TilemapRenderer.html | https://docs.unity3d.com/2022.3/Documentation/Manual/class-Skybox.html | https://docs.unity3d.com/2022.3/Documentation/Manual/class-TextMesh.html"
updated: 2026-08-19
---
# Unity 环境/地形/文本杂项合并专页（Terrain / Tilemap / Skybox / TextMesh）

> **白名单地位**: 本页 4 个组件 **Terrain、Tilemap、Skybox、TextMesh** 均经逐项核验，**全部在**官方 [Allowlisted World Components](../whitelisted-world-components.md)「Unity Components」内，**World 可用**。同段还并列 TerrainCollider、TilemapRenderer 两个伴随组件（详情见逐组件小节）。
> **本页为合并页**：4 个组件各自无独立专页，统一收录于此。低通相关/视觉可选链见「相关页面」。
> **VRChat 专属限制**: 4 个组件 VRChat 侧均无专属专页 → **[FACT] 白名单可用** + **[UNKNOWN] 无 VRChat 侧限定/优化说明**。

---

## 逐项白名单核验结论 [FACT]

> 依据官方白名单页面原文（https://creators.vrchat.com/worlds/whitelisted-world-components/ ，Last updated **Nov 25, 2025**）「Unity Components」段逐一比对（本地 `whitelisted-world-components.md` 的 Rendering/Physics 分段是本知识库的归类，官方原文为扁平列表）。

| 组件 | 白名单在/不在 | 官方原文锚 |
|---|---|---|
| **Terrain** | ✅ **在** | "Terrain" 条目（链接 `script-Terrain.html`） |
| **TerrainCollider** | ✅ **在**（Terrain 伴随） | "TerrainCollider" 条目（链接 `class-TerrainCollider.html`） |
| **Tilemap** | ✅ **在** | "Tilemap" 条目（链接 `class-Tilemap.html`） |
| **TilemapRenderer** | ✅ **在**（Tilemap 伴随） | "TilemapRenderer" 条目（链接 `class-TilemapRenderer.html`） |
| **Skybox** | ✅ **在** | "Skybox" 条目（链接 `class-Skybox.html`） |
| **TextMesh** | ✅ **在** | "TextMesh" 条目（链接 `class-TextMesh.html`） |

**核验方法**：直接抓取上述官方白名单 URL（HTTP 200），在其「Unity Components」扁平清单中逐项命中以上 6 个条目；无一项缺失。**因此本页 4 个主组件均可宣称白名单地位（[FACT]**，证据=官方原文条目）。以下各小节同步标注来源 URL。

---

## 各组件核心参数对比（[FACT] Unity Manual 2022.3）

| 组件 | 类/语义 | 作用 / 效果 | 核心参数 | 白名单 |
|---|---|---|---|---|
| **Terrain** | `Terrain` | 场景地形景观：创建多个 Terrain tile、调节高度/外观、添加树木与草丛；运行时优化地形渲染 | 地形绘制/高度/植被（编辑器工具集，无单一组件参数表） | ✅ |
| **TerrainCollider** | `TerrainCollider` | 匹配 Terrain 几何的物理碰撞体（对 Terrain 而言最精准的碰撞体） | Provides Contacts、Material、Terrain Data、Enable Tree Colliders、Layer Overrides | ✅ |
| **Tilemap** | `Tilemap` | 存储/管理 Tile Assets 创建 2D 关卡；把瓦片信息转给 Tilemap Renderer 与 Tilemap Collider 2D | Animation Frame Rate、Color、Tile Anchor、Orientation（XY/XZ/YX/YZ/ZX/ZY/Custom）、Offset/Rotation/Scale、Info（Tiles/Sprites） | ✅ |
| **TilemapRenderer** | `TilemapRenderer` | Tilemap GameObject 的一部分，决定其上 Tiles 如何被渲染 | Sort Order、Mode（Chunk/Individual）、Detect Chunk Culling Bounds、Sorting Layer、Order in Layer、Mask Interaction | ✅ |
| **Skybox** | `Skybox` | 覆盖同一 GameObject 上 Cameras 所绘制的天空盒；多 Camera 场景可按 Camera 绘制不同天空盒 | Custom Skybox（须用 skybox Shader 的 Material）；HDRP 不支持本组件 | ✅ |
| **TextMesh** | `TextMesh`（legacy） | 生成显示文本字符串的 3D 几何（旧版组件，功能有限） | Text、Offset Z、Character Size、Line Spacing、Anchor、Alignment、Tab Size、Font Size、Font Style、Rich Text、Font、Color | ✅ |

---

## 逐组件小节

### Terrain（+ TerrainCollider）

**官方语义 [FACT]**（https://docs.unity3d.com/2022.3/Documentation/Manual/script-Terrain.html ）：

> "The Unity Editor includes a built-in set of Terrain features that allow you to add landscapes to your game. In the Editor, you can create multiple Terrain tiles, adjust the height or appearance of your landscape, and add trees or grass to it. At runtime, Unity optimizes built-in Terrain rendering for efficiency."

- 官方将 **Terrain** 作为「World building」下的地形工具集呈现（非单一组件参数页）；**运行时 Unity 对内置地形渲染做了效率优化**（[FACT]）。
- ⚠️ **URL 备注**：白名单链接指向 `script-Terrain.html`，而 `class-Terrain.html` 返回 404——本页以白名单官方链接为准（做法参考 [[playable-director.md|PlayableDirector]] 对官方链接的处理）。
- **TerrainCollider**（伴随，官方 https://docs.unity3d.com/2022.3/Documentation/Manual/class-TerrainCollider.html）：

> "The Terrain Collider builds a collider that matches the geometry of the Terrain it is attached to. It is the best and most accurate collider type for Terrains."

- 关键参数 [FACT]：**Provides Contacts**（默认关，常开会持续生成接触数据、消耗资源）、**Material**、**Terrain Data**、**Enable Tree Colliders**（默认开，为 Terrain 上的 Tree 自动生成碰撞，更准但更耗算力）、**Layer Overrides**（Layer Override Priority / Include / Exclude Layers）。

**VRChat 侧**：
- **[FACT] 白名单**：Terrain 与 TerrainCollider 均在列。
- **[FACT] 无 VRChat 官方专页**：官方无 World 侧专属说明。
- **[UNKNOWN]**：VRChat World 内 Terrain 的高度图/植被/运行时优化的实际边界（如是否与烘焙光照、玩家物理交互完全一致）——**官方未定论**，须实测复核。
- **[UNKNOWN]**：TerrainCollider 的 Provides Contacts / Enable Tree Colliders 在 World 内的性能影响——官方仅给 Unity 侧定性，无 VRChat 基准。

### Tilemap（+ TilemapRenderer）

**官方语义 [FACT]**（https://docs.unity3d.com/2022.3/Documentation/Manual/class-Tilemap.html ）：

> "The Tilemap component stores and manages Tile Assets for creating 2D levels. It transfers the required information from the tiles placed on it to other related components such as the Tilemap Renderer and the Tilemap Collider 2D."

- 关键参数 [FACT]：**Animation Frame Rate**（瓦片动画帧率倍率）、**Color**（着色，默认白=无着色）、**Tile Anchor**、**Orientation**（XY/XZ/YX/YZ/ZX/ZY/Custom）、Custom 下的 **Offset/Rotation/Scale**、**Info**（Tiles / Sprites 列表）。
- 2D Tilemap Editor 包随 2D 模板自动安装，或经 Package Manager 安装（[FACT]）。
- **TilemapRenderer**（伴随，官方 https://docs.unity3d.com/2022.3/Documentation/Manual/class-TilemapRenderer.html ）：

> "The Tilemap Renderer component is part of the Tilemap GameObject. It determines how Tiles set on the Tilemap are rendered."

- 关键参数 [FACT]：**Sort Order**、**Mode**（**Chunk**=按位置分组批量渲染，官方标明 Tilemap 最佳性能模式；**Individual**=逐瓦片渲染，可与场景其他 Renderer/自定义排序轴交互）、**Detect Chunk Culling Bounds**（Auto/Manual）、**Sorting Layer**、**Order in Layer**、**Mask Interaction**（None / Visible Inside Mask / Visible Outside Mask）。

**VRChat 侧**：
- **[FACT] 白名单**：Tilemap 与 TilemapRenderer 均在列。
- **[FACT] 无 VRChat 官方专页**：官方无 World 侧专属说明。
- **[UNKNOWN]**：Tilemap 是 2D 关卡工具，VRChat 多为 3D 世界——其 World 内可用性（如 2D 平面放置、SpriteMask 交互）**官方未定论**，须实测复核。

### Skybox

**官方语义 [FACT]**（https://docs.unity3d.com/2022.3/Documentation/Manual/class-Skybox.html ）：

> "The Skybox component overrides the skybox that Cameras on the same GameObject draw. It gives you control over what skybox a Camera renders and is useful when you have multiple Cameras in your Scene and want to draw different skyboxes depending on the Camera."

- 关键参数 [FACT]：**Custom Skybox**（Material，须使用 skybox Shader）。
- **渲染管线兼容 [FACT]**：Built-in Render Pipeline ✅、Universal Render Pipeline (URP) ✅、**High Definition Render Pipeline (HDRP) ❌ 不支持**（HDRP 自带多套天空生成方案）。

**VRChat 侧**：
- **[FACT] 白名单**：Skybox 在列。VRChat 世界天空通常由 Lighting/Skybox 设置驱动，但本组件可用于按 Camera 覆盖天空盒。
- **[FACT] 无 VRChat 官方专页**：官方无 World 侧专属说明。
- **[UNKNOWN]**：VRChat 渲染管线基线（Built-in/URP）下本组件的实际可用性与多 Camera 覆盖行为——**官方未定论**，须实测复核。

### TextMesh（经典 3D 文本，非 TMP）

**官方语义 [FACT]**（https://docs.unity3d.com/2022.3/Documentation/Manual/class-TextMesh.html ）：

> "The Text Mesh component generates 3D geometry that displays text strings."
> "Note: This is a legacy component that has limited functionality."

- **明确是 legacy 组件**，功能有限（[FACT]）；可通过 **Component > Mesh > Text Mesh** 或 **GameObject > 3D Text** 创建。
- 关键参数 [FACT]：**Text**、**Offset Z**、**Character Size**、**Line Spacing**、**Anchor**、**Alignment**、**Tab Size**、**Font Size**、**Font Style**、**Rich Text**、**Font**、**Color**。
- 最佳实践 [FACT]：文本字符串用 `\n` 换行；可用简单标记（Styled Text）着色样式；更换 Font 时须同时设置字体的材质纹理。

**与 TextMeshPro 的关系（对照）**：
- ⚠️ **两个不同组件**：本页只写**经典 TextMesh**（legacy 3D 网格文本）；**TextMeshPro 已由 [[textmeshpro.md|TextMesh Pro]] 专页覆盖**，两者勿混淆。
- 经典 TextMesh 与 UI.Text/TextMeshPro 在亮白名单中的分区也不同：TextMesh 在「Unity Components」段，而 TMP 的 `TextMesh`（类名 TextMeshPro 等 11 项）在「Text Mesh Pro」段。

**VRChat 侧**：
- **[FACT] 白名单**：经典 `TextMesh`（Unity Components 段）在列。
- **[FACT] 无 VRChat 官方专页**：官方为 World 提供的文本标准是 TextMeshPro（见 [[textmeshpro.md|TextMesh Pro]] 与 [[../../api/ui.md|UI API]]），对经典 TextMesh 无 World 侧专属说明；[[textmeshpro.md|TextMesh Pro]] 页亦从「避免使用内置 Text 组件」角度建议优先 TMP。
- **[UNKNOWN]**：经典 TextMesh 在 World 内的显示质量/性能/中文与字体资产行为——**官方未定论**，且 Unity 官方已定性其为 legacy 有限功能。

---

## VRChat 特定注意（合并）

- **[FACT] 白名单地位**：Terrain、Tilemap、Skybox、TextMesh（及其伴随 TerrainCollider、TilemapRenderer）均在官方「Unity Components」段，World 可用。
- **[FACT] 无 VRChat 官方专页**：官方文档对这 4+2 项均无 World 侧专属说明。
- **[UNKNOWN]**：这些组件在 VRChat World 内的实际行为边界（Terrain 运行时/植被、Tilemap 2D 放置、Skybox 多 Camera、经典 TextMesh 渲染）——**官方未定论**，本页不臆测，须实测复核。
- **性能定性（[INFERENCE]）**：TerrainCollider 常开 Provides Contacts / Enable Tree Colliders 会增加物理接触与碰撞计算；TilemapRenderer 应优先 **Chunk** 模式（官方最佳性能）。但均无 VRChat 基准定量，不得据此估算具体开销。

## 相关页面

[[textmeshpro.md|TextMesh Pro]] · [[../whitelisted-world-components.md|Allowlisted World Components]] · [[../../api/ui.md|UI API]] · [[index.md|Scene Components 总览]]

## 源清单

- VRChat Allowlist（官方白名单，Last updated 2025-11-25）: https://creators.vrchat.com/worlds/whitelisted-world-components/
- Unity Manual Terrain（白名单官方链接；`class-Terrain.html` 404）: https://docs.unity3d.com/2022.3/Documentation/Manual/script-Terrain.html
- Unity Manual Terrain Collider: https://docs.unity3d.com/2022.3/Documentation/Manual/class-TerrainCollider.html
- Unity Manual Tilemap: https://docs.unity3d.com/2022.3/Documentation/Manual/class-Tilemap.html
- Unity Manual Tilemap Renderer: https://docs.unity3d.com/2022.3/Documentation/Manual/class-TilemapRenderer.html
- Unity Manual Skybox: https://docs.unity3d.com/2022.3/Documentation/Manual/class-Skybox.html
- Unity Manual Text Mesh (legacy): https://docs.unity3d.com/2022.3/Documentation/Manual/class-TextMesh.html
