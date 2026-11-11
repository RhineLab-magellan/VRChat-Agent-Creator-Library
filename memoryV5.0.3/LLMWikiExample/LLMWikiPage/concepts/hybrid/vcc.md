---
title: "VRChat Creator Companion (VCC)"
category: hybrid
knowledge_level: applied
status: active
source: "https://vcc.docs.vrchat.com/ (2026-07-01 全站下载)"
source_type: official
version: "2.5.0-beta.2"
last_review: 2026-08-13
confidence: High
tags:
  - hybrid
  - vcc
  - vpm
  - toolchain
  - project-management
aliases:
  - VCC
  - "Creator Companion"
  - "VRChat Creator Companion"
related:
  - alcom.md
  - audio-link.md
  - osc-protocol.md
  - udon-world-plugins.md
type: concept
created: 2026-07-01
sources: "https://vcc.docs.vrchat.com/ (2026-07-01 全站下载) + Creator Companion GitHub releases (2.3.0, 2.5.0-beta.1/beta.2, 2026)"
updated: 2026-08-13
---

# VRChat Creator Companion (VCC)

> **Domain**: Hybrid（Avatar + World 共享 Base 基础设施）
> **定位**: VRChat 官方项目与依赖管理器
> **官方文档**: https://vcc.docs.vrchat.com/
> **本地副本**: （来源：VCC 官方文档本地副本）

---

## 1. 概述

**[FACT]** VCC 是 VRChat 官方推出的桌面应用，负责：

| 功能域 | 职责 |
|--------|------|
| **项目管理** | 创建 / 打开 / 备份 Unity 项目 |
| **VPM 集成** | VRChat Package Manager — 统一包管理 |
| **模板系统** | Avatar / World / UdonSharp 三种项目模板 |
| **SDK 分发** | 2023 年起，SDK 更新仅通过 VCC 推送 |
| **迁移工具** | Legacy `.unitypackage` → VPM 格式自动迁移 |
| **社区仓库** | 第三方 VPM Repo 的添加/管理 |

**平台**: Windows 10/11 (64-bit) — CLI 可运行于 macOS/Linux  
**UI 技术栈**: WebView2（Edge 内核）  
**安装路径**: `%LOCALAPPDATA%\Programs`（可自定义）  
**配置文件**: `%LocalAppData%\VRChatCreatorCompanion\settings.json`

---

## 2. 架构概念

```
┌──────────────────────────────────────────────┐
│              VCC Desktop App                  │
│  ┌────────────┐  ┌──────────┐  ┌──────────┐ │
│  │ Project Mgr│  │ VPM Core │  │ Settings │ │
│  │ (创建/备份) │  │ (依赖解析)│  │ (配置)   │ │
│  └────────────┘  └──────────┘  └──────────┘ │
│         │              │              │       │
│         ▼              ▼              ▼       │
│  ┌──────────────────────────────────────┐    │
│  │         VPM Library (SemVer)         │    │
│  │   依赖解析 · 版本范围 · 包安装/删除    │    │
│  └──────────────────────────────────────┘    │
│         │              │                     │
│         ▼              ▼                     │
│  ┌──────────┐  ┌──────────────┐             │
│  │ Official │  │  Community   │             │
│  │  Repos   │  │   Repos      │             │
│  │(Base/    │  │  (vpm.json)  │             │
│  │ Worlds/  │  │              │             │
│  │ Avatars) │  │              │             │
│  └──────────┘  └──────────────┘             │
└──────────────────────────────────────────────┘
         │
         ▼
┌──────────────────────────────────────┐
│         Unity Project                 │
│  ┌──────────────────────────────┐    │
│  │ Packages/vpm-manifest.json   │    │
│  │ Packages/com.vrchat.core.    │    │
│  │   vpm-resolver/  ← 自动恢复  │    │
│  └──────────────────────────────┘    │
└──────────────────────────────────────┘
```

---

## 3. 核心工作流

### 3.1 创建新项目

```
VCC → New → 选择模板 (Avatar / World / UdonSharp) → 命名 → Create Project
```

**内部过程**:
1. 创建目录 `{location}/{projectName}`
2. 复制选中模板到目标目录
3. 更新 `ProjectSettings` 中的 `productName`
4. 若失败则删除目录并报错
5. 将项目加入 `userProjects[]` 列表

### 3.2 管理已有项目

```
VCC → Add Existing Project → 选择文件夹 → Manage Project → 勾选/升级包
```

- "Installed Version" = 已安装
- "Not Installed" = 未安装
- 点击 `+` 自动安装最新版
- 版本下拉可查看所有历史版本

### 3.3 迁移 Legacy 项目

```
VCC → Add Existing Project → 检测到 Assets/VRCSDK → 显示 Migrate Project 按钮
```

**两种模式**:
- **Migrate a copy**: 创建 `{ProjectName}-Migrated` 副本（安全）
- **Migrate in place**: 直接原地修改（需有备份）

