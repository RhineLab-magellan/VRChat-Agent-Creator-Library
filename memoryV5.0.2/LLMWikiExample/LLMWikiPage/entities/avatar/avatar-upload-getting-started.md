---
title: "Avatar 上传入门指南(2026)"
category: avatar
knowledge_level: applied
status: active
source: "VRCD 文档库 + VRCLibrary How to Upload an Avatar (Iconoclass) + VRChat 官方文档"
source_type: community
version: 1.0
last_review: 2026-07-04
confidence: High
tags:
  - avatar
  - getting-started
  - upload
  - rig
  - shader
  - thumbnail
aliases:
  - "Avatar 上传入门"
  - "How to Upload an Avatar"
  - avatar-upload-getting-started.md
related:
  - avatar-modding-guide.md
  - optimization-guide.md
  - blender-shape-keys.md
  - shader/index.md
  - trust-rank.md
  - free-avatar-resources.md
  - ../platform/backup-strategy.md
type: entity
created: 2026-07-04
sources: "VRCD 文档库 + VRCLibrary How to Upload an Avatar (Iconoclass) + VRChat 官方文档"
updated: 2026-07-04
---

# Avatar 上传入门指南(2026)

> **适用**: VRChat Avatar 创作者新手(从 0 到可上传)
> **来源**: VRCLibrary "How to Upload an Avatar" (Iconoclass) + VRChat 官方文档
> **状态**: 2026-07-04 v1.0
> **P 等级**: P1 (结合 VRChat 官方 + Unity 官方 + 社区验证)

---

## §1 前置条件

### 1.1 系统要求
- **Windows 10/11 64-bit**(VCC 仅支持完整功能)
- Mac/Linux 用户: VCC CLI 有部分功能, 但不完整

### 1.2 VRChat 账户类型 ⚠️ 关键

> [FACT] **来源**: https://docs.vrchat.com/docs/frequently-asked-questions
> **VRChat 官方明确**: "Steam, Oculus and Viveport Accounts will never gain the ability to upload content."

| 账户类型 | 是否可上传 | 识别方法 |
|---------|----------|---------|
| **VRChat 正式账户** | ✅ 可 | 名字后**不会**带 4 个随机字母和数字 |
| Steam 账户 | ❌ 不可 | 需转换为正式账户 |
| Oculus 账户 | ❌ 不可 | 需转换为正式账户 |
| Viveport 账户 | ❌ 不可 | 需转换为正式账户 |

**转换方法**: 登录 VRChat 网站创建新的正式账户 → 通过 Settings → 转换

### 1.3 信任等级

> [FACT] **来源**: https://help.vrchat.com/hc/en-us/articles/360062658813 + https://wiki.vrchat.com/wiki/Trust_Rank
> **官方明确**: "In order to upload content or use avatar pedestals in private worlds, the New User rank is required."

- 必须是 **New User**(蓝色身份)及以上
- Visitor(访客)等级**不能**上传任何内容
- 升级方法: 多在 VRChat 中活动、交朋友、遵守社区准则

详见: [[entities/avatar/trust-rank]]

### 1.4 工程管理建议(创作者最佳实践)

> [COMMUNITY] **来源**: VRCLibrary How to Upload an Avatar + 社区经验

- **每 Avatar 独立 Unity 项目**: 崩溃时只丢失一个 Avatar 的进度
- **存储在容量最大的硬盘**: Unity 项目可能非常大
- **专用文件夹结构**: `VRChat Projects/<AvatarName>/`

---

## §2 工具准备

### 2.1 VCC 下载与安装

> [FACT] **来源**: https://vcc.docs.vrchat.com/

- **官方下载**: https://vrchat.com/home/download
- **直接下载**: https://vrchat.com/download/vcc
- **安装路径**: `%LOCALAPPDATA%\Programs`(默认)
- **平台支持**: Windows 10/11 64-bit 完整功能, Mac/Linux CLI 部分

### 2.2 Unity 版本绑定 ⚠️ 关键警告

> [FACT] **来源**: https://creators.vrchat.com/sdk/upgrade/current-unity-version/
> **官方明确**: "Upgrading your version will result in content not loading once uploaded to VRChat - make sure to stay on our recommended version."

