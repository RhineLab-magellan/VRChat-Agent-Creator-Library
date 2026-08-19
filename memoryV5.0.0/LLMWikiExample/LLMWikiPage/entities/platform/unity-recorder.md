---
title: Unity Recorder — 截图与录屏工具
category: platform
subcategory: tool
knowledge_level: applied
status: active
source: 本地知识库整理
source_type: official
version: 1.0
last_review: 2026-07-04
confidence: High
tags:
  - platform
  - unity
  - tool
  - screenshot
  - recording
  - recorder
aliases:
  - Unity Recorder
  - 截图工具
  - 录屏工具
  - com.unity.recorder
related:
  - unity-admin-warning.md
  - ../avatar/avatar-modding-guide.md
source_origin: 从 07-16790.md §Q "Unity里面截完图然后直接放在菜单该怎么做" 提取 (P1)
type: entity
created: 2026-07-04
sources: 本地知识库整理
updated: 2026-07-04
---

# Unity Recorder — 截图与录屏工具

> Unity 官方的 Editor-only 截图/录屏工具,用于在 Unity 编辑器中捕获 Game View 内容。
>
> **官方文档**: https://docs.unity3d.com/Packages/com.unity.recorder@5.1/manual/
> **源 QA**: 07-16790.md §"Unity 里面截完图然后直接放在菜单该怎么做"

---

## 0. 速查表

| 问题 | 解答 |
|------|------|
| Recorder 在哪里? | `Window` > `General` > `Recorder` > `Recorder Window` |
| 怎么安装? | `Window` > `Package Manager` > `Unity Registry` > 搜索 "Recorder" > `Install` |
| 截图存在哪? | 默认: `<项目根目录>/Screenshots/` |
| 能用于 Build 吗? | ❌ **不能**,仅 Editor 工具 |
| VRChat SDK 兼容版本? | 需用 **Recorder 4.x**(SDK 3.5.0+ 绑定 Unity 2022.3.22f1 LTS) |

---

## 1. 核心要点

### 1.1 关键限制(必读)

> 🔴 **Editor-only**
>
> The Recorder package is an Editor-only tool. It does **not** work in standalone Unity Players or builds.
>
> — Unity Official Documentation

**含义**:
- 只能在 Unity 编辑器中使用
- 不能在 Build 出的游戏中使用
- VRChat 是 Build 出的游戏,所以 **VRChat 客户端内无法使用 Recorder**
- 仅在 Editor 中(开发时)用于截图

### 1.2 VRChat Creator 实际使用场景

虽然 Recorder 不能在 VRChat 中运行,但在以下场景下极有用:

- **Avatar 截图**: 在 Editor 中截取 Avatar 在场景中的样子,作为宣传图
- **World 截图**: 截取 World 的特定角度画面作为菜单图、社交媒体分享图
- **调试可视化**: 配合 UI、动画在特定帧的截图
- **文档撰写**: 写教程、README 时的截图

---

## 2. 安装

### 2.1 路径

```
Window → Package Manager → 左侧下拉选 "Unity Registry" → 搜索 "Recorder" → Install
```

### 2.2 版本要求(重要 ⚠️)

| Recorder 版本 | Unity 版本要求 | VRChat SDK 兼容性 |
|---------------|----------------|-------------------|
| **5.x** (当前) | 2023.1+ | ❌ **不兼容**(VRChat SDK 3.5.0+ 绑定 2022.3.22f1) |
| **4.x** | 2022.3 LTS+ | ✅ **推荐** (与 VRChat SDK 3.5.0+ 兼容) |
| 3.x | 2021.3+ | ⚠️ 旧版本,功能受限 |

> **VRChat Creator 必读**:
> 当前 VRChat SDK 3.5.0+ 绑定 Unity 2022.3.22f1 LTS。
> 如果用 Unity Hub 切到此版本,Package Manager 中 Recorder 应固定为 **4.x**(可能需要 "Show preview packages" 或通过 Git URL 安装)。

---

## 3. 使用流程

### 3.1 基本截图

