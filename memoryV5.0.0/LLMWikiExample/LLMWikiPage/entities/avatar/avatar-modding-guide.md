---
title: "VRChat Avatar 改模基础知识"
category: avatar
knowledge_level: applied
status: active
source: "VRCD 文档库 + 86vrchat + Iconoclass 教程（VRCLibrary）+ ColorlessColor/Avatar_Pose_Matcher (2026-07-04 补充)"
source_type: community
version: 1.4
last_review: 2026-07-04
confidence: Medium
tags:
  - avatar
  - animator
  - modular-avatar
  - unity-basics
  - liltoon
  - sdk-panel
  - model-import
aliases:
  - "VRChat Avatar 改模基础知识"
  - avatar-modding-guide.md
related:
  - ndmf-tools.md
  - animator-system.md
  - bone-binding-workflow.md
  - blend-shape-troubleshooting.md
  - avatar-particle-system-creation.md
  - ma-component-cards.md
  - ma2bt.md
  - modular-avatar-tutorials-detailed.md
  - expression-menu.md
  - avatar-30-toggle-tools.md
  - ../platform/unity-install-cn.md
  - ../platform/backup-strategy.md
type: entity
created: 2026-07-04
sources: "VRCD 文档库 + 86vrchat + Iconoclass 教程（VRCLibrary）+ ColorlessColor/Avatar_Pose_Matcher (2026-07-04 补充)"
updated: 2026-07-04
---
# VRChat Avatar 改模基础知识

> 来源: VRCD 文档库 + 86vrchat | 更新: 2026-07-04

---

## 开发环境

- **Unity 版本**: 2022.3.22f1
- **MA 当前版本**: 1.17.1+（2026-05-14 stable） | 1.18.0-alpha.0（2026-05-31 alpha）

---

## Bike Pose 问题

### 问题描述

这是 Unity（不是 Modular Avatar 或 VRCFury）的 Bug。

录制动画时，预览姿势可能被错误写入到场景中。如果**模型本身被完全解压缩**，将无法简单恢复成原始的 T-Pose 或 A-Pose。

**表现**：模型呈现扭曲的"自行车姿势"。

### 解决方案