| 项目 | 推荐值 | 警告 |
|------|--------|------|
| **Unity 版本** | **2022.3.22f1**(唯一) | ⚠️ 升级会导致内容无法加载 |
| **2025-10 安全漏洞** | **不适用 VRChat 项目** | 留在 2022.3.22f1 是安全的 |
| **Unity Hub 警告** | 可安全忽略 | 不影响 VRChat 项目 |

### 2.3 VCC 创建项目流程

1. 打开 VCC → 自动检测 Unity 是否安装
2. **New** → **Avatar** → 输入项目名 + 位置
3. **Create** → VCC 自动配置 Unity 项目
4. 在 VCC 项目列表中点击 **Open Project** 打开

---

## §3 资源导入

### 3.1 Avatar 资源形式分类

> [COMMUNITY] **来源**: VRCLibrary + 整合

| 资源形式 | 图标/特征 | 处理方式 |
|---------|----------|---------|
| **设置好的场景** | 含已配置 Avatar 的 `.unity` | 直接使用该场景 |
| **Prefab** | 蓝色方框图标 | 拖入场景即可 |
| **裸 FBX + 纹理** | 仅模型 + 材质 | 需手动配置(见 §4) |
| **完全无场景** | 无文件 | 右键 `Create > Scene` 创建 |

### 3.2 资源导入 3 种方式

1. **拖拽法**: 从 Windows 资源管理器拖到 Project 窗口
2. **菜单法**: `Assets > Import custom Assets > Custom package`
3. **VCC 仓库法**: 通过 VPM 列表一键安装(推荐)

### 3.3 Avatar SDK 内置示例

> [FACT] **来源**: VRChat SDK 包结构
> VRChat Avatar SDK 自带 **Physbones 示例 Avatar**

- 路径: VCC 左下角 → `packages` → `VRChat` → `Examples` → `Physbones`
- 用途: **作为上传测试的标准参考 Avatar**

### 3.4 免费 Avatar 资源

> [FACT] **来源**: 多源验证(详见 [[entities/avatar/free-avatar-resources]])

| 资源 | 协议 | 数量 | URL |
|------|------|------|-----|
| SCSS Wiki Avatar Models | (各模型独立) | 多 | https://gitlab.com/s-ilent/SCSS/-/wikis/Other/Avatar-Models |
| 100Avatars (ToxSam) | CC0 | 300+ | https://github.com/ToxSam/osa-gallery |
| VRChat Asset Browser | (各资源独立) | 数百 | https://github.com/vanquish3r/vrchat-asset-browser |
| Public Use Permissions Avatars | (各模型独立) | 持续更新 | https://gitlab.com/vrchat6114543/public-use-permissions-avatars |

⚠️ **使用前务必检查每个 Avatar 的 TOS/TOU**

详见: [[entities/avatar/free-avatar-resources]](独立文档)

---

## §4 FBX 骨骼配置(Humanoid)

### 4.1 Rig = Humanoid

> [FACT] **来源**: https://creators.vrchat.com/avatars/rig-requirements/

1. 选中 FBX 文件
2. Inspector → **Rig** 选项卡
3. **Animation Type** → **Humanoid**
4. 点击 **Configure** 进入骨骼映射

### 4.2 VRChat 必需骨骼清单 ⚠️ 关键

> [FACT] **来源**: https://creators.vrchat.com/avatars/rig-requirements/

#### 必需映射(VRChat 强制)

| 骨骼 | Body 选项卡位置 | 必需性 |
|------|---------------|-------|
| **Pelvis** | Body | ✅ 必需 |
| **Spine** | Body | ✅ 必需 |
| **Chest** | Body | ✅ 必需(Mecanim 可选, VRChat 必需) |
| **Neck** | Head | ✅ 必需(Mecanim 可选, VRChat 必需) |
| **Left/Right Shoulders** | Left/Right Arm | ✅ 必需 |
| **Head** | Head | ✅ 必需 |
| **Left/Right Hands** | Left/Right Arm | ✅ 必需 |
| **Left/Right Feet** | Left/Right Leg | ✅ 必需 |

#### 可选/不要映射

| 骨骼 | 备注 |
|------|------|
| **Upper Chest** | ❌ **不要映射**!会导致 IK 问题 |
| **Toes(脚趾)** | ⚪ 可选 |
| **Jaw(下巴)** | ⚠️ 见 §4.4(不应分配) |

#### 父级关系(IK 工作前提)

> [FACT] **官方明确**: "your shoulder bones... must be direct children of your chest bone... Also, the neck bone... must also be a direct child of the Chest."