1. **打开 Recorder 窗口**
   ```
   Window → General → Recorder → Recorder Window
   ```

2. **添加 Recorder**
   - 点击 `+ Add Recorder`
   - 选择 `Image Sequence` (逐帧图片) 或 `Movie` (视频)

3. **配置参数**
   - **Source**: `Game View` (默认)
   - **Output Resolution**: 1080p / 4K 等预设
   - **Output Path**: 默认 `<项目>/Screenshots/`

4. **开始录制**
   - 点击 `START RECORDING`
   - Recorder 会自动进入 Play Mode
   - 再次点击 `STOP RECORDING` 结束

### 3.2 快捷键 (F10)

- 某些自定义配置下 `F10` 可触发截图(社区习惯,但官方文档未明确列出)
- 推荐使用窗口内的 `START RECORDING` 按钮

---

## 4. 录制类型

| 类型 | 用途 | 输出 |
|------|------|------|
| **Movie** | 录视频 | MP4 (H.264) / WebM (VP8) / QuickTime (ProRes) |
| **Image Sequence** | 截逐帧图(推荐用于 Avatar 截图) | JPEG / PNG / EXR |
| **Animation Clip** | 录制 Animator 动画 | .anim |
| **Audio** | 单独录音频 | WAV |

> **Avatar Creator 推荐**: 截 Avatar 宣传图时,用 **Image Sequence** + **PNG** 格式,画质无损。

---

## 5. 高级设置

### 5.1 Game View vs Targeted Camera

- **Game View** (默认): 截取 Game 窗口内容
- **Targeted Camera**: 截取指定 Camera 的画面 (Game View 不用时也可截)

### 5.2 Include UI

- 勾选后,UI 元素会包含在截图中
- Avatar 截图时通常不勾选(避免 UI 干扰)

### 5.3 输出分辨率

| 用途 | 推荐分辨率 |
|------|-----------|
| Twitter / 微博 分享 | 1920×1080 (FHD) |
| 高清宣传图 | 2560×1440 (2K) |
| BOOTH 商品图 | 3000×3000 (正方形) |
| 4K 输出 | 3840×2160 |

> ⚠️ 录完后 Game View 分辨率不会自动恢复(需手动调整)

---

## 6. 替代方案 (Build 中)

> 由于 Recorder 仅限 Editor,如需在 VRChat 中截图:

| 方案 | 说明 |
|------|------|
| **Steam 截图键** | VR 头显中,SteamVR 截图键(默认 F12) |
| **Oculus 截图** | Quest 头显:同时按 `音量-` + `电源键` |
| **Windows 截图工具** | `Win + Shift + S` (Snip & Sketch) |
| **NVIDIA ShadowPlay** | `Alt + F10` (如安装了 GeForce Experience) |
| **OBS Studio** | 实时录屏,功能最全 |

---

## 7. 已知问题

| 问题 | 解决方案 |
|------|---------|
| START RECORDING 按钮灰着 | 需先添加 Recorder 类型 |
| 截图模糊 | 检查 Output Resolution,默认用 Game View 分辨率 |
| 截图没声音 | Movie 模式需勾选 "Capture Audio" |
| 5.x 版本不兼容 Unity 2022 LTS | 用 4.x 版本(通过 VPM/Git URL) |

---

## 8. 相关文档

- [[entities/platform/unity-admin-warning]] — Unity 管理员权限警告
- [[entities/avatar/avatar-modding-guide]] — Avatar 改模指南(截图应用场景)
- [[entities/world/items]] — World 资产(World 截图)

---

## 9. 引用

- **官方文档**: https://docs.unity3d.com/Packages/com.unity.recorder@5.1/manual/
- **官方文档 (4.x)**: https://docs.unity3d.com/Packages/com.unity.recorder@4.0/manual/index.html
- **源 QA**: https://docs.vrcd.org.cn/books/16790
- **Unity Learn 教程**: https://learn.unity.com/tutorial/working-with-unity-recorder

---

## 相关页面

[[unity-admin-warning.md]] · [[../avatar/avatar-modding-guide.md]]
