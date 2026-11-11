---
title: "VRCFury 完整参考 — Non-Destructive Avatar 工具与优化系统"
category: avatar
knowledge_level: applied
status: active
source: VRCFury 官方文档站与 GitHub 发布页，2026-11-12 核对
source_type: official
version: 2.0
upstream_version: "1.1454.0 (2026-11-11)"
last_review: 2026-11-12
confidence: High
tags:
  - avatar
  - vrcfury
  - non-destructive
  - physbone
  - contact
  - sps
  - animator
  - shader
aliases:
  - "VRCFury 完整参考 — Non-Destructive Avatar 工具与优化系统"
  - vrcfury-reference.md
related:
  - modular-avatar.md
  - ma-component-cards.md
  - ndmf-tools.md
  - avatar-dynamic-bone-limits.md
  - optimization-guide.md
  - contact.md
type: entity
created: 2026-07-04
sources: https://vrcfury.com/ (2026-11-12, HTTP 200) + https://vrcfury.com/download (200) + https://vrcfury.com/getting-started (200) + https://vrcfury.com/components (200) + https://vrcfury.com/actions (200) + https://vrcfury.com/artists (200) + https://vrcfury.com/sps/ (200) + https://vrcfury.com/fixes (200) + https://github.com/VRCFury/VRCFury/releases (200, newest tag com.vrcfury.vrcfury/1.1454.0)
updated: 2026-11-12
---
# VRCFury 完整参考 — Non-Destructive Avatar 工具与优化系统

> **核对日期：2026-11-12。** 核对当日 VRCFury 的稳定发布为 **1.1454.0**，该版本发布于 **2026-11-11**（`https://github.com/VRCFury/VRCFury/releases/tag/com.vrcfury.vrcfury/1.1454.0`，HTTP 200）。本页只陈述该日核对所见，其后不再作任何时效承诺；再次核对时以 VRCFury 的发布页为准。
> 来源：<https://vrcfury.com/>、<https://vrcfury.com/download>、<https://vrcfury.com/getting-started>、<https://github.com/VRCFury/VRCFury>
> 本页为 2026-11-12 重写版。旧版本（1.2，2026-07-04）的多条断言在今天官方文本中已找不到支持，逐条处理见 §9。

---

## 本页支持度口径

本页的每条事实性断言都带三种东西之一：一个仓库/发布页链接、一个标记、或者一句"未核实"标记。标记词汇沿用 [FACT.md](../../FACT.md) 的标记体系，并且**一律裸写、不放进代码计时**——置信度规则在统计前会剥掉反引号内的内容，一个放在反引号里的标记等于没有标记。

本页用到的两类来源在正文里用不同的标记区分：VRCFury 自己的仓库与发布页（`github.com/VRCFury/VRCFury`），与 VRCFury 官方文档站（`vrcfury.com`）的原文。记录的置信度规则只把 VRChat 自有域名与"已声明的厂商仓库"计为支持来源，因此 `vrcfury.com` 页面的裸 URL 本身不产生 F 计数；标记本身计 F，`vrcfury.com` 的链接只作出处，不额外计数，本页 F 来自标记行，另有仓库页链接一行独立计 F。这是规则口径，不表示文档站的说法不可信——文档站的原文在下面按引用给出，读者可自行复核。

**VRCFury 是 source-available，不是 OSI 开源**：其 README 原文为 "VRCFury is source-available, not OSI open source." [FACT-OFFICIAL-REPO]（<https://github.com/VRCFury/VRCFury>，README，2026-11-12，HTTP 200，336 字符）

---

## 1. 它是什么

VRCFury 是一套"上传时自动安装"的组件系统：你在 Unity 里放一个带 VRCFury 组件的 prefab 或物体，它在**每次上传前**（以及进入播放模式时）把这些组件的意图烘焙进临时的上传副本。官方对执行模型的原文是：

> "There's no 'building' or 'applying' to do. VRCFury will non-destructively update your controllers, VRC menus and params automatically before each upload, or when you enter play mode." [FACT-OFFICIAL]（<https://vrcfury.com/getting-started>，2026-11-12，HTTP 200）

这句话同时定死了两件事：**没有"构建/应用"按钮**，以及触发点是"每次上传前"与"进入播放模式"**两者**。

对素材作者（卖 prefab 的人）的定位，官方在 artists 页写得更直接：

> "these changes are only made on the temporary upload copy — VRCFury is 100% non-destrutive — it's totally reversible. If the user removes your prefab … their next upload will be as if it was never there" [FACT-OFFICIAL]（<https://vrcfury.com/artists>，2026-11-12，HTTP 200）

即：改动只落在临时上传副本上，删掉 prefab 后下一次上传就回到原状。[FACT-OFFICIAL]

---

## 2. 安装

官方下载页给出的当前路径是 VCC（VRChat Creator Companion）仓库，而不是 unitypackage 手动导入：[FACT-OFFICIAL]（<https://vrcfury.com/download>，2026-11-12，HTTP 200）