- **Shoulders** 必须是 **Chest** 的直接子级
- **Neck** 必须是 **Chest** 的直接子级

### 4.3 Chest 手动配置

如果模型未自动配置 Chest:
1. 在 Chest 旁边的框中
2. 单击圆圈(打开骨骼选择器)
3. 搜索 **"chest"**
4. 选中匹配骨骼 → 应用

### 4.4 Head 骨骼配置 + Jaw 取消 ⚠️ 关键

> [FACT] **来源**: http://vrchat.wikidot.com/tutorial:avatars:unity-upload-tutorial + VRChat Rig Requirements

#### Head 必须分配的骨骼
- 除 **Jaw** 外的所有 Head 骨骼都应被分配
- **Neck** 必填(已在 §4.2)

#### Jaw 不应分配(常见错误)

> [FACT] **VRChat Wiki 明确**: "Assigning a eye bone to the jaw bone... The first problem can easily be fixed by clearing the jaw bone assignment."

- 许多模型**会自动**将错误骨骼分配给 Jaw(常见错误!)
- **Jaw 在 VRChat 是可选的**(不强制)
- 如分配错误: **必须清除** Jaw 分配
- 清除方法: 点击 "jaw" 旁边的绿色圆圈 → 按 **Delete** 键

#### Viseme 配置方式(替代 Jaw)

> [FACT] **来源**: https://creators.vrchat.com/avatars/

VRChat 支持 2 种 viseme 方式:
1. **Jaw-flap bone** - 在 Rigging Configuration Screen 分配 jaw-flap 骨骼
2. **Blendshapes** - 通过 Blend Shape 定义 viseme(更灵活)

### 4.5 验证与保存

1. 检查所有骨骼映射
2. **Done** → **Apply**

---

## §5 材质与着色器

### 5.1 Extract Materials 步骤

> [FACT] **来源**: https://docs.unity3d.com/Manual/FBXImporter-Materials.html

1. 选中 FBX 文件
2. Inspector → **Materials** 选项卡
3. 点击 **Extract Materials** 按钮(将材质提取到项目)
4. 必要时点击 **Extract Textures** 按钮(提取纹理)

**注意事项**:
- 按钮在无子资源时灰显
- Unity 2017.2+ 默认 **Use Embedded Materials**
- Legacy 模式: Use External Materials(自动提取为外部资产)

### 5.2 纹理应用到 Material

1. 点击 Material 资产
2. **Albedo** / **Base Color** 槽位 → 拖入纹理

### 5.3 Shader 选型

> [FACT] **来源**: https://creators.vrchat.com/platforms/android/quest-content-limitations + https://wiki.vrchat.com/wiki/Shaders

| Avatar 风格 | 推荐 Shader | 平台 |
|------------|------------|------|
| 写实 | **Standard** (Unity 默认) | PC |
| 卡通(基础) | **VRChat/Mobile/Toon Lit** | PC + Quest 兼容 |
| 卡通(推荐) | **VRChat/Mobile/Toon Standard** | PC + Quest 兼容 ⭐ |
| 高级效果 | lilToon / Poiyomi / SCSS 等 | 视 Shader 而定 |

#### Toon Lit vs Toon Standard ⚠️ 时效性

| 特性 | Toon Lit(老) | Toon Standard(新, 2025-04-24) |
|------|------------|---------------------------|
| Detail Maps | ❌ | ✅ |
| Emission | ❌ | ✅ |
| Rim Light | ❌ | ✅ |
| Hue Shift(动画) | ❌ | ✅ |
| Custom Shadow Ramps | ❌ | ✅ |
| Matcap | ❌ | ✅ |
| 平台 | 全部 | 全部 |
| 推荐度 | 兜底 | ⭐ **首选** |

> [FACT] **VRChat 官方 Dev Update (2025-04-24)**: "Coming to an SDK near you very soon, we will have a new shader named Toon Standard... provide a powerful baseline that can run well on all hardware configurations that VRChat supports."

详见: `[[entities/avatar/shader/index]] §9`

---

## §6 上传流程

### 6.1 SDK Control Panel

> [FACT] **来源**: https://creators.vrchat.com/getting-started/

1. 顶部菜单 → `VRChat SDK` → `Show Control Panel`
2. 登录 VRChat 账户
3. 切换到 **Builder** 选项卡