**迁移排除的文件夹**:
`Library`, `Logs`, `Assets\VRCSDK`, `Assets\Udon`, `Assets\UdonSharp`,
`Assets\VRChat Examples`, `Packages\com.vrchat.vrcsdk3`, `ProjectSettings\ProjectVersion.txt`

**特殊包迁移**: UdonSharp、AudioLink、GestureManager 等 Curated Packages 自动从 `/Assets/` 中替换为新版。

### 3.4 添加社区仓库

```
VCC → Settings → Packages → Add Repository → 输入 URL → Add → 确认
```

- 仓库信息存储在 `settings.json` → `userRepos` 数组
- 可通过 cogwheel 按钮添加自定义 Headers（如私有仓库 Token）
- 每个仓库左侧有勾选框控制启用/禁用

---

## 4. VPM 格式速览

### 4.1 三种官方包

| 包名 | 说明 |
|------|------|
| `com.vrchat.base` | Base SDK（World + Avatar 公用） |
| `com.vrchat.worlds` | Worlds SDK（Udon 支持） |
| `com.vrchat.avatars` | Avatars SDK（Avatars 3.0） |

### 4.2 VPM Manifest 关键字段

```json
{
  "name": "com.author.packagename",
  "displayName": "My Package",
  "version": "1.0.0",
  "vpmDependencies": {
    "com.vrchat.worlds": "3.5.x"
  },
  "legacyFolders": {
    "Assets\\OldFolder": "guid..."
  },
  "legacyFiles": {},
  "legacyPackages": []
}
```

- `vpmDependencies` 使用 SemVer 范围（如 `3.5.x` = `>=3.5.0 <3.6.0`）
- `legacyFolders` 自动检测并移除旧版 `.unitypackage` 目录

### 4.3 版本策略: Branding.Breaking.Bumps

VRChat 官方包的准语义化版本：`{Branding}.{Breaking}.{Bumps}`

| 位 | 含义 | 触发条件 |
|----|------|---------|
| Branding | 重大系统变更 | SDK2→SDK3 (2.x→3.x) |
| Breaking | API 不兼容变更 | 需更新依赖包 |
| Bumps | 常规更新 | Bug 修复 + 新功能 |

> **[FACT]** SDK 3.3.0 首次声明 Public SDK API。该 API 内的接口不会因 Bumps 升级而破坏。

---

## 5. Package Resolver

**[FACT]** `com.vrchat.core.vpm-resolver` 是唯一**不可移除**的 VPM 包。

**工作原理**:
1. Unity 打开项目时，对比 `Packages/vpm-manifest.json` 与实际 `Packages/` 目录
2. 发现缺失 → 弹出对话框提示
3. 自动下载并安装缺失的包（无需 VCC 客户端）

**Git 集成**:
- `.gitignore` 排除所有 `com.vrchat.*` 包，但**保留** `com.vrchat.core.*` (Resolver)
- 克隆后首次打开 Unity → Resolver 自动恢复依赖

---

## 6. 已知问题

> ⚠️ **社区层补充**: 中国大陆社区长期存在 patcher / mirror / 加速类 workaround，用于改善下载可达性；这些方案不属于 VCC 官方主干，应与本页的官方结论分开看待。具体步骤优先留在 `源资料层《vcc》` 或参考层文档。

