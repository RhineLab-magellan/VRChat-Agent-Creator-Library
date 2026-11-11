---
title: "Avatar 端摄影工具索引(Avatar Camera Plugins)"
category: hybrid
knowledge_level: applied
status: active
source: 本地知识库整理 + BOOTH 官方(goat-cannery.booth.pm) + VRCD 文档库(docs.vrcd.org.cn)
source_type: community
version: 1.0
last_review: 2026-07-04
confidence: Medium
tags:
  - hybrid
  - avatar
  - camera
  - photography
  - modular-avatar
  - fisheye
  - recommended-tools
aliases:
  - "Avatar Camera Plugins"
  - "Flex Fisheye Lens"
  - "鱼眼镜头"
  - "Avatar 摄影工具"
  - "FlexFishEye"
related:
  - udon-world-plugins.md
  - index.md
  - audio-link.md
  - ../../entities/avatar/modular-avatar.md
  - ../../entities/avatar/shader/index.md
  - osc-protocol.md
type: concept
created: 2026-07-04
sources: 本地知识库整理 + BOOTH 官方(goat-cannery.booth.pm) + VRCD 文档库(docs.vrcd.org.cn)
updated: 2026-07-04
---

# Avatar 端摄影工具索引(Avatar Camera Plugins)

> **Domain**: Hybrid(Avatar ↔ World 摄影增强工具)
> **类型**: 推荐工具索引(Avatar 端,与 `udon-world-plugins.md` 对应)
> **目标读者**: VRChat Avatar 创作者(增强 Avatar 端的摄影能力)
> **状态**: ✅ 已收录 1 个插件(Flex Fisheye Lens)
> **最后更新**: 2026-07-04

---

## ⚠️ 重要前置警告

> 🔴 **本类工具与 `udon-world-plugins.md` 的区别**:
> - `udon-world-plugins.md` = **World 端** Udon 工具(创作者安装在 World 中)
> - 本文档 = **Avatar 端** 摄影工具(玩家安装在 Avatar 中)
> - **典型代表**: Flex Fisheye Lens — 闭源,需要 Modular Avatar 集成到 Avatar

> 🔴 **平台限制**: 已收录的 Flex Fisheye Lens **仅支持 PC VR 模式**,桌面模式和 Quest 均不可用。

---

## §0 速查表(Quick Reference)