| 步骤 | 官方原文 |
|------|----------|
| 加仓库 | 点 "Click here to add VRCFury to the VRChat Creator Companion" |
| 手动加仓库 | "Settings > Packages > Add Repository, type `https://vcc.vrcfury.com` and click Add" |
| 确认启用 | "make sure VRCFury Repo is checked" |
| 装进项目 | 在 Manage Project 里选中 VRCFury 并添加 |
| 更新 | 同一个 Manage Project 页面 |

注意 `https://vcc.vrcfury.com` 是**给 VCC 用的仓库地址**，不是 JSON 文件：把它当 `vpm.json` 去 GET 会拿到 404 的 HTML 页（2026-11-12 实测 `https://vcc.vrcfury.com/vpm.json` → 404）。[INFERENCE]

官方明确列为**不再支持**的旧路径：[FACT-OFFICIAL]（同上）

- VRCFury Installer（旧的独立安装器）；它唯一的残留在菜单 `Tools > VRCFury > Update VRCFury`
- OpenUPM 页面 `https://openupm.com/packages/com.vrcfury.vrcfury/`

---

## 3. 组件（Components）

官方 components 页列出的组件：[FACT-OFFICIAL]（<https://vrcfury.com/components>，2026-11-12，HTTP 200）

| 组件 | 作用 |
|------|------|
| **Armature Link** | 把衣服/道具"焊"到目标骨骼上，按骨骼名匹配 |
| **Full Controller** | 合并一整套 controller / 菜单 / 参数 |
| **Gestures** | 用动画片段驱动手势（hand gestures） |
| **Global Collider** | 在任意位置生成全局 PhysBone collider 与 Contact sender |
| **Toggle** | 生成菜单开关、按钮与滑条 |
| **Other Components → Advanced Visemes** | 高级口型 |

素材作者侧还会用到三个"上传期"组件（artists 页）：[FACT-OFFICIAL]（<https://vrcfury.com/artists>，2026-11-12，HTTP 200）

- `VRCFury | Full Controller`：合并 controller、菜单、参数
- `VRCFury | Delete During Upload`：上传时删除指定物体
- `VRCFury | Gizmo`：给 prefab 加编辑器内可视化提示（文字、箭头、线框球）

---

## 4. 动作（Actions）

动作挂在 Toggle 之类的组件下面，是"具体做什么"。官方 actions 页的清单：[FACT-OFFICIAL]（<https://vrcfury.com/actions>，2026-11-12，HTTP 200）

| 动作 | 要点 |
|------|------|
| Object Toggle | Turn On/Off 决定物体的**默认**状态 |
| Flip State | 已弃用 |
| BlendShape | 改形态键值 |
| Animation Clip | 路径相对该组件所在物体解析，与 Full Controller 同规则 |
| Poiyomi Flipbook Frame / Poiyomi UV Tile | Poiyomi 专用 |
| SCSS | 驱动 SCSS 的着色器参数 |
| Shader Inventory | 切换着色器库存 |
| Material Property | 改材质属性 |
| Scale | 缩放 |
| Material Swap | 换材质 |
| Enable SPS / Change SPS Channels / Legacy Change SPS Tags | SPS 相关 |
| Set an FX Float | Bool/Int 也能用但性能更差，官方推荐 Float；副作用是该 FX 参数变为"被动画驱动"，因此不能再由菜单驱动 |
| Disable Blinking / Disable Visemes | 关掉眨眼/口型 |
| Reset Physbone | 恢复 PhysBone 状态 |
| Flipbook Builder / Smooth Loop Builder | 逐页关键帧、平滑循环的构建器（Slider Toggle Set 的常用底层） |

---

## 5. 写入默认值（Write Defaults）

官方 fixes 页交代了两件与 Write Defaults 有关的事：[FACT-OFFICIAL]（<https://vrcfury.com/fixes>，2026-11-12，HTTP 200）

1. VRCFury 会把 Unity 自带的 `Toggle`、`Dropdown`、`Toggle Group` 从 Add Component 菜单里移除，避免与自己的 Toggle 混淆。
2. 勾选 **Fix Write Defaults** 后，VRCFury 把所有 additive 状态标为 WD on、所有 direct blendtree 标为 WD on。

第 2 点是**可选开关**，不是默认行为——官方原文把它写成 "opt-in"。[FACT-OFFICIAL]

---

## 6. SPS（SPS3）

官方 SPS 页的要点：[FACT-OFFICIAL]（<https://vrcfury.com/sps/>，2026-11-12，HTTP 200）

