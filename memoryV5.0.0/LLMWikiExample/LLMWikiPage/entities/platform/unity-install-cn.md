---
title: Unity Hub 与编辑器安装 — 国内网络环境
category: platform
knowledge_level: applied
status: active
source: https://docs.vrcd.org.cn/books/86vrchat + Unity 官方 + VRCQuestTools
source_type: community
version: 1.1
last_review: 2026-07-04
changelog: "2026-07-04 新增 §Quest 支持的 Unity 模块安装"
confidence: High
tags:
  - platform
  - unity
  - unity-hub
  - china
  - quest
  - android
aliases:
  - Unity 国内安装
  - Unity Hub 国内安装
  - Unity Install CN
  - 国内网络环境安装 Unity
related:
  - ../world/development/vm-setup.md
  - ../../FACT.md
  - android-development.md
type: entity
created: 2026-07-04
sources: https://docs.vrcd.org.cn/books/86vrchat + Unity 官方 + VRCQuestTools
updated: 2026-07-04
---

# Unity Hub 与编辑器安装 — 国内网络环境

> Domain: Platform | 适用: 国内 VRChat 创作者
> 来源: Unity 官方 + 86vrchat 中文社区

---

## 1. 三个硬性版本约束(VRChat)

**[FACT-OFFICIAL]** VRChat SDK 强制使用以下 Unity 版本:

- **Unity 2022.3.22f1**(LTS)
- **不要 C1 中国版编辑器**(后缀 `c1` 不可用)
- **Build ID**: 887be4894c44

(来源: creators.vrchat.com + FACT.md §渲染管线)

> **为什么不能用 C1?**
> **[COMMUNITY-86vrchat]** C1 中国版编辑器在 VRChat SDK 编译/上传时存在兼容性问题
> (来源: https://docs.vrcd.org.cn/books/86vrchat)

---

## 2. Unity Hub 国际版 vs 中国版

**[FACT]** Unity Hub 分为两个版本:

| 维度 | 国际版 | 中国版 |
|------|--------|--------|
| 客户端下载 | public-cdn.cloud.unity3d.com | unity.cn |
| 标题标识 | 无 C 数字 | 含 C 数字(如 C7) |
| 账户互通 | ✅ 跨版本登录 | ✅ |
| **国内下载** | ⚠️ 较困难 | ✅ 较顺畅 |

**[COMMUNITY-86vrchat]** 推荐**尽可能使用国际版客户端**
(来源: https://docs.vrcd.org.cn/books/86vrchat)

### 2.1 第三方下载源(国内)

> **已采纳**(用户决策 2026-07-04):
>
> - ✅ **NoUnityCN**(https://www.nounitycn.top/unityhub)— Unity 国际版下载站

> **未采纳**:
>
> - ❌ 冻鳗仓库(分发链接)— 第三方分发源,合规性存疑

### 2.2 Hub 安装路径约束

**[COMMUNITY-86vrchat]** **Hub 路径不应包含中文**

- 否则可能"低概率出现各类玄学问题"
- 来源: https://docs.vrcd.org.cn/books/86vrchat
- 状态: **[INFERENCE]** 社区经验,无 Unity 官方文档支持

---

## 3. 编辑器安装

### 3.1 在线安装(推荐)

**[COMMUNITY-86vrchat]** 通过 UnityHub 一键安装:

- 链接: `unityhub://2022.3.22f1/887be4894c44`
- 创建快捷方式(.lnk)保存链接,后续复用

(来源: https://docs.vrcd.org.cn/books/86vrchat)

### 3.2 模块组件选择

**[FACT]** 改模场景的推荐模块配置:

| 模块 | 是否勾选 | 说明 |
|------|---------|------|
| Microsoft Visual Studio Community 2022 | ❌ 取消 | 改模不必须 |
| Android Build Support | ✅ | VR 一体机/安卓手机版 VRC |
| IL2CPP Support | ✅ | 安卓发布必须 |
| iOS Build Support | ⚠️ 可选 | iOS 移动版支持 |
| 中文语言包 | ✅ | UI 汉化 |

(来源: https://docs.vrcd.org.cn/books/86vrchat)

---

## 4. 汉化

**[FACT]** Unity 2022.3.22f1 简中语言包:

- **在线安装**: 勾选"简体中文"语言包
- **离线汉化**: 下载 `zh-hans.po` → 放入 `*\2022.3.22f1\Editor\Data\Localization`
- 路径: https://new-translate.unity3d.jp/v1/live/54/2022.3/zh-hans

**[COMMUNITY-86vrchat]** **改完语言必须关闭工程重启**
(来源: https://docs.vrcd.org.cn/books/86vrchat)

---

## 5. Quest 支持的 Unity 模块安装（2026-07 补充）

> **来源**: VRCQuestTools 官方文档 (kurotu.github.io/VRCQuestTools/docs/tutorial/set-up-environment)
> **状态**: ✅ FACT-AUTHORITATIVE

### 5.1 必装模块

- **Android Build Support**（必须）

### 5.2 不必装模块（VRChat 创作者）

| 模块 | 必要性 | 说明 |
|------|--------|------|
| **OpenJDK** | ❌ 不必装 | VRCQuestTools 明确:"You don't have to select OpenJDK" |
| **Android SDK & NDK Tools** | ❌ 不必装 | 除非需要在 Android 设备上 Build & Test |

> 🔴 **国内创作者提示**:
> - VCC（VRChat Creator Companion）默认会安装 Android Build Support
> - Unity Hub 手动安装时,OpenJDK / SDK / NDK **全部可不勾选**
> - 验证方法: Unity Editor → File → Build Settings,应能选到 "Android" 平台

### 5.3 常见安装错误

- 错误信息: `Unable to switch to Android build target`
- 原因: 未安装 Android Build Support
- 解决: 重新打开 Unity Hub → 找到对应版本 → 齿轮 → Add Modules → 勾选 Android Build Support

### 5.4 引用

- VRCQuestTools Set up Environment: <https://kurotu.github.io/VRCQuestTools/docs/tutorial/set-up-environment/>

---

## 6. 关联文档

- `FACT.md` §渲染管线 — Unity 版本约束
- `world/development/vm-setup.md` — VM 设置(可能涉及)
- `android-development.md` — Android/Quest 开发基础

---

## 相关页面

[[../world/development/vm-setup.md]] · [[FACT|FACT]] · [[android-development.md]]