使用 [Avatar Pose Matcher](https://github.com/ColorlessColor/Avatar_Pose_Matcher) 工具：

1. 下载 `.unitypackage` 并导入 Unity
2. 菜单栏 `Tools > Avatar Pose Matcher`
3. Bugged Avatar：拖入需要修复的模型
4. Target Avatar：拖入原版模型
5. 点击 **Match** 按钮

> ⚠️ Avatar Pose Matcher 会复制坐标和旋转数据，但**不会修改缩放（Scale）数据**

> 💡 不推荐使用旧的 Avatar_Matcher 脚本，Avatar Pose Matcher 是重写版，有更多错误检查。

### 完整工作流补充 (2026-07-04)

> 来源: ColorlessColor/Avatar_Pose_Matcher GitHub + VRCD 文档库 16-vrchat-avatar-BzV
> 置信度: High (原始仓库可访问 + 第三方文档)

#### 工具元数据

| 字段 | 值 |
|------|---|
| **GitHub** | https://github.com/ColorlessColor/Avatar_Pose_Matcher |
| **最新 release** | 1.0.1 |
| **Star** | 5(社区认可度低,但专门解决该 Bug) |
| **替代关系** | **重写并修复了** `Avatar_Matcher`(旧版 BOOTH: https://mankalo.booth.pm/items/2761082) |
| **文件数** | 1 个 Editor 文件夹(含 .cs + .meta) |

#### 完整操作流程

**Step 1: 导入**
```
1. 从 GitHub Releases 下载最新的 `.unitypackage`
   (https://github.com/ColorlessColor/Avatar_Pose_Matcher/releases)
2. 在 Unity 中双击导入
```

**Step 2: 打开窗口**
```
菜单栏 Tools → Avatar Pose Matcher
```

**Step 3: 配置字段**

| 字段 | 拖入对象 | 用途 |
|------|---------|------|
| **Bugged Avatar** | 出现 Bike Pose 的模型 | 待修复的模型 |
| **Target Avatar** | 原始正常模型 | 数据源(参考) |

**Step 4: 执行修复**
```
点击 Match 按钮
```

#### 行为细节

**控制台输出**:
- 点击 Match 后,Console 会显示被修改的骨骼名
- **骨骼不存在时自动跳过**(不会报错)
- 可作为"修复范围检查"的依据

**复制范围**:
- ✅ 复制 **坐标(Position)** 数据
- ✅ 复制 **旋转(Rotation)** 数据
- ❌ **不会修改缩放(Scale) 数据**(重要)

**为什么有 Scale 限制**:
- Scale 是模型结构的核心数据,错误修改会导致模型完全变形
- 工具作者选择只同步位置和旋转,缩放问题由用户自行处理(保留原 FBX 替换方案)

#### 已知限制与陷阱

| 场景 | 行为 | 应对 |
|------|------|------|
| 骨骼名不完全一致 | 自动跳过缺失骨骼 | 检查 Console 日志 |
| 缩放不一致 | 不修复 | 手动处理或使用原始 FBX 替换 |
| 完全解压缩的模型 | 可正常修复(这是此工具的主要场景) | — |
| 模型已修复 T-Pose | 不影响 | 跳过此工具 |

#### 与其他工具的关系

| 工具 | 关系 |
|------|------|
| `Avatar_Matcher` (旧版) | **被替代**——本文档推荐的 `Avatar_Pose_Matcher` 是重写版,有更多错误检查 |
| `Avatar 3.0 Emulator` (lyuma) | 互补——可测试修复后的 Avatar 行为 |
| `Bike Pose` Unity Bug 本身 | 此工具不修复根因,只恢复 T-Pose/A-Pose |

---

## MA Merge Animator 的 WD 兼容问题

MA Merge Animator 有"匹配 Avatar WD"设置，上传模型时 SDK 会检查是否使用了未定义的参数。

> ⚠️ VRChat SDK 默认动画控制器都是 **WD OFF**

---

## Avatar 复制工作流（Toggle 实验最佳实践）⭐NEW 2026-07-04

> [FACT] **来源**:
> - Iconoclass 教程（VRCLibrary）: https://docs.vrcd.org.cn/books/avatar-30
> - VRChat 官方: https://creators.vrchat.com/avatars/expression-menu-and-controls/
> - VRChat Wiki: https://wiki.vrchat.com/wiki/Expressions
>
> **本地化日期**: 2026-07-04
> **用途**: 在不干扰当前 Avatar 的情况下，**创建副本**作为 Toggle 实验沙盒

### 完整复制步骤

#### 1. 复制 Avatar GameObject

```
1. 在 Hierarchy 选中虚拟形象对应的 GameObject
2. 按住 Ctrl 键 + 单击方向箭头 → 将其移到模型后方（不挡视野）
3. Ctrl + D 复制
4. 将复制的对象移到前方
5. 在副本名称末尾添加 "toggles"（如 "MyAvatar_toggles"）
```

#### 2. 复制 Effect Layer（FX Layer）

```
1. 选中虚拟形象
2. 滑动至 FX Layer 字段
3. 单击该字段（聚焦）
4. Ctrl + D 复制资产
5. F2 重命名（如 "MyAvatar_FX_Toggles"）
6. 拖回 FX slot
```

#### 3. 复制 Expression Menu / Expression Parameters

同 FX Layer 步骤。

### 关键设计原则

| 原则 | 原因 |
|------|------|
| **不直接修改原始 Avatar** | 避免破坏已有功能 |
| **复制完整链路** | Avatar / FX / Menu / Params 全部复制避免引用冲突 |
| **副本统一加后缀** | 易于识别和管理 |
| **独立 Project 文件夹** | 复制 Animator Controller 等资产到独立位置 |

### 创建 Expression Menu / Parameters 资产

> [FACT] **官方路径**（来源: https://creators.vrchat.com/avatars/expression-menu-and-controls/）:
> - `Project 右键 > Create > VRChat > Avatars > Expressions Menu`
> - `Project 右键 > Create > VRChat > Avatars > Expression Parameters`

### ⚠️ 重要警告

> [FACT] 来源: https://wiki.vrchat.com/wiki/Expressions
> **使用自定义 Expression Menu 将删除默认的 emotes menu**，因此需要创建一个类似于 emotes menu 的新子菜单来替代。

> 详见 `expression-menu.md §9.6` - 完整警告说明

### 创建 Toggles 动画文件夹

```
1. 在 Project 找到合适位置（如 animations 文件夹）
2. 右键 > Create > Folder
3. 重命名为 Toggles
4. 按虚拟形象分子文件夹：
   Toggles/
   ├── MyAvatar1/
   └── MyAvatar2/
```

### 关联工具

- **VRChat Toggle Assistant**: 一键生成 Toggle，详见 `avatar-30-toggle-tools.md §2`
- **MA Object Toggle**: 编译时生成，详见 `modular-avatar-tutorials-detailed.md` 教程 3
- **Av3Emulator**: 测试 Toggle 行为，详见 `avatar-30-emulator.md`

### 常见问题

| 问题 | 解决方案 |
|------|---------|
| 复制后 toggle 仍影响原 Avatar | 检查 FX/Menu/Params 是否完全独立 |
| 菜单中出现重复项 | 确认复制了完整资产（不是引用） |
| 玩家失去默认 Emotes | 详见 `expression-menu.md §9.6` |

---

## 工程管理最佳实践

### 预制体（Prefab）与预制体变体（Prefab Variant）

- 使用 Prefab 和 Prefab Variant 管理组件，模块化工程
- 使用 git 进行版本控制
- 使用 VCC/ALCOM 进行包管理

### 导出注意事项

- 导出 unitypackage 时不要导出公共依赖
- 导入时注意导入列表

---

## 相关工具

| 工具 | 用途 |
|------|------|
| [Avatar Pose Matcher](https://github.com/ColorlessColor/Avatar_Pose_Matcher) | 修复 Bike Pose |
| [VRC Head Chop](https://creators.vrchat.com/avatars/avatar-components/vrc-headchop/) | 隐藏头上装饰品/光环/眼镜 |

---

## 相关文档

- `avatar/ndmf-tools.md` — NDMF 工具生态
- `avatar/animator-system.md` — Animator 系统

---

## Unity 编辑器基础(86vrchat 视角)

> 来源: https://docs.vrcd.org.cn/books/86vrchat
> 状态: 通用知识 + 改模专有约束

### 核心快捷键

**[COMMUNITY-86vrchat]** 改模常用快捷键:

| 快捷键 | 功能 |
|--------|------|
| F | 快速定位场景物品 |
| Ctrl+D | 复制 |
| Ctrl+S | 保存场景 |
| Ctrl+Z / Ctrl+Y | 撤销/重做 |
| W / E / R | 移动/旋转/缩放 |
| 方向键 ↑↓ | 在选中物品间移动 |
| 方向键 ←→ | 展开/折叠当前层级 |
| Alt + 方向键 | 展开/折叠全部子级 |

### 透视 vs 正交投影

**[FACT]** Unity 两种投影模式:

| 模式 | 特点 | 用途 |
|------|------|------|
| **透视投影** | 远处物体变小,模拟真实视觉 | 场景预览 |
| **正交投影** | 物体大小保持不变 | **推荐改模编辑** |

**[COMMUNITY-86vrchat]** **大部分情况下更推荐正交投影进行编辑**
(来源: https://docs.vrcd.org.cn/books/86vrchat)

### Play Mode 注意事项

**[FACT]** Unity Play Mode 特性:

- 进入播放 → 退出播放 = 回到播放前状态
- **不能在播放模式下存档**
- Play 模式下 Ctrl+C → 退出 → Ctrl+V 可带出内容

**[COMMUNITY-86vrchat]** **改模大部分操作在场景(Scene)进行,游戏(Game)窗口可放一边**
(来源: https://docs.vrcd.org.cn/books/86vrchat)

### 资源 GUID 与 meta 机制

**[FACT]** Unity 资源引用机制:

- **每个资源文件导入时生成唯一 GUID**
- GUID 记录在 `.meta` 文件中
- 重命名/移动资源时 GUID 保持不变
- 场景/Prefab 基于 GUID 引用,而非路径

**[COMMUNITY-86vrchat]** **在 Unity 外(资源管理器)改文件名 = 引用断裂**
(来源: https://docs.vrcd.org.cn/books/86vrchat)

### 路径/命名约束(VRChat 专有)

**[COMMUNITY-86vrchat]** VRChat 创作者必须遵守:

| 路径/命名 | 约束 | 原因 |
|----------|------|------|
| 场景文件 | ✅ 可在 `Assets/` 内用中文 | Unity 兼容 |
| 工程名/路径 | ⚠️ 优先无中文 | 低概率"神秘问题" |
| **电脑名** | ❌ **不能含中文** | **上传必然失败** |

(来源: https://docs.vrcd.org.cn/books/86vrchat)

---

## 最简 Avatar 流程(一图流)

> 来源: https://docs.vrcd.org.cn/books/86vrchat
> 适用: 最低上传要求的 Avatar

### 前置要求

**[FACT]** 必备组件:

- **lilToon 着色器**(VPM: `jp.lilxyzw.liltoon`)
- 透明 PNG/PSD 角色图

### 制作步骤

```
1. Assets/ 下创建文件夹(可中文)
2. 拖入 PNG/PSD 资源
3. 图片处理: 勾选 "Alpha is Transparent" + Apply
4. 场景中创建 3D Object → Plane 作为载体
5. 创建 Material → Shader = lilToon
6. 渲染模式: Transparent
7. 贴图(PSD/PNG)导入材质
8. 材质拖到 Plane
9. 平面旋转 90° 让角色朝前
10. 上提让脚踩地平面
11. 添加 VRC Avatar Descriptor 组件
12. 填入必要参数 → 完成
```

### 关键点

| 步骤 | 必需值 |
|------|-------|
| 图片导入 | Alpha is Transparent = ✅ |
| 材质着色器 | lilToon |
| 渲染模式 | Transparent |
| 必备组件 | VRC Avatar Descriptor |

(来源: https://docs.vrcd.org.cn/books/86vrchat)

---

## ⭐ 模型导入工作流(2026-07-04 补充)

> 本节描述 VRChat Avatar 模型导入 Unity 工程的标准工作流
> 来源:VRCD 视频蒸馏(散落多个视频) + Unity 官方文档

### 核心规则(视频反复强调)

| 规则 | 说明 |
|------|------|
| **所有模型/衣服/头发/配件必须是 `.prefab` 格式** | 不能是 fbx、png |
| **unitypackage 必须拖入 Unity 的 Assets 文件夹**(不是场景) | 这是新手最大错误 |

> 🔴 **关键约束**:拖入场景而非 Assets 文件夹会导致资源无法被正确引用,需重新导入。

### Prefab 选择

- **改模推荐用"裸体版本"**(无自带衣服)
  - 优势:不会被自带衣服遮挡,便于后续添加衣服
- **带衣服版本**
  - 需要手动删衣服
  - 适用:无需换衣服的快速改模

### 导入方式

| 方法 | 操作 | 适用 |
|------|------|------|
| **方法 1** | Windows 资源管理器直接拖入 Assets 文件夹 | 通用(推荐) |
| **方法 2** | Show in Explorer 打开文件夹,放入后 Unity 自动检测 | 已在 Unity 内操作时 |

**推荐流程**:

```
Step 1: 打开 Unity 工程
Step 2: 在 Project 面板右键 → Show in Explorer
Step 3: 在打开的资源管理器中,放入 .prefab / .unitypackage
Step 4: 回到 Unity,等待自动导入完成
Step 5: 在 Project 面板中验证资源已就位
```

### 材质资源兼容性

#### "紫色" = 缺 shader

- **诊断信号**:Avatar 在 Scene 视图呈现紫色
- **根因**:Shader missing(通常是 lilToon / Poiyomi 等未导入)
- **解决**:先导入对应 Shader 包的 .unitypackage,再导入模型

#### 兼容性检查清单

- [ ] lilToon Shader 包已导入(最常见)
- [ ] Poiyomi Shader 包已导入(部分模型使用)
- [ ] Avatar SDK 包已导入(VRChat 必需)
- [ ] Modular Avatar 已安装(可选,推荐)

### 视频来源

- 散落多个视频 — 蒸馏产物见 `../../../../（见源库 raw/ 目录）参考文献/笔记/avatar-video-consolidated-2026-07-04/KP11-模型导入工作流.md`

---

## ⭐ SDK 面板诊断流程(2026-07-04 补充)

> 本节详细描述 VRChat SDK 4 个核心 Tab 的功能,补充"诊断流程"和"Auto Fix 关键规则"
> 来源:VRCD 视频蒸馏(video_5) + VRChat 官方文档

### Authentication Tab

- **登录凭据**:VRChat 用户名 / 邮箱(**不是** Steam 账号)
- **Steam 用户注意**:需先在 VRChat 官网绑定 Steam 账号
- **绑定前警告**:保存好友名(绑定过程可能删除好友列表)
- **登录后显示信息**:
  - 游戏内显示名
  - 上传权限(World Creator Status / Avatar Creator Status)

### Builder Tab

#### 基础字段

| 字段 | 说明 | 默认值 |
|------|------|--------|
| Selected Avatar | 选择要构建的 Avatar | 只有一个时自动选 |
| Name | Avatar 显示名 | 必填 |
| Visibility | 可见性 | **Private** |
| Style | 风格标签 | - |
| Content Warning | 内容警告 | 默认(错设可能封号) |
| Tags | 标签 | - |
| Image | 封面图(PNG) | 推荐 1200×900 |
| Capture In Scene | 用当前 Scene 视图作封面 | - |

#### Visibility 详解

- **Private(默认)**:仅自己可见
- **Public**:其他玩家可搜索并复制(Clone)
- ⚠️ **Content Warning 错设警告**:可能触发账号处罚

### ⭐ Build Type 关键区分

| Build Type | 行为 | 适用场景 |
|------------|------|---------|
| **Build And Publish** | 上传到 VRChat 服务器 | 正式发布 |
| **Build And Test** | 本地构建(不进游戏看不到) | 测试 |

#### 诊断技巧(关键)

```
Publish 失败 → 用 Test
  ├─ Test 成功 = 网络问题(检查 VRChat 账号状态 / 网络)
  └─ Test 也失败 = 模型问题(检查模型 / 编译错误)
```

### ⭐ Auto Fix 规则(关键)

| 错误类型 | 视觉特征 | 处理方式 |
|---------|---------|---------|
| **可自动修复** | 🔴 红色六边形 | ✅ 点 Auto Fix 按钮 |
| **不可自动修复** | 🟡 黄色 / ⚪ 白色 | ❌ 不要点 Auto Fix |

> ⚠️ **不要点非红色六边形的 Auto Fix**,可能破坏模型结构。
> 黄色/白色错误通常需要手动修复(如 shader 缺失、参数冲突等)。

### Content Manager Tab

- **功能**:管理已上传 Avatar
- **核心操作**:
  - **Copy ID**:复制 blueprint ID(用于跨工程恢复)
  - **Make Public / Private**:修改可见性
  - **Delete**:删除(不可恢复)
  - **Open On Web**:在 VRChat 网站打开
  - **Fetch**:刷新列表

#### Copy ID 跨工程恢复流程

```
1. 在 Content Manager 找到目标 Avatar
2. 点击 Copy ID(获得 blueprint ID)
3. 在新工程的 Avatar Descriptor 中粘贴 ID
4. 关联恢复(同步上传历史 / 状态)
```

### Settings Tab

- **用途**:仅开发者用
- **常规用户**:不需要修改

### 完整上传诊断流程(整合)

```
Step 1: 准备
  ├─ 选中 Avatar GameObject
  ├─ 确认 VRC Avatar Descriptor 配置完整
  └─ 检查 NDMF 工具无错误(AAO / MA 等)

Step 2: 登录
  └─ Authentication Tab → 登录 VRChat 账号

Step 3: 填写 Builder
  ├─ 选 Avatar + Name + Visibility
  ├─ 设置 Image
  └─ **Content Warning 用默认**

Step 4: 第一次 Build → Build And Test
  ├─ Test 成功 → 进入 Step 5
  └─ Test 失败 → 看 Auto Fix 规则
      ├─ 红色六边形 → 点 Auto Fix
      └─ 黄色/白色 → 手动修复

Step 5: 第二次 Build → Build And Publish
  ├─ Publish 成功 → 结束
  └─ Publish 失败
      ├─ Test 也失败 → 重新 Step 4
      └─ Test 成功 → 网络问题(检查账号 / 网络)

Step 6: Content Manager
  └─ 找到刚上传的 Avatar → Copy ID 保存
```

### 参考资料

- [L1 官方] VRChat Content Manager: https://docs.vrchat.com/docs/vrc-content-manager
- [L1 官方] VRChat Avatar Creator Status: https://docs.vrchat.com/docs/avatar-creator-status
- [L1 官方] VRChat SDK 文档: https://docs.vrchat.com/docs/
- [L1 官方] VRChat Avatar Performance: https://creators.vrchat.com/avatars/avatar-performance-ranking-system/

### 视频来源

- video_5 (6:50, 102 帧) — 蒸馏产物见 `../../../../（见源库 raw/ 目录）参考文献/笔记/avatar-video-consolidated-2026-07-04/KP05-VRChat-SDK-面板诊断流程.md`

---

## 扩展阅读(2026-07-04 新增)

- `../sources/blender-preprocessing.md` — Blender 预处理工具链(CATS + VRM)
- `../platform/unity-install-cn.md` — Unity Hub 国内安装指南
- `../platform/backup-strategy.md` — 创作者备份策略

---

## CATS 硬性要求(2026-07-04 来自 yexcadocs)

> 来源:yexcadocs §MMD Workflow(P1)
> 本节为社区实操经验补充。

### ⚠️ CATS 识别模型的硬性要求

- **Body mesh 必须命名为 "Body"**
- **表情(Blend Shape)必须放入 Body mesh**
- 否则 CATS 不能正确识别模型

### Dynamic Bone 参数参考表

> ⚠️ **Dynamic Bone 已弃用**,应迁移到 PhysBones
> 以下为多创作者经验值,供历史模型迁移参考

| 部位 | Damping | Elasticity | Stiffness | Inert |
|------|---------|-----------|-----------|-------|
| 裙子 | 0.6 | 0.06 | 0.2 | 0.4 |
| 胸部 | 0.08 | 0.06 | — | — |
| 头发 | 0.7 | 0.06 | 0.6 | 0.85 |

迁移到 PhysBones 后,参数语义有所变化,详见 `avatar-dynamic-bone-limits.md §9`。

---

## SDK2 → SDK3 升级(2026-07-04 来自 yexcadocs)

> 来源:yexcadocs §SDK2→SDK3(P2)
> 本节为社区实操经验补充。

### 简化流程

1. 升级前删除**最后两个组件**(SDK2 描述等)
2. 仅保留 **Transform + Animator**
3. 切换 Mode 设置

### 背景

旧模型(SDK2)需要升级到 SDK3 才能继续使用。常见错误信息:
- `A VRCAvatarDescriptor is required to build an Avatar`
- 上传面板 SDK 警告:Model uses legacy SDK

详细上传流程见 `optimization-guide.md §上传流程`。

---

## Blender 换头工作流(2026-07-04 来自 yexcadocs)

> 来源:yexcadocs §Blender 换头(P2)
> 本节为社区实操经验补充。

### 准备工作

1. 将两个模型文件放入同一文件夹
2. 如果有重名文件可重命名
3. 参考贴图问题进入软件后选择相应贴图

### Blender 基本操作

| 操作 | 快捷键 |
|------|--------|
| 旋转视角 | 鼠标中键按住 |
| 放大/缩小 | 鼠标滑轮 |
| 移动视角 | Shift + 鼠标中键按住 |
| 上下移动 = 放大/缩小 | CTRL + 鼠标中键按住 |

### 完整换头步骤

#### 1. 导入第一个模型(头部)

- 导入后若是 MMD 模型请 `Fix Model`(非 MMD 跳过)
- 按 `H` 隐藏选中骨骼,方便观察
- 点击 `CATS` → `Model Option` → `Separate by` → `材质` 分离材质

#### 2. 分离头部

- 直接将身体部分删除(若分离)
- 若连一起:选衣服/身体 → TAB 进入编辑模式 → CTRL+L 选连接面 → X 删除顶点 → TAB 退出
- 骨骼保留至 `Neck`

#### 3. 导入第二个模型(身体)

- 重复分离步骤,去掉头部
- 保留 Neck 以下骨骼

#### 4. 接头

- 物体模式下 Shift 选材质 → CTRL+J 合并
- 移动头部至大致位置
- 选中骨骼 → G → X/Y/Z 移动

#### 5. 合并

- `CATS` → `Custom Model Creation`
- Base/基础 = 头部
- To Merge = 身体
- 或已按顺序则直接 `Merge Armatures`

#### 6. 分离脖子

- 分离材质
- 选中头部 → TAB 进入编辑模式
- 选一圈脖子 → C 键拖拽选择
- P → 选中项 分离
- ⚠️ **必须删除分离出的脖子上的形态键**(否则破坏 Blend Shape)

#### 7. 接面

- 物体模式 Shift 选两个 → CTRL+J 合并
- TAB 进入编辑模式
- 选两顶点 → M → 到中心
- 右击 → 平滑顶点(粗细)
- 若颜色差异大,进 Photoshop 或纹理绘制

#### 8. 测试

- 选中骨架 → 左上姿态模式
- R → X/Y/Z 旋转骨骼测试
- 注:一般权重是对的,Unity 会变正常

#### 9. 权重(可选)

- 物体模式选骨架 + Shift 选需要刷的部分
- 权重绘制模式
- CTRL + 鼠标左键选骨骼绘制权重

### 关键原则

| 原则 | 说明 |
|------|------|
| **骨骼保留至 Neck** | 头部换身体时,身体部分删除但骨骼保留至 Neck |
| **删除形态键** | 分离出的脖子必须删除形态键 |
| **一般不需要重刷权重** | CATS 自动处理,Unity 验证即可 |

### 写写字笔(VRLabs Marker)

- 工具:VRLabs Marker | マーカー
- BOOTH:https://booth.pm/zh-cn/items/2911163
- **需要至少 13 bits 数值可用**
- 菜单至少 1 个可用位
---

## 相关页面

[[ndmf-tools.md]] · [[animator-system.md]] · [[bone-binding-workflow.md]] · [[blend-shape-troubleshooting.md]] · [[avatar-particle-system-creation.md]] · [[ma-component-cards.md]] · [[ma2bt.md]] · [[modular-avatar-tutorials-detailed.md]] · [[expression-menu.md]] · [[avatar-30-toggle-tools.md]] · [CATS Blender 插件（GitHub）](https://github.com/absolute-quantum/cats-blender-plugin) · [[../platform/unity-install-cn.md]] · [[../platform/backup-strategy.md]]
