---
title: VRCCameraSettings
category: world
knowledge_level: applied
status: active
source: 本地知识库整理
source_type: community
version: 1.1
last_review: 2026-07-04
confidence: Medium
tags:
  - world
  - udonsharp
  - reference
  - integral
  - camera-plugin
  - third-party-integration
aliases:
  - VRCCameraSettings
  - vrc-camera-settings.md
related:
  - udon/vrc-graphics/index.md
  - vrc-quality-settings.md
  - performance-guide.md
  - ../avatar/integral-camera-params.md
type: entity
created: 2026-07-04
sources: 本地知识库整理
updated: 2026-07-04
---
# VRCCameraSettings


---

## Overview

`VRCCameraSettings` 暴露用户屏幕相机、手持相机以及 Unity 全局质量设置的**只读信息**和**有限控制**能力。

**重要限制**:
- 不能直接访问原始 `Camera` 组件（出于安全考虑）
- 用户自己放入 World 的 `Camera` 组件可正常访问
- 替换了 Unity 的 `Camera.current`（因为 `Camera.current` 仅在渲染事件中填充）

**使用场景**:
- 在 `OnWillRenderObject` 中判断当前是哪个相机正在渲染
- 获取 VR 双眼位置/旋转
- 检测手持相机是否启用（Active）
- 响应相机设置变化

---

## 静态实例

```csharp
VRCCameraSettings.ScreenCamera   // 屏幕主相机（始终存在）
VRCCameraSettings.PhotoCamera    // 手持拍照相机（Spout 推流启用时也 Active）
```

| 实例 | 说明 |
|------|------|
| `ScreenCamera` | 始终 Active，代表用户的主视点相机 |
| `PhotoCamera` | 手持拍照相机的预览相机，**按快门时**属性会与拍照/推流相机同步 |

**注意**: `PhotoCamera` 始终指向**预览图像**的相机，不是拍照本身的相机或推流相机的渲染视图。

---

## 实例属性

这些实例会暴露 `UnityEngine.Camera` 的部分属性（参考 Unity 文档获取详细信息）。

### Active

```csharp
bool VRCCameraSettings.ScreenCamera.Active;  // 始终为 true
bool VRCCameraSettings.PhotoCamera.Active;   // 手持相机启用时为 true
```

| 值 | 含义 |
|---|---|
| `ScreenCamera.Active` | 始终 `true` |
| `PhotoCamera.Active` | 手持相机启用时 `true`；Spout 推流启用时也 `true` |

**典型用法**: 检测手持相机是否启用。
```csharp
if (VRCCameraSettings.PhotoCamera.Active) { /* 手持相机已开 */ }
```

### StereoEnabled

```csharp
bool VRCCameraSettings.ScreenCamera.StereoEnabled;  // VR 用户为 true
```

**典型用法**: 检测用户是否在 VR 中（**比 `XR.enabled` 更可靠**）。

### CameraMode

```csharp
CameraMode VRCCameraSettings.ScreenCamera.CameraMode;
CameraMode VRCCameraSettings.PhotoCamera.CameraMode;
```