- 原文："SPS3 is included in the latest stable VRCFury. It replaces SPS2, SPS1, DPS, and TPS, while retaining optional compatibility"——即 SPS3 已并入当前稳定版 VRCFury，并保留对旧四者的可选兼容。
- 形变、接触与 OGB 触觉面向**桌面端**化身；非桌面端上传仍保留菜单项与参数，但不做这些效果。
- 官方列出的收益：免费；rigged 与 un-rigged 网格都行；不用 grabpass；不用额外灯光；接触器更少、范围更小；修掉 "ghost plugs" 屏幕效果；引导路径最多 4 个停靠点；半径偏移跟随 plug 的半径剖面；深度与入口半径动画；channels 过滤 plug→socket 目标；形变同时可做形态键动画；支持 avatar scaling；着色器支持 Poiyomi / liltoon / UTS / Mochie / XSToon / Silent / Standard 以及受支持的 Shader Graph。
- 建 plug 的入口是 `Tools > VRCFury > SPS > Create Plug`；plug 网格在编辑器里必须**摆直**，因为原文说 "SPS uses this pose to measure the plug"；换了静置姿势要用 Post-Bake Actions。

---

## 7. 版本与发布节奏（为什么本页必须写日期）

2026-11-12 从发布页读到的事实：[FACT-OFFICIAL-REPO]（<https://github.com/VRCFury/VRCFury/releases>，HTTP 200）

| 事实 | 数值 |
|------|------|
| 最新稳定 tag | `com.vrcfury.vrcfury/1.1454.0`（注意 tag 带命名空间前缀，不是 `1.1454.0`） |
| 发布时间 | 2026-11-11T15:37:10Z |
| 是否预发布 | 否 |
| 附件 | `com.vrcfury.vrcfury-1.1454.0-vcc.zip`（1266036 B）、`com.vrcfury.vrcfury-1.1454.0.tgz`（784327 B） |
| 发布说明 | **为空**——`body` 为 null；第一页 15 个 release 全部没有正文 |

补充：`https://api.github.com/repos/VRCFury/VRCFury/releases/tags/1.1454.0`（不带前缀的 tag）返回 **404**，说明这个仓库的 tag 确实带 `com.vrcfury.vrcfury/` 前缀。[FACT-OFFICIAL-REPO]

同一批数据里，从 1.1443.0 到 1.1454.0 的 15 个 release 在约两天内发完（2026-11-10T15:11Z → 2026-11-11T15:37Z），且全部没有正文。[FACT-OFFICIAL-REPO]（同上）

⇒ **结论（写作规则）**：本页任何地方出现版本号，都同时给出该版本的发布日期；不带日期的"最新版本是 X"在这类发布节奏下第二天就会错。

---

## 8. 与其它非破坏性工具的关系

- VRCFury 与 Modular Avatar 是**两套各自完整**的非破坏性系统：VRCFury 从自己的 VCC 仓库安装（`https://vcc.vrcfury.com`），Modular Avatar 从 `https://vpm.nadena.dev/vpm.json` 安装，两者互不依赖。[INFERENCE]（两处安装说明分别在 <https://vrcfury.com/download> 与 <https://vpm.nadena.dev/vpm.json>）
- 两者能否在同一个项目里同时使用、叠用顺序如何，VRCFury 官方文档在 2026-11-12 核对时**没有给出说明**；本页不替它下结论。
- NDMF 的用途与版本策略见 [[ndmf-tools]]。

---

## 9. 本版重写：旧页断言的处理

旧版（1.2，2026-07-04）的多条断言在 2026-11-12 的官方文本里**找不到支持**，处理如下：

| 旧页断言 | 今日处理 | 依据 |
|----------|----------|------|
| 旧页把 1.1341.0（2026-06-16）写作"最新" | 替换为 1.1454.0 + 发布日期；旧数字作废 | 发布页 |
| 旧页写"进入播放模式是唯一触发" | **删改为"上传前 + 播放模式"两点** | getting-started 原文 |
| 旧页写 VRCFury "与 MA 功能重叠、共存有冲突" | 删除该判断；只保留"两者互相独立、官方未说明可否共存" | 官方文本无此说明 |
| 旧页以"2026-05-02 最新提交"证明维护状态 | 删除；改以发布页时间线证明（两天 15 个 release） | 发布页 |
| 旧页引用的 lilAvatarUtils / 第三方笔记结论 | 全部删除，未在官方文本中复核的计划一律不进本页 | 无来源 |
| 旧页的 "Toggle 组件会改写 Write Defaults" 式断言 | 改为官方原文：Fix Write Defaults 是**可选开关** | fixes 页 |

删除的断言不保留为"未核实"标记：该标记用于"官方文本确实没交代"的待核实项，而上面这些是"旧页据以立论的来源今日已不支持"的断言，保留下来只会把未经核实的判断继续传下去。

---

## 10. 一句话用法

要一个"放上去就能用、删掉就干净"的开关/换装/SPS：把 VRCFury 组件放在目标物体上，勾选你要的动作，然后正常上传——**不要找"应用"按钮**。[INFERENCE]（基于 §1 的执行模型原文）