| 玩家最常问的问题 | 解答 | 跳转 |
|------------------|------|------|
| Flex Fisheye Lens 装上后菜单在哪? | Expressions 菜单 `[FishEyeLens]` + 轮盘菜单 `FishEyeCam` | [§2](#2-安装与配置installation) |
| 怎么拍照? | 启用 FishEyeON → 打开 VRC 原相机 → 拍摄(原相机需跟随玩家) | [§3](#3-使用流程usage-workflow) |
| 桌面模式能用吗? | ❌ **不能**,仅 PC VR 模式 | [§1.3](#13-平台与硬件限制) |
| Quest 能用吗? | ❌ **不能**,仅 PC VR 模式 | [§1.3](#13-平台与硬件限制) |
| 照片出现网格? | 720P 分辨率所致,改用 1080P+ | [§6 Q1](#q1-照片中会出现网格或高光吗) |
| 远景/Skybox 不可见? | 调整 `[Setting] → [Clipping Far]` | [§6 Q2](#q2-远处的物体和-skybox-不可见怎么办) |
| 物体莫名消失? | LOD 触发,超广角导致物体被切走 | [§6 Q3](#q3-为什么有些物体不可见) |
| 部分 Shader 渲染错误? | 2 相机架构与 UV 计算 Shader 不兼容 | [§6 Q4](#q4-一些着色器渲染不正确怎么办) |
| 怎么换相机外形? | Addon 文件夹有 2 个 prefab(GlobalShow/Local) | [§5.3](#53-更改相机外形) |
| 需要 Modular Avatar 吗? | ✅ 必需,否则无法安装 | [§2.1](#21-安装前置条件) |
| 升级后质量设置变了? | 升级副作用,需重新设置 | [§5.1](#51-调整拍摄质量) |

---

## §1 概述(Overview)

### 1.1 工具索引

| 插件名称 | 作者 | 价格 | 平台 | 状态 |
|----------|------|------|------|------|
| **Flex Fisheye Lens** | Goat-Cannery(山羊罐头厂) | ¥2,000 | PC VR Only | ✅ 已收录 |

### 1.2 Flex Fisheye Lens 简介

**Flex Fisheye Lens** 是装载在 Avatar 上的鱼眼相机插件,使用户拍摄的照片具有鱼眼镜头畸变效果。

**适用场景**:
- 拍摄较宽阔的场景
- 视野太狭窄时中和压迫感
- 创意作品(利用视角独特性)
- 视觉冲击力强的街头摄影
- "原图直出"即可获得鱼眼效果(无需后处理)

**分发信息**:
| 属性 | 值 |
|------|-----|
| **分发平台** | BOOTH |
| **链接** | `https://booth.pm/en/items/5512392` |
| **日文链接** | `https://booth.pm/ja/items/5512392` |
| **首次发布** | 2024-02-21 |
| **售价** | ¥2,000 |
| **渲染管线** | BRP(VRChat 兼容的 Built-in Rendering Pipeline)|

### 1.3 平台与硬件限制

> 🔴 **本节是核心警告,请务必完整阅读**

| 限制项 | 说明 |
|--------|------|
| **VR 模式强制** | 只能在 PC VR 模式下使用,桌面模式无法操作 |
| **Quest 不支持** | 移动端 Quest 完全不可用 |
| **内部相机数量** | 2 个高分辨率相机叠加 |
| **性能影响** | 召出鱼眼相机时游戏可能变卡 |
| **照片分辨率** | FullHD (1080P) 以上才能保证效果 |
| **分辨率陷阱** | 720P 拍摄时画面会显示网格线 |

> 💡 **设计优势**: 该插件采用 **Local Only** 模式,鱼眼效果完全本地处理,**零网络同步开销**。

### 1.4 关键术语对照表

| 术语 | 英文 | 含义 |
|------|------|------|
| 鱼眼 | FishEye | 凸透镜式广角镜头,边缘畸变 |
| 凹透镜 | Reverse FishEye | 反向效果(菜单中可切换) |
| 折射率 | Distortion | 畸变强度,可分别调整 X/Y 轴 |
| 景深 | Depth of Field / DOF | 焦点内外模糊效果(本插件不支持前景模糊) |
| 无人机模式 | Drone | 双手分离操控,单手移动 + 单手朝向 |
| 固定位置 | World Fix | 将相机固定到当前世界坐标 |
| 看向模型面部 | LookAtHead | 相机自动锁定 Avatar 面部 |

---

## §2 安装与配置(Installation)

### 2.1 安装前置条件

| 前置条件 | 说明 |
|----------|------|
| **Modular Avatar (MA)** | 🔴 **必需**,未导入会导致安装失败 |
| **MA 版本** | 建议使用最新版,旧版可能出现导入错误 |
| **Avatar 结构** | 标准 Humanoid Avatar(可正常处理 Prefab) |

> 📚 详见 [[entities/avatar/modular-avatar]] 了解 MA 完整知识。

### 2.2 安装步骤

1. 导入 `.unitypackage`
2. 在 Project 窗口定位到 `Assets/GoatCannery/FishEyeCam/`
3. 将 `FishEyeCam.prefab` 拖入 Avatar,作为子物体
4. (可选)左撇子用户改用 `FishEyeCam_Left` 预制体
5. 上传 Avatar

**安装成功标志**:
- Expressions 菜单中出现 `[FishEyeLens]` 选项
- 轮盘菜单(Radial Menu)中出现 `FishEyeCam` 菜单

### 2.3 示例资源

| 资源类型 | 链接 | 用途 |
|----------|------|------|
| 示例 World | `wrld_44b14599-7011-48d3-b40f-41c0dc80252e` | 照片展览、试用 Avatar、拍摄点 |
| 试用 Avatar | `avtr_04cd67b2-3a7d-4a20-b322-17dd437221b8` | 功能受限的体验模型 |

### 2.4 社交传播要求

- 在 Twitter 等社交媒体发布使用该插件的内容时,需添加 `#FlexFishEye` 标签(作者要求)

### 2.5 与 Modular Avatar 的协同 [INFERENCE]

| 操作 | 是否需要 MA |
|------|--------------|
| 基础安装 | ✅ MA 是必需的 |
| 质量调整 | ❌ 直接编辑 Inspector |
| 位置调整 | ❌ 直接编辑 Hierarchy |
| 相机外形替换 | ❌ 直接编辑 Addon |
| 菜单自动生成 | ✅ MA 处理菜单挂载 |

**结论**: MA 在此插件中**只承担安装和菜单挂载**,运行期配置仍由原生组件管理。

---

## §3 使用流程(Usage Workflow)

### 3.1 完整操作链

1. 打开轮盘菜单 → 启用 `[FishEyeON]`
2. 打开 VRC 原相机(系统相机)
3. 🔴 **关键约束**: 原相机必须 **跟随玩家移动**(非飞行模式)
4. 拍摄照片(由 VRC 原相机保存到本地)
5. 关闭 `[FishEyeON]`

> ⚠️ **如果不满足第 3 步**,鱼眼相机的画面无法传递给 VRC 原相机,最终拍出的照片**不会包含鱼眼效果**。

### 3.2 镜头可见性

| 状态 | 是否可见 |
|------|----------|
| PC VR 模式 | ✅ 手上会显示鱼眼镜头模型 |
| 桌面模式 | ❌ 不可见(VR-only) |
| Quest | ❌ 不可用 |
| 其他人视角 | ❌ 鱼眼镜头仅本地可见(Local Only) |
| 照片中 | ✅ 鱼眼效果被拍摄下来 |

> 💡 "Local Only" 是关键设计: 不需要网络同步,完全本地处理,**零网络开销**。

### 3.3 调试工作流 [INFERENCE]

```
1. 安装插件 → 验证 Expressions 菜单出现 [FishEyeLens]
2. 召出 FishEyeON → 验证手上出现镜头
3. 选择 Far Camera → 预览原始画面
4. (可选) 调整 CameraTracker 位置
5. (可选) 调整 LookAtHead[Offset]
6. (可选) 替换相机外形
7. 拍摄测试照片 → 验证鱼眼效果
8. 调整 Preset / Manual 参数
9. 拍摄成品
```

---

## §4 菜单参考(Menu Reference)

### 4.0 菜单总览

插件共有 **6 个主菜单**:

| 菜单 | 全名 | 用途 |
|------|------|------|
| **FishEye ON** | FishEye ON | 鱼眼相机开关(主开关) |
| **P** | Preset | 进入预设菜单(常用场景预设) |
| **M** | Manual | 进入手动调整菜单(细粒度控制) |
| **Drone** | Drone | 进入无人机操控菜单(分离式操控) |
| **Behavior** | Behavior | 进入相机行为菜单(行为模式) |
| **Setting** | Setting | 进入设置菜单(全局参数) |

### 4.0.1 菜单架构图 [INFERENCE]

```
FishEyeCam (主菜单)
├── FishEye ON         ← 开关
├── P (Preset)         ← 预设模式(快速选择)
│   ├── 多种预设效果
│   ├── [Zoom]
│   └── [World Fix]
├── M (Manual)         ← 手动模式(细调)
│   ├── [Fov]
│   ├── [Zoom]
│   ├── [Switch Zoom Level]
│   ├── [Distortion]    ← 子菜单:折射率
│   ├── [SceenSize]
│   ├── [Depth of field]← 子菜单:景深
│   └── [World Fix]
├── Drone              ← 无人机模式
│   ├── [Drone]
│   ├── [Drone (RotateOnly)]
│   ├── [Fov] [Zoom] [Switch Zoom Level]
│   ├── [Auto Level]
│   └── [WorldFix (Drone Reset)]
├── Behavior           ← 行为模式
│   ├── [World Fix]
│   ├── [Auto Level]
│   ├── [LookAtHead]
│   ├── [HeadPos Move]
│   ├── [Color Adjust]
│   └── [Reverse FishEye]
└── Setting            ← 全局设置
    ├── [Grid]
    ├── [Clipping Far]
    ├── [Distoriton Check]
    ├── [Camera Culling OFF]
    └── [AspectRatio]
```

### 4.1 预设模式 P (Preset)

**定位**: 快速选择常用拍摄场景或对象的镜头效果预设

**使用方式**:
- 选择预设后,在原相机屏幕右下角会显示预设图标(短时间)
- 每种预设都试一下,挑选最满意的效果(不需要严格遵循预设分类)

**子选项**:

| 选项 | 英文 | 功能 |
|------|------|------|
| **缩放** | [Zoom] | 调整鱼眼镜头的缩放 |
| **固定位置** | [World Fix] | 将相机固定到当前世界位置 / 解除固定 |

### 4.2 手动模式 M (Manual)

**定位**: 细粒度手动调节镜头参数

| 选项 | 英文 | 功能 |
|------|------|------|
| 视角 | [Fov] | 调整相机视角(视野) |
| 变焦 | [Zoom] | 在不改变相机视角的情况下变焦 |
| 远距变焦 | [Switch Zoom Level] | 转换为远距离变焦 |
| 失真 | [Distortion] | 进入鱼眼折射率编辑菜单 |
| 画面大小 | [SceenSize] | 放大相机画面(画质可能降低) |
| 景深 | [Depth of field] | 进入景深菜单 |
| 固定位置 | [World Fix] | 固定在当前位置 |

> 💡 [SceenSize] 工作机制: 高亮部分画面处在可拍摄范围外,提示用户当前取景边界。

### 4.3 失真菜单 (Distortion)

**进入后**: 显示折射率指南,编辑数值时在拍摄范围外加亮光(可视化反馈)

| 选项 | 英文 | 功能 |
|------|------|------|
| 折射率倍数 | [Distortion Multiple] | 折射率整体倍数变更 |
| 折射率分轴 | [Distortion X&Y] | 折射率 x 轴 / y 轴分别变更 |
| 中心 | [Center] | 更改镜头的中心位置 |
| 凹透镜 | [Reverse FishEye] | 将鱼眼变成凹透镜(反向效果) |

### 4.4 景深菜单 (Depth of Field)

> ⚠️ **重要限制**:
> - 只支持焦点后方的模糊
> - **不支持前景模糊**(功能简单)

| 选项 | 英文 | 功能 |
|------|------|------|
| 景深开关 | [DOF ON] | 开启景深功能 |
| 焦点锁定 | [Focus Lock] | 固定对焦距离 |
| 光圈 | [Aperture] | 调整焦点对准的距离 |
| 模糊强度 | [BokehPower] | 调节模糊程度 |

**视觉反馈机制**:

| 区域 | 颜色 |
|------|------|
| 合焦区域 | **红色** |
| 即将失焦 | **蓝色** |
| 完全失焦 | (无标记) |

**调参技巧**:
- 如果能够确保蓝色部分不覆盖想要聚焦的区域
- 背景的模糊程度就会减少(更接近无景深效果)

### 4.5 无人机模式 (Drone)

**定位**: 双手分离操控的独立机位模式,适合大场景航拍式摄影

**核心操作模型**:
```
玩家左手 → 移动无人机位置
玩家右手 → 控制无人机摄像头方向
```

**进入/退出行为**:
- **进入**: 选择 `[Drone]` 进入无人机模式
- **退出**: 离开无人机模式后,相机将**固定在该位置**(不会回到玩家身边)
- **收回**: 选择 `[WorldFix (Drone Reset)]` 将无人机收回至玩家手中

**子选项**:

| 选项 | 英文 | 功能 |
|------|------|------|
| 无人机模式 | [Drone] | 进入无人机模式 |
| 仅旋转模式 | [Drone (RotateOnly)] | 不移动无人机,只操作方向(通过菜单旋钮开启) |
| 视角/变焦 | [Fov] [Zoom] [Switch Zoom Level] | 与手动模式相同 |
| 自动水平 | [Auto Level] | 控制相机倾斜(左/正/右固定) |
| 收回 | [WorldFix (Drone Reset)] | 收回至手中 |

**设计思想** [INFERENCE]:

| 维度 | 设计 |
|------|------|
| **机位分离** | 相机不再束缚在玩家身上,机位独立 |
| **空间感** | 通过双手分工实现第三人称操控 |
| **持久化** | 退出后位置固定,可作为"半永久机位" |
| **借鉴来源** | 25/10/01 更新: 借鉴 suzuki_ith 的 [Integral] 相机无人机操控系统 |

### 4.6 行为模式 (Behavior)

**定位**: 相机行为模式切换

| 选项 | 英文 | 功能 |
|------|------|------|
| 固定位置 | [World Fix] | 将相机固定在当前位置 |
| 自动水平 | [Auto Level] | 控制相机倾斜(左·正面·右固定) |
| 看向面部 | [LookAtHead] | 镜头对向自己的脸 |
| 微调方向 | [HeadPos Move] | 微调镜头的方向 |
| 颜色调整 | [Color Adjust] | 校正亮度 |
| 凹透镜 | [Reverse FishEye] | 将鱼眼变为凹透镜 |

### 4.7 设置菜单 (Setting)

**定位**: 全局参数配置,影响所有模式

#### 4.7.1 网格显示 (Grid)

| 选项 | 功能 |
|------|------|
| [Grid] | 选择各种网格显示 |
| 0% | 1k(1 千米) |
| (可调) | 不同百分比对应不同距离 |

#### 4.7.2 远景裁剪 (Clipping Far)

| 百分比 | 远景显示距离 |
|--------|--------------|
| 0% | 1 km |
| 50% | 10 km |
| 100% | 1000 km (100 万米) |

**用途**: 当 Skybox 或远处对象无法显示时,调整到更大数值

#### 4.7.3 变形检查 (Distortion Check)

- 在手持相机内显示镜头的变形情况
- 用于调试 / 创作时确认失真分布

#### 4.7.4 相机剔除关闭 (Camera Culling OFF)

- 显示所有物体(包括被剔除的)
- ⚠️ **性能警告**: 会显著让游戏变卡顿

#### 4.7.5 长宽比 (AspectRatio)

| 长宽比 | 用途 |
|--------|------|
| 16:9 | 标准宽屏 |
| 1:1 | 正方形 |
| 3:2 | 经典胶片 |
| 4:3 | 传统电视 |
| 2.35:1 | 宽银幕电影 |
| 1:√2 (白银比) | ⭐ 25/10/07 新增,银盐比例 |

**附加功能**:
- 照片外侧变黑色(或白色,开启 `[White Back]`)
- 注 1: 根据世界后处理设置,效果可能更适合黑色区域
- 注 2: 使用 VRC+ 打印功能时,打印的白色与照片内白色颜色不同

### 4.8 关键操作模式 [INFERENCE]

| 模式 | 适用场景 | 核心特性 |
|------|----------|----------|
| **预设模式 (P)** | 快速拍摄 | 一键应用,无需调参 |
| **手动模式 (M)** | 专业拍摄 | 完全控制 Fov、Zoom、失真、景深 |
| **无人机模式 (Drone)** | 大场景/特殊角度 | 双手分离操控,离开后位置固定 |
| **行为模式 (Behavior)** | 自拍/锁定 | LookAtHead、AutoLevel 等辅助 |

---

## §5 自定义与调整(Customization)

### 5.1 调整拍摄质量

**入口**: Hierarchy 窗口 → 点击 `FishEyeCam` → 右侧 Inspector

**可调参数**:
- `Select Preset`: 调整照片质量(预设档位)
- `Custom`: 自行调整 `Resolution Far` 和 `Resolution Near`

> 🔴 **关键约束**: 调整值务必为 **4 的倍数**!否则可能出渲染异常

**升级副作用**:
- 插件更新后,质量设置会**初始化**
- 更新后需要重新设置

### 5.2 调整相机与手的相对位置

**目标**: 控制镜头与玩家手部之间的距离

**操作步骤**:
1. 在 Hierarchy 中找到 `FishEyeCam - Tracer - Hand - CameraTracker`
2. 移动 `CameraTracker` 到期望位置
3. 保存

### 5.3 调整 LookAtHead(看向模型面部)

**目标**: 调整相机看向的位置(默认看向玩家面部)

**操作步骤**:
1. 在 Hierarchy 中找到 `Tracer - Head - LookAtHead[Offset]`
2. 移动到模型的脸部中间位置
3. 保存

### 5.4 更改相机外形

**目标**: 用自己喜欢的相机模型替换默认的鱼眼相机模型

#### 5.4.1 Addon 预制体清单

Addon 路径: `Assets/GoatCannery/FishEyeCam/Addon/`

| 预制体 | 可见性 | 适用手 |
|--------|--------|--------|
| `FFELens_Addon_ChangeCameraOBJ(GlobalShow)` | 对其他人相机**可见** | 右手 |
| `FFELens_Addon_ChangeCameraOBJ(Local)` | 对其他人相机**不可见** | 右手 |
| (left 文件夹下) | 同上 | **左手** |

#### 5.4.2 替换步骤

1. 在 Addon 文件夹选择合适的 prefab
2. **重要**: 拖入到**模型**(Avatar)下,成为子物体
3. **不是** FishEyeCam 的子物体
4. 在 `[inCameraOBJ]` 层级下,用准备好的相机模型**替代**原有模型
5. 调整新相机的位置

#### 5.4.3 关键注意事项

> 🔴 **位置要求**: 相机外形安装在镜头的**背面**,否则会被拍到画面中

**调试方法**:
- 选择 `Far Camera` 进行相机预览
- 确认新加入的相机是否被拍摄到画面中

#### 5.4.4 上传时的特殊行为

- 上传时,原本包含的鱼眼镜头对象**将被删除**
- 这是 Avatar 端 Prefab 替换的标准流程

#### 5.4.5 自定义路径的陷阱

> ⚠️ **警告**: 如果不通过标准方法更换相机模型,而是另辟蹊径(如直接用 `WorldFix` 和 `LookAtHead·Drone` 移动相机),**相机位置本身与其他人的位置和方向也不会同步**。在他人看来,相机会永远停留在手边。

**结论**: **必须使用 Addon 方式**,而不是手动移动。

### 5.5 性能影响

| 操作 | 性能影响 |
|------|----------|
| 召出鱼眼相机 | 中等(2 个高分辨率相机) |
| Camera Culling OFF | 高(显示所有物体) |
| 远景 Clipping Far 调高 | 中(渲染距离增加) |
| 其他操作 | 低 |

---

## §6 Q&A 与故障排除(FAQ & Troubleshooting)

### Q1: 照片中会出现网格或高光吗?

**A**: 通常不可见,**仅当以 720P 分辨率拍摄时,它才会显示在照片中**。

**解决方法**:
- 使用 1080P 或更高分辨率拍摄
- 在 Inspector 中调整 Resolution Far/Near 质量

### Q2: 远处的物体和 skybox 不可见怎么办?

**A**: 在 expression 菜单中,增加 `[Setting] - [Clipping Far]` 的值。

| 距离 | 设置 |
|------|------|
| 默认 | 1 km |
| 中等 | 10 km |
| 极远 | 1000 km |

**如果是近处物体不可见**: 参考 Q3 (LOD 问题)

### Q3: 为什么有些物体不可见?

**A**: 由于 Unity 的规格,在启用 LOD 的世界中物体可能会消失。

**原理**:
- LOD 是一种根据摄像机距离切换为简单物体的功能
- FlexFishEyeLens 使用的相机是**超广角**的
- 高度宽度变小,看起来像是低 LOD
- Unity 触发 LOD 切换,远处物体变成更简单的模型甚至消失

**这是已知的设计限制**,无直接解决方法,需在世界搭建时考虑。

### Q4: 一些着色器渲染不正确怎么办?

**A**: FlexFishEyeLens 使用两个摄像头捕捉图像。

**根本原因**:
- 它与从相机屏幕 UV 计算和渲染的着色器不兼容
- 两个摄像头的 UV 拼接会导致基于 UV 的 Shader 计算错误

**Workaround**:
- 增加 `[SceenSize]`
- 仅使用内置摄像头
- 代价: 图像质量会降低

> 📚 **知识库映射**: 这是 **Avatar Shader 域** 的典型问题。屏幕 UV 计算类 Shader(后处理、全屏效果)与多相机架构不兼容。详见 [[entities/avatar/shader/index]]。

### Q&A 分类整理 [INFERENCE]

| 类别 | 问题 | 涉及技术点 |
|------|------|------------|
| **照片质量** | 网格、高光 | 分辨率阈值 |
| **场景范围** | 远景不可见 | 相机 Clipping Far |
| **LOD 兼容** | 部分物体消失 | Unity LOD 系统 + 广角触发 |
| **Shader 兼容** | 部分 Shader 渲染错误 | 2 相机架构 + UV 计算 Shader |

### 故障排除决策树 [INFERENCE]

```
照片有网格?
├── 是 → 分辨率 < 1080P → 提高到 1080P+
└── 否 → 继续

远景 / Skybox 不可见?
├── 是 → 调整 [Clipping Far] → 50% 或 100%
└── 否 → 继续

物体看起来消失 / 简化?
├── 是 → 可能是 LOD 触发 → 世界端问题,无解
└── 否 → 继续

Shader 渲染异常(全屏、UV 类)?
├── 是 → 增加 [SceenSize] + 使用内置摄像头
└── 否 → 咨询作者
```

### 与 BOOTH 官方 Q&A 对照

从 BOOTH 页面 (`https://goat-cannery.booth.pm/items/5512392`) 提取的 Q&A:

> **Q**: Flex FishEye Lens を使用している時に、一部のオブジェクトが写らない時があります。
> **A**: LODが効いているワールドだと、Unityのカメラの仕様上、オブジェクトが消える事があります

> **Q**: 一部のShaderがちゃんと描写されない
> **A**: FlexFishEyeLensはカメラを２つ使用して描写しているために、カメラの画面UVから計算して描写をしているShaderとの相性が悪いです

**对照结论**: BOOTH 原版与 VRCD 汉化版 Q&A **完全一致**,验证了翻译的准确性。

### 调试技巧汇总 [INFERENCE]

| 技巧 | 适用场景 |
|------|----------|
| 使用 `Far Camera` 预览 | 任何调试场景 |
| 调整 Resolution Far/Near(4 倍数) | 性能与画质平衡 |
| 调整 CameraTracker | 机位舒适度 |
| 调整 LookAtHead[Offset] | 看向位置精度 |
| 选择正确 Addon 预制体 | 多人可见性 |
| 关闭 `Camera Culling OFF` 调试 | 仅用于调试,会卡顿 |

---

## §7 版本历史(Version History)

### 7.1 版本演进时间线

| 日期 | 变更 |
|------|------|
| 2024-02-21 | 首次发布于 BOOTH(`booth.pm/ja/items/5512392`) |
| 2025-10-01 | Preset 调整;无人机操控借鉴 Integral;新增 Arm 模式;相机上下角度调整 |
| 2025-10-07 | 新增 1:√2 (Silver Ratio) 长宽比;预览模式黑框;左手相机旋转修复 |
| 2025-12 | VRCD 中文文档汉化发布 |

### 7.2 25/10/07 更新详情

| 变更项 | 描述 |
|--------|------|
| **新增 1:√2 (Silver Ratio)** | 增加白银比例长宽比选项 |
| **预览黑框模式** | 预览时显示黑框,辅助构图 |
| **左手相机旋转修复** | 修正左手用户使用时的相机旋转 bug |

### 7.3 25/10/01 更新详情

| 变更项 | 描述 |
|--------|------|
| **Preset 调整** | 部分预设参数优化 |
| **无人机操作** | 借鉴 `suzuki_ith` 的 `[Integral]` 系统操控方式 |
| **Arm 模式** | 无人机扩展模式 |
| **相机上下角度** | 相机垂直角度可调整 |

### 7.4 作者致敬

> **致谢**: 无人机操作系统的设计借鉴了 `suzuki_ith` 制作的相机无人机操作系统
> - 引用: 【VRChat用】Integral 相机ギミック
> - BOOTH 链接: `https://booth.pm/ja/items/4724145`
> - 获得作者许可

### 7.5 版本注意事项

> ⚠️ **重要**: 升级插件时,拍摄质量设置会被初始化,请在更新后重新设置。

**实践建议**:
- 升级前记录当前的 Resolution Far / Near 值
- 升级后第一时间重新设置
- 测试拍照效果确认无回退

### 7.6 长期维护承诺评估 [INFERENCE]

| 维度 | 评估 | 依据 |
|------|------|------|
| **维护活跃度** | ✅ 高 | 2025 年内 2 次大更新(10/01, 10/07) |
| **作者响应** | ✅ 积极 | BOOTH 上有详细 Q&A,持续发布更新 |
| **社区认可** | ✅ 良好 | VRCD 文档库收录,中文社区有汉化版 |
| **价格稳定** | ¥2,000 | BOOTH 售价未变 |
| **首发时间** | 2024-02-21 | 已运营 2+ 年 |
| **跨平台** | ❌ PC VR Only | Quest 不支持 |

---

## §8 设计分析(Design Analysis)[INFERENCE]

> ⚠️ **本章为作者观点/分析**,非原作者陈述,仅供参考。

### 8.1 生态关系图

```
Integral (无人机操控原型) ← Flex FishEye Lens 借鉴
Modular Avatar (Avatar 框架) → Flex FishEye Lens 依赖
```

| 项目 | 价值 |
|------|------|
| **Flex FishEye Lens** | 鱼眼相机插件(本次研究) |
| **Integral 相机ギミック** | 无人机操控原型系统(被引用) |
| **Modular Avatar** | 安装机制(前置依赖) |

### 8.2 多模态操控分离设计

```
本体 (FishEyeCam)        ← 玩家绑定
├── 预设模式            ← 快速应用
├── 手动模式            ← 细粒度调参
├── 无人机模式 (Drone)  ← 机位独立 ← 双手分离
└── 行为模式 (Behavior) ← 行为控制
```

### 8.3 与 Unity Camera 的对比

| 维度 | Unity Camera | Flex Fisheye Lens |
|------|--------------|-------------------|
| 渲染目标 | 单一相机 → RenderTexture | 2 个高分辨率相机叠加 |
| 畸变控制 | 需自定义 Shader/后处理 | 菜单直接调节 |
| 景深 | 内置 Bokeh 等高级效果 | 简单景深,不支持前景模糊 |
| 网络开销 | N/A | Local Only,零网络 |
| 平台限制 | 跨平台 | PC VR Only |
| 性能开销 | 单相机开销 | 2x 相机 + 畸变计算 |

### 8.4 [World Fix] 的多入口设计

[World Fix] 在 **预设、手动、行为** 三个菜单中均出现 — 这是常见的设计模式:高频操作多个入口,降低导航成本。

**对照**:
- P → [World Fix]: 与预设配合,快速固定机位
- M → [World Fix]: 与手动调参配合,固定后细调
- Behavior → [World Fix]: 行为级控制,与其他行为(LookAtHead 等)联动

---

## §9 知识库关系(KB Relations)

### 9.1 与其他知识库文档的关系

| 关联文档 | 关联类型 | 说明 |
|----------|----------|------|
| [[concepts/hybrid/udon-world-plugins]] | **兄弟索引** | 同为插件索引,但属于 World 端 Udon 工具 |
| [[entities/avatar/modular-avatar]] | **依赖** | MA 是安装前置,本插件是 MA 生态子应用 |
| [[entities/avatar/shader/index]] | **注意事项** | 2 相机架构与 UV Shader 兼容问题 |
| [[concepts/hybrid/audio-link]] | **场景联动** | 摄影场景可与音频可视化结合 |
| [[concepts/hybrid/index]] | **所属域** | Hybrid 域(Avatar 侧) |
| [[concepts/hybrid/osc-protocol]] | **参考** | OSC 协议可扩展摄影工具能力 |
| （来源：Open Source Projects，本地整理） | **对照** | 本插件闭源,与开源参考工程区分 |

### 9.2 推荐工具评估标准

如果一个第三方 Avatar 端插件希望被收录到本文档,需满足:

| 维度 | 要求 |
|------|------|
| **维护状态** | 最近 6 个月内至少有 1 次 commit/update |
| **社区反馈** | 10+ stars 或 100+ downloads |
| **文档完整** | README + 至少 3 个示例 Prefab |
| **分发渠道** | 有可访问的购买/下载链接(BOOTH/GitHub) |
| **设计质量** | 遵循 VRChat 最佳实践(无反射黑魔法、无 GC 滥用) |
| **跨平台** | PC/Quest 兼容(优先);不兼容需明确标注 |
| **可验证** | 知识库已建立相关分析文档 |

---

## §10 维护记录(Maintenance Log)

| 日期 | 变更 | 操作者 |
|------|------|--------|
| 2026-07-04 | 创建本文档,收录 Flex Fisheye Lens 作为首个 Avatar 端摄影工具 | Knowledge Curator Agent |

---

## 附录:关键警告清单(必读)

> 🔴 **本节是使用前的必读警告,所有 Flex Fisheye Lens 用户都应知晓**

1. **VR-only** — 桌面模式和 Quest 均不可用
2. **MA 必需** — 必须先安装 Modular Avatar
3. **原相机需跟随玩家** — 飞行模式会导致效果失效
4. **2 相机架构** — 性能开销 + UV Shader 不兼容
5. **4 倍数约束** — Resolution Far/Near 必须是 4 的倍数
6. **升级副作用** — 质量设置会被重置
7. **Addon 替换** — 必须使用 Addon 预制体,不能手动移动
8. **Local Only** — 其他人看不到你手上的镜头(零网络开销)
9. **价格** — ¥2,000 (截至 2025-12)
10. **作者要求** — Twitter 等社交媒体发布需加 `#FlexFishEye` 标签

---

## 外部资源链接

| 资源 | 链接 |
|------|------|
| **BOOTH 商品页(日文)** | `https://booth.pm/ja/items/5512392` |
| **BOOTH 商品页(英文)** | `https://booth.pm/en/items/5512392` |
| **BOOTH 商品页(原始)** | `https://goat-cannery.booth.pm/items/5512392` |
| **VRCD 汉化文档** | `https://docs.vrcd.org.cn/books/flex-fisheye-lens` |
| **VRCD 概览页** | `https://docs.vrcd.org.cn/books/vrc-uyG/page/flex-fisheye-lens` |
| **官方使用手册(Google Docs)** | `https://docs.google.com/document/d/14vIpZHvVXfoYedPgE2tbDNJkNOJvSwp_cLzaTu-ARag/edit?tab=t.0` |
| **作者 Twitter** | 需搜索 `#FlexFishEye` 标签 |
| **作者站点** | Goat-Cannery BOOTH 主页 |
| **示例 World** | `wrld_44b14599-7011-48d3-b40f-41c0dc80252e` |
| **试用 Avatar** | `avtr_04cd67b2-3a7d-4a20-b322-17dd437221b8` |
| **引用 Integral 插件** | `https://booth.pm/ja/items/4724145` |

---

**文档结束。如有更新,请在维护记录中追加。**

---

## 相关页面

[[udon-world-plugins.md]] · [[index.md]] · [[audio-link.md]] · [[../../entities/avatar/modular-avatar.md]] · [[../../entities/avatar/shader/index.md]] · [[osc-protocol.md]]