参见 [Camera Mode](#camera-mode) 章节。

### 屏幕 / 拍照相机 Pos/Rot

知识库 `world/udon/vrc-graphics/` 的 `_VRChatScreenCameraPos` / `_VRChatPhotoCameraPos` 等全局着色器变量有同样信息。VRCCameraSettings 提供 C# 端访问。

---

## Camera Mode

`CameraMode` 属性在 `ScreenCamera` 和 `PhotoCamera` 上可用。类型为 `VRCCameraMode` 枚举：

| 枚举值 | 适用相机 | 含义 |
|--------|---------|------|
| `Screen` | ScreenCamera | 正常渲染（VR 双眼、Desktop 单眼） |
| `FocusView` | ScreenCamera | 聚焦视图模式 |
| `PhotoOrVideo` | PhotoCamera | 照片/视频构图模式 |
| `Print` | PhotoCamera | 截图/快照模式 |
| `DroneHandheld` | PhotoCamera | Drone 手持模式 |
| `DroneFPV` | PhotoCamera | Drone 第一人称视角模式 |
| `Unknown` | PhotoCamera | 未知/未初始化 |

> **来源**: 官方 [`VRCCameraSettings#camera-mode`](https://creators.vrchat.com/worlds/udon/vrc-graphics/vrc-camera-settings/#camera-mode)。原知识库误将 `_VRChatCameraMode` shader global 的数值含义当作 C# 枚举，已修正。

**典型用法**: 区分手持相机的 VR/Desktop 渲染。

---

## 可写属性（官方 read-write）

以下属性在 `ScreenCamera` / `PhotoCamera` / 全局可用，可在运行时修改：

| 属性 | 说明 |
|------|------|
| `NearClipPlane` / `FarClipPlane` | 近/远裁剪面（clamp 范围 0.001–0.05 等，官方有边界） |
| `AllowHDR` | 是否允许 HDR 渲染 |
| `DepthTextureMode` | **PhotoCamera 恒为 `Depth` 且不可关闭** |
| `UseOcclusionCulling` | 遮挡剔除开关 |
| `AllowMSAA` | 多重采样抗锯齿开关 |
| `CullingMask` | 相机渲染的层掩码 |
| `ClearFlags` / `BackgroundColor` | 清除标志 / 背景色 |
| `LayerCullDistances` | 各层剔除距离 |

> ⚠️ **ClientSim**: ClientSim 中 `PhotoCamera` 为 **null**（官方提示），访问前需判空。

> 来源: 官方 [VRCCameraSettings](https://creators.vrchat.com/worlds/udon/vrc-graphics/vrc-camera-settings/)。

---

## 静态函数

### GetEyePosition / GetEyeRotation

对 **VR 用户**最有用：

```csharp
Vector3 VRCCameraSettings.GetEyePosition(Camera.StereoscopicEye eye);
Quaternion VRCCameraSettings.GetEyeRotation(Camera.StereoscopicEye eye);
```

| 参数 | 值 | 说明 |
|------|---|------|
| `eye` | `Camera.StereoscopicEye.Left` | 左眼 |
| `eye` | `Camera.StereoscopicEye.Right` | 右眼 |

**注意**:
- 对**非 VR 用户**，调用是合法的，但**值等同于** `VRCCameraSettings.ScreenCamera` 的 Pos/Rot
- 推荐用 `StereoEnabled` 检测 VR 用户，**不要假设** `GetEyePosition` 双返回值

### GetCurrentCamera

```csharp
void VRCCameraSettings.GetCurrentCamera(
    out VRCCameraSettings internalComponent,
    out Camera externalComponent
);
```

`Camera.current` 的**安全替代**，因为 `Camera.current` 仅在渲染事件中填充。

| 场景 | `internalComponent` | `externalComponent` |
|------|---------------------|---------------------|
| 已知内部相机渲染中 | `ScreenCamera` / `PhotoCamera` | `null` |
| 用户自定义相机渲染中 | `null` | `UnityEngine.Camera` |
| `Camera.current` 为 null | `null` | `null` |
| Udon 无权访问的相机（如 Avatar 上的相机）| `null` | `null` |

**⚠️ 关键警告**:
- 即使没有 Avatar 相机，**这个函数可能**在渲染事件中**返回双 null**
- 原因：VRChat 内部渲染步骤（如内置菜单）
- **必须**处理 `internalComponent == null && externalComponent == null` 的情况

**典型用法**:
```csharp
public override void OnWillRenderObject()
{
    VRCCameraSettings.GetCurrentCamera(out var internalCam, out var externalCam);
    if (internalCam != null) {
        // 屏幕相机或手持相机渲染中
    } else if (externalCam != null) {
        // 用户自定义相机渲染中
    } else {
        // 不可访问的相机，或内置渲染步骤
        return;
    }
}
```

---

## 事件

### OnVRCCameraSettingsChanged

```csharp
public override void OnVRCCameraSettingsChanged(VRCCameraSettings camera)
{
    // camera 是变化的相机实例
    if (camera != VRCCameraSettings.ScreenCamera) return;
    // 处理分辨率等变化
}
```

**触发场景**:
- 分辨率变化
- 少数其他属性变化

**生命周期提示**:
- `Start()` 中**不会**自动调用
- 需要在 `Start()` 手动调用一次以初始化
- 配合 `OnVRCCameraSettingsChanged` 实现响应式更新

---

## 完整示例：CameraInfoDisplay

```csharp
using TMPro;
using UdonSharp;
using UnityEngine;
using VRC.SDK3.Rendering;

public class CameraInfoDisplay : UdonSharpBehaviour
{
    [SerializeField] private TextMeshProUGUI info;

    void Start()
    {
        // 手动初始化一次（事件不会自动触发 Start）
        OnVRCCameraSettingsChanged(VRCCameraSettings.ScreenCamera);

        Debug.Log($"Started CameraInfoDisplay at resolution of " +
                  $"{VRCCameraSettings.ScreenCamera.PixelWidth}x" +
                  $"{VRCCameraSettings.ScreenCamera.PixelHeight}");
        Debug.Log($"The handheld photo camera is " +
                  $"{(VRCCameraSettings.PhotoCamera.Active ? "enabled" : "disabled")}");
    }

    public override void OnVRCCameraSettingsChanged(VRCCameraSettings camera)
    {
        // 忽略手持相机
        if (camera != VRCCameraSettings.ScreenCamera) return;

        info.text = $"{camera.PixelWidth}x{camera.PixelHeight} " +
                    $"fov={camera.FieldOfView} frame={Time.frameCount}°";
    }
}
```

**注意**:
- `using VRC.SDK3.Rendering;` 必须
- 完整字段列表（`PixelWidth` / `PixelHeight` / `FieldOfView` 等）参考 Unity `Camera` 文档

---

## 与 `_VRChatCameraMode` 全局变量对应

`VRCCameraSettings` 的 C# 端 `CameraMode`（`VRCCameraMode` 枚举）与 Shader 全局变量 `_VRChatCameraMode`（整数）描述**同一概念**，但值编码**不同**：C# 侧为枚举名（`Screen`/`FocusView`/`PhotoOrVideo`…），shader 侧为整数。写 shader 时只能用 `_VRChatCameraMode` 的整数约定；写 U# 时用枚举名。

| 用途 | C# API | Shader 全局 |
|------|--------|------------|
| 屏幕相机模式 | `VRCCameraSettings.ScreenCamera.CameraMode` | `_VRChatCameraMode` |
| 手持相机模式 | `VRCCameraSettings.PhotoCamera.CameraMode` | `_VRChatCameraMode` |
| 屏幕相机位置 | `VRCCameraSettings.ScreenCamera.transform.position` | `_VRChatScreenCameraPos` |
| 拍照相机位置 | `VRCCameraSettings.PhotoCamera.transform.position` | `_VRChatPhotoCameraPos` |

---

## 性能与陷阱

| 陷阱 | 说明 |
|------|------|
| `OnVRCCameraSettingsChanged` 不会自动触发 | `Start()` 中需手动调用一次 |
| `GetCurrentCamera` 可能返回双 null | 即便在渲染事件中也要处理 |
| `PhotoCamera` 不等于拍照相机 | 始终指向**预览**相机 |
| VR 双眼位置仅在 `StereoEnabled == true` 时有意义 | 非 VR 模式下与 `ScreenCamera` 相同 |

---

## 相关文档

- [[entities/world/udon/vrc-graphics/index]] - VRCShader / VRCGraphics API + Shader Globals
- [[entities/world/vrc-quality-settings]] - VRCQualitySettings（同一上级页面）
- [[entities/world/performance-guide]] - 渲染性能优化

---

## 📌 第三方相机插件与 VRChat 原相机的集成

> 📅 2026-07-04 新增 - 来自 Integral 笔记整理
> 来源：[Integral（SuzuFactory Booth）](https://suzufactory.booth.pm/items/4724145) §4 基础曝光模式

### 集成模式

所有第三方 Avatar 相机插件（**Integral / VRCLens / VirtualLens2**）都遵循同一模式：

```
1. 用户在 Avatar 端启用第三方相机插件
2. 插件必须检测 VRChat 原相机（VRCCameraSettings.PhotoCamera）已启用
3. 插件通过 RenderTexture / Shader 替换原相机画面
4. 拍摄触发仍由原相机快门完成
```

**为什么必须开原相机**：

- VRChat 的拍照/推流功能只能通过原相机触发
- 第三方插件只能"替换画面"，无法"触发拍照"
- 关闭原相机 = 无法拍摄/推流

### VRChat 2025.3.3 新增 OSC 相机端点

VRChat 2025.3.3 版本（[Release Notes](https://docs.vrchat.com/docs/vrchat-202533)）新增了完整的 OSC 相机控制端点：

**模式控制**：

| OSC 端点 | 类型 | 取值范围 |
|---------|------|---------|
| `/usercamera/Mode` | Get/Set | 0=Off, 1=Photo, 2=Stream, 3=Emoji, 4=Multilayer, 5=Print, 6=Drone |
| `/usercamera/Pose` | Get/Set | 相机位置 + 旋转 |
| `/usercamera/Close` | Action | 关闭相机 |
| `/usercamera/Capture` | Action | 立即拍照 |
| `/usercamera/CaptureDelayed` | Action | 延时拍照 |

**滑块控制**（float）：

| 端点 | 默认 | 最小 | 最大 |
|------|------|------|------|
| `/usercamera/Zoom` | 45 | 20 | 150 |
| `/usercamera/Exposure` | 0 | -10 | 4 |
| `/usercamera/FocalDistance` | 1.5 | 0 | 10 |
| `/usercamera/Aperture` | 15 | 1.4 | 32 |
| `/usercamera/FlySpeed` | 3 | 0.1 | 15 |
| `/usercamera/TurnSpeed` | 1 | 0.1 | 5 |
| `/usercamera/SmoothingStrength` | 5 | 0.1 | 10 |

**意义**：
- 第三方插件可通过 OSC 监听/控制原相机参数
- 与官方系统联动成为可能
- 解释了"为什么第三方插件必须开原相机"——因为 OSC 端点也是基于原相机的

### 相关文档

- [Integral（SuzuFactory Booth）](https://suzufactory.booth.pm/items/4724145) §4 基础曝光模式
- （来源：UdonVoiceUtils 分析，本地整理）
- [[concepts/hybrid/osc-protocol]]

---

## 📌 相机 Anchor 模式（与第三方相机插件的对应关系）

> 📅 2026-07-04 新增 - 来自 Integral 笔记整理
> 来源：[Integral（SuzuFactory Booth）](https://suzufactory.booth.pm/items/4724145) §5 相机移动系统

### VRChat 原相机的 Anchor 功能

VRChat Wiki 官方文档（[Camera](https://wiki.vrchat.com/wiki/Camera)）定义了相机的 Anchor 行为：

| Anchor | 行为 |
|--------|------|
| **Default** | Attached to the user（默认：附着于用户） |
| **Local** | Follows user movement but not head movement（跟随移动但不跟头部） |
| **World** | **Attached to world, independent of user movement**（附着于世界，独立于用户移动） |

### 与第三方相机插件 Fix Mode 的对应

| VRChat 原相机 Anchor | Integral Fix Mode | VRCLens | VirtualLens2 |
|---------------------|------------------|---------|--------------|
| Default | Avatar 角色固定 | Avatar-fixed | （类似） |
| World | World 世界固定 | World-fixed | （类似） |
| Local | （无对应） | （无对应） | （无对应） |

**关键发现**：
- 第三方相机插件的 **World/Avatar 固定模式本质上是 VRChat 原相机 Anchor 功能的复用**
- 这是**业界标准模式**（VRCLens/VirtualLens2/Integral 都采用同样模式）
- Integral 的"Fix While Open"（曝光时锁定位置+朝向+AF）是**独创功能**

### Anchor 行为详解

| 场景 | Default | Local | World |
|------|---------|-------|-------|
| 用户移动 | 相机跟随 | 相机跟随 | 相机不动 |
| 用户转身 | 相机跟随 | 相机跟随 | 相机不动 |
| 用户头部转动 | 相机跟随 | 相机不动 | 相机不动 |
| 适用 | 一般情况 | 跟随身体但不跟头 | 固定机位 |

### 相关文档

- [Integral（SuzuFactory Booth）](https://suzufactory.booth.pm/items/4724145) §5 相机移动系统 + §5.2 Fix Mode
- [[entities/avatar/integral-camera-params]] §6.2 Auto Level 自动水平

---

## Avatar 相机限制（2026-07 新增）⭐

> 🔴 **来源**: VRChat 官方 Canny feedback 平台
> 🔴 **状态**: tracked by VRChat Staff (`_tau_`)

### 已知问题清单

| 痛点 | VRChat 官方确认 | 影响 | 跟踪状态 |
|------|----------------|------|----------|
| **Avatar Camera 无 DepthTextureMode** | ✅ 官方承认 | DoF 效果异常、shader 行为异常 | tracked |
| **DoF 噪声无法关闭** | ✅ 官方承认 | 视频录制和后期降噪困难 | tracked (32 票) |
| **Avatar 相机使用 stale depth texture** | ✅ 官方承认 | 深度相关效果失效 | tracked |

### 官方原话引用

> "VRCCameraSettings - DepthTextureMode doesn't affect avatar cameras. This produces artifacts and broken shader behavior with **third party camera systems like VRCLens, etc**, in worlds that utilize DepthTextureMode instead of a depth light."
> — VRChat Canny, marked as tracked by `_tau_` (VRChat Staff)

### 对创作者的影响

1. **Avatar 摄影**: 标准相机在复杂 Avatar（多部件、配饰、PhysBone）下景深判断不可靠
2. **第三方工具存在合理性**: VRCLens、VirtualLens2 等工具填补了这一空白
3. **Shader 开发**: 在 Avatar 中使用 DepthTexture 相关 shader 时需考虑 stale depth 风险

### 推荐应对方案

- 创作复杂 Avatar 时: 考虑使用第三方摄影工具（VirtualLens2 / VRCLens）
- World 端 Shader: 避免假设 Avatar 相机具有 DepthTextureMode 能力
- 等待 VRChat 官方修复（tracked 但未解决）

### 引用

- [EXTERNAL] VRChat Canny - DepthTextureMode doesn't affect avatar cameras: <https://feedback.vrchat.com/feature-requests/p/vrccamerasettings-depthtexturemode-doesnt-affect-avatar-cameras>
- [EXTERNAL] VRChat Canny - Toggle the noise from the camera: <https://feedback.vrchat.com/feature-requests/p/toggle-the-noise-from-the-camera>

> ⚠️ 截至 2026-07-04，这些问题状态为 tracked by Staff 但**未解决**。建议在创作者工作流中作为长期约束对待。
> 完整分析见 [[entities/avatar/camera-tools]] §5

---

## 第三方工具对光照错误的修复（2026-07 新增）

### VRCLens 修复世界光照错误

> 来源: VRCLens 中文文档 + Hirabiki 官方

**已知问题**: VRChat 部分世界（特别是未充分烘焙光照的 World）中存在以下错误：
- 蓝屏 viewfinder 问题
- 弱定向光下对象外观异常
- 曝光判断不准确

**VRCLens 应对方案**:
- 通过 EVILS Tonemapper（Filament PBR）实现 HDR + White Balance
- 在 Avatar 相机上覆盖 PostProcess 设置
- 提供了比原相机更准确的光照表现

**对比 VirtualLens2**:
- VirtualLens2 提到"受弱定向光影响较大的对象"作为已知问题，建议暂时禁用深度纹理生成
- 文档未提供"修复光照"的明确能力
- **结论**: 在光照修复能力上，VRCLens > VirtualLens2

### 引用

- [L2] VRCLens 中文文档: <https://docs.vrcd.org.cn/books/vrclens>
- 详细对比见 [[entities/avatar/camera-tools]] §3

---

## 4K 模式与分辨率匹配（2026-07 新增）

> 来源: VirtualLens2 官方文档

### VirtualLens2 4K 启动条件

1. **VirtualLens2 启用 4K 捕获**（在"算法设置"中）
2. **VRChat 内置相机分辨率** 设置为 **4K (2160p) = 3840x2160**

### 分辨率不匹配时的行为

| VRChat 设置 | VirtualLens2 行为 | 质量影响 |
|-------------|------------------|----------|
| 3840x2160 | 直接使用 | ✅ 最佳 |
| 1920x1080（默认）| 启用 4K 时略降（用于预览）| ⚠️ 略降 |
| 其他分辨率 | 强制缩放至 1920x1080 | 🔴 质量下降 |

### Stream Camera 兼容

- Stream Camera 模式：相机分辨率 = 桌面窗口大小
- **建议**: 使用 VRChat 启动选项设置窗口为 3840x2160（全屏）

### VRChat 启动选项参考

```
vrchat://launch?resolution=3840x2160
```

详细文档: [VRChat Launch Options](https://docs.vrchat.com/docs/launch-options#resolution-and-screen-choice)

### 引用

- [L2] VirtualLens2 4K 章节: <https://docs.vrcd.org.cn/books/virtual-lens2/page/4k>
- [L1] VRChat Launch Options: <https://docs.vrchat.com/docs/launch-options#resolution-and-screen-choice>