| 问题 | 严重度 | 来源 | 状态 |
|------|--------|------|------|
| **"Failed to add Repo" 误报** | 🟡 中等 | `modular-avatar.md §8.2` | VCC 已知 bug，99% 情况仓库已成功添加，点 Cancel 即可 |
| **Pre-release 依赖解析 bug** | 🟡 中等 | `modular-avatar.md` | MA 预发布版在 VCC 中依赖解析失败，建议用 ALCOM |
| **WebView2 白屏** | 🔴 严重 | FAQ | 重装 WebView2 或用 `http://localhost:5476/` 浏览器访问 |
| **长路径导致包丢失** | 🟡 中等 | FAQ | Windows 路径过长，建议 `<盘符>:\Projects\` 短路径或启用 Win32 长路径 |
| **settings.json 损坏** | 🔴 严重 | FAQ | 删除 `%LocalAppData%\VRChatCreatorCompanion\settings.json` 后重启 VCC |

---

## 7. CLI (`vpm`)

**[FACT]** VCC 提供独立的 CLI 工具，通过 .NET 8 安装：

```bash
dotnet tool install --global vrchat.vpm.cli
```

**常用命令**:
```bash
vpm new MyProject World              # 创建 World 项目
vpm add package com.vrchat.udonsharp # 安装包
vpm list projects                    # 列出项目
vpm migrate legacy ./MyProject       # 迁移 Legacy 项目
vpm add repo https://example.com/vpm.json  # 添加仓库
vpm install templates                # 安装最新模板
vpm install unity                    # 安装兼容的 Unity 版本
```

> 完整 CLI 参考见 （来源：VCC 官方文档本地副本），包含 `migrate 2022`、`--headers`、`--inplace` 等高级用法。

---

## 8. 2026 版本增量

### 8.1 Creator Companion 2.3.0（2026-03-21）

- 增加 Open Beta 模式，Beta 数据放入独立目录，不影响主安装。
- 支持在 VCC 内直接 Resolve 项目包，并提供 `Reinstall All Packages`。
- 日志界面增加类型筛选、全文搜索和更详细的启动/操作日志。
- 改进 legacyFiles/legacyFolders 处理，降低项目文件误删风险。

### 8.2 Creator Companion 2.5.0 Beta（2026-06-18/25）

- 支持全局或单项目 Unity Editor launch parameters。
- 包有 `changelogUrl` 时，在 VCC 中直接打开包更新日志。
- 引入版本化的 `settings_v2.json`；首次启动会迁移现有设置。
- 启动时备份设置与项目数据库，便于从 `.bak` 恢复。
- 重构仓库缓存：仅刷新已启用仓库，过期缓存超过 1 小时才重新获取。
- VPM CLI 增加 `list packages`、`list repos`、`repo toggle`，并降低对本机 Unity/Unity Hub 配置的依赖。

来源：[FACT] [Creator Companion releases](https://github.com/vrchat-community/creator-companion/releases)

> **版本边界**：上述 2.5.0 内容来自 Beta release，不能直接视为稳定版行为。升级前应备份 `settings_v2.json`、项目数据库和 Unity 项目。

## 9. 定位总结

> VCC = 项目创建器 + 包管理器 + SDK 分发器 + 迁移工具

- **对于 World 创作者**: 创建 World/UdonSharp 项目，管理 UdonSharp/ClientSim/AudioLink 等包
- **对于 Avatar 创作者**: 创建 Avatar 项目，管理 MA/AAO/VRCFury 等 VPM 包
- **对于工具开发者**: 使用 Package Template + Package Listing 模板分发自己的工具

> **与 ALCOM 的关系**: ALCOM 是社区开发的开源替代品，共享配置文件，可共存。详见 `alcom.md`。

---

## 10. 第三方工具分发模式对比（2026-07 补充）

> 来源: VirtualLens2 + VRCLens 评估

### 10.1 VRChat 创作者经济的两种成熟模式

| 模式 | 代表工具 | 渠道 | 价格 |
|------|----------|------|------|
| **VPM 免费 + Patreon/捐赠** | VRCFury / MA / AAO / UVU / VVMW | VPM 仓库 | 免费 |
| **Gumroad/BOOTH 独家付费** | VRCLens / VirtualLens2 | 作者官网 | 付费 |

### 10.2 VRCLens 分发渠道反例

**销售渠道**:
- **官方销售**: <https://hirabiki.gumroad.com/l/rpnel>
- **独占性**: 仅在作者官网销售
- **不在 BOOTH 等其他正版平台销售**

**价格模式**:
- **Pay What You Want** ($11 起步)

### 10.3 VirtualLens2 分发

- 通过 BOOTH 销售（unitypackage）
- Pay What You Want 模式（¥1,500 起步）

### 10.4 与 VPM 对比

| 工具 | 分发渠道 | 价格 | VPM 支持 |
|------|----------|------|----------|
| VRCFury | VPM (`vcc.vrcfury.com`) | 免费 | ✅ |
| Modular Avatar | VPM (`vpm.nadena.dev`) | 免费 | ✅ |
| AvatarOptimizer | VPM (`vpm.anatawa12.com`) | 免费 | ✅ |
| UVU | VPM | 免费 | ✅ |
| VVMW | VPM | 免费 | ✅ |
| **VirtualLens2** | BOOTH（unitypackage）| 付费 ($11+) | ❌ |
| **VRCLens** | **Gumroad 独家**（unitypackage）| 付费 ($11+) | ❌ |

### 10.5 业务判断

> **【元知识】**: VRChat 创作者经济存在两种成熟模式：
> 1. **VPM 免费 + Patreon/捐赠** (主流开源工具)
> 2. **Gumroad/BOOTH 独家付费** (个人开发者商业模式)
>
> VRCLens / VirtualLens2 是第二种模式的成功代表，证明个人开发者通过单一渠道 + 持续更新是可行的。

### 引用

- [L2] Hirabiki 官方: <https://hirabiki.gumroad.com/l/rpnel>
- [L2] logi_9 / ろじらぼ BOOTH: <https://logilabo.booth.pm/>
- 详见 [[entities/avatar/camera-tools]] §3.6

---

## 相关页面

[[alcom.md]] · 源资料层（原始归档，不随发布包分发） · [[audio-link.md]] · [[osc-protocol.md]] · [[udon-world-plugins.md]]