### 6.2 ThumbnailCapture / VRCCam(Avatar 缩略图)⚠️ 时效性

> [FACT] **来源**: https://creators.vrchat.com/avatars/creating-your-first-avatar/ + https://feedback.vrchat.com/sdk-bug-reports/p/sdk-382-unity-crash-upon-saving-scene-after-using-camera-for-thumbnail

#### 当前(2026 推荐): ThumbnailCapture 组件

**重要**: VRCCam 已被替换为 `ThumbnailCapture` 组件(2025-08 后)

- 标准: **4:3 宽高比** 缩略图
- **方法 A(简单)**: 在上传窗口直接选择图片文件 / 从 Unity 场景捕获
- **方法 B(高级)**: 放置 `GameObject > 3D Object > Plane` + `Sharky's VRC Avatar Utils > Align Upload Camera` 组件
  - 工具 URL: https://sharkys-vrc-avatar-utils.readthedocs.io/

#### 旧版(参考): VRCCam

- 旧 SDK 行为: VRCCam 是在场景中的对象, 上传前自动创建
- 拖动 VRCCam 可改变相机位置
- Inspector 可调整 FOV、背景、纯色
- 滴管工具可从 Avatar 拾取背景色

### 6.3 滴管工具取色技巧

- 选中背景 → **Color Picker** 工具
- 从 Avatar 上拾取颜色
- 微调得到与 Avatar 协调的纯色背景

### 6.4 Build and Publish

1. **Builder** 选项卡
2. 选择 Avatar GameObject
3. 填写 Name / Description / Tags
4. 选择缩略图(4:3)
5. **Build & Publish** → Unity 编译 → 弹出上传窗口
6. 确认 → **Upload**

### 6.5 Build & Test 模式(无需上传测试)

> [FACT] **来源**: https://creators.vrchat.com/avatars/creating-your-first-avatar/

- 切换构建类型到 **Build & Test Your Avatar**
- 本地构建后, Avatar 出现在 "SDK Test Avatars" 部分
- **仅自己可见**, 适合上传前测试

---

## §7 组件说明

### 7.1 Pipeline Manager(Blueprint ID)

> [COMMUNITY] **来源**: VRCLibrary + SDK 文档

- 包含 Avatar 的 **Blueprint ID**(唯一标识)
- 标识 Avatar 是"你的 VRChat Avatar"
- 上传后自动填充

### 7.2 VRC Avatar Descriptor(核心组件)

- 包含 Avatar 的**所有重要部分**
- 上传**必须**有此组件, 否则 SDK 报错

### 7.3 Pumkin's Avatar Tools(快速设置)

- 仓库: `https://rurre.github.io/vpm/`
- **Quick Setup Avatar** 一键添加 Pipeline Manager + VRC Avatar Descriptor
- 通过 VCC 一键安装

---

## §8 关联文档

- [[entities/avatar/avatar-modding-guide]] - VRChat Avatar 改模基础知识
- [[entities/avatar/optimization-guide]] - Avatar 最佳化实操指南
- [[entities/avatar/blender-shape-keys]] - 形态键 + Rig 配置
- [[entities/avatar/shader/index]] - Shader 知识库
- [[entities/avatar/trust-rank]] - Trust Rank 体系
- [[entities/avatar/free-avatar-resources]] - 免费 Avatar 资源索引
- [VCC 文档（官方）](https://vcc.docs.vrchat.com/) - VCC 工具
- [[entities/platform/backup-strategy]] - 创作者备份策略

## §9 已知限制与时效性

- **VCC vs ALCOM**: 2026 起, ALCOM 是部分预发布工具的推荐选择; VCC 仍受支持
- **VRCCam → ThumbnailCapture**: 旧 VRCCam 已被替换(2025-08 后)
- **Toon Standard 替代 Toon Lit**: 新 Toon Shader 主力(2025-04-24)
- **Pumkin 工具**: 2026 年活跃度需持续验证
- **教程完整性**: 本文档基于 Iconoclass 2023 年教程, 部分内容由 Sippbox 视频补充

---

## 相关页面

[[avatar-modding-guide.md]] · [[optimization-guide.md]] · [[blender-shape-keys.md]] · [[shader/index.md]] · [[trust-rank.md]] · [[free-avatar-resources.md]] · [VCC 文档（官方）](https://vcc.docs.vrchat.com/) · [[../platform/backup-strategy.md]]
