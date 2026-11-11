---
title: "NDMF 工具生态体系"
category: avatar
knowledge_level: applied
status: active
source: NDMF 官方文档站与 GitHub 发布页，2026-11-12 核对；VPM 列表地址同日实测
source_type: official
version: 2.0
upstream_version: "1.14.8 (2026-08-29)"
last_review: 2026-11-12
confidence: High
tags:
  - avatar
  - ndmf
  - non-destructive
  - modular-avatar
  - vrcfury
  - vpm
  - build-pipeline
aliases:
  - "NDMF 工具生态体系"
  - ndmf-tools.md
related:
  - modular-avatar.md
  - ma-component-cards.md
  - vrcfury-reference.md
  - avatar-optimizer.md
  - optimization-guide.md
  - performance-rank.md
type: entity
created: 2026-07-04
sources: https://ndmf.nadena.dev/ (2026-11-12, HTTP 200) + https://ndmf.nadena.dev/versioning-policy.html (200) + https://ndmf.nadena.dev/api/index.html (200) + https://api.github.com/repos/bdunderscore/ndmf/releases/tags/1.14.8 (200) + https://vpm.nadena.dev/vpm.json (200)
updated: 2026-11-12
---
# NDMF 工具生态体系

> **核对日期：2026-11-12。** 核对当日 NDMF 的稳定发布为 **1.14.8**，该版本发布于 **2026-08-29**（`https://api.github.com/repos/bdunderscore/ndmf/releases/tags/1.14.8`，HTTP 200）。本页只陈述该日核对所见，其后不再作任何时效承诺；再次核对时以 NDMF 的发布页为准。同日的预发布线为 `1.15.0-beta.5`（2026-11-08），它**不是**稳定版本。
> 来源：<https://ndmf.nadena.dev/>、<https://github.com/bdunderscore/ndmf>
> 本页为 2026-11-12 重写版；旧版（1.3，2026-08-13）中今日官方文本已不支持的断言，逐条处理见 §7。

---

## 0. 名字与定位

官方文档开头给出两句话，一句定义、一句缩写来历：[FACT-OFFICIAL]（<https://ndmf.nadena.dev/>，2026-11-12，HTTP 200）

> "NDM Framework ('Nademof' for short) is a framework for running non-destructive build plugins when building avatars for VRChat (and, eventually, for other VRSNS platforms)"

即：NDMF 不是"一个优化工具"，而是**一个让非破坏性插件按序执行的框架**；具体优化由插件（如 Modular Avatar、AvatarOptimizer）实现。官方用了 "build plugins" 一词，而它工作的时机不止 build——见 §2。

---

## 1. 它解决什么问题

官方给出的理由是工程性的，不是风格问题：[FACT-OFFICIAL]（同上）

- VRChat SDK 提供的是 **build-time 回调**，没有播放模式的挂点。
- 于是每个插件若各自在播放模式里做自己的逻辑，多个插件同时存在就会互相干扰——官方原文说这种兼容性问题 "this has happened already"。
- 因此 NDMF 的首要目标是：**多个非破坏性插件同时加载时的兼容性**，办法是把执行顺序统一到一个框架里。

⇒ 判断方法：一个工具"支持 NDMF"意味着它把改动交给框架排序执行，而不是自己抢时机。[INFERENCE]

---

## 2. 执行模型

README 把框架能力列成四项：[FACT-OFFICIAL-REPO]（<https://github.com/bdunderscore/ndmf>，README，2026-11-12，HTTP 200，6556 字符）

| 能力 | 说明 |
|------|------|
| 按阶段排序执行 | 插件声明自己在哪个高层阶段跑，以及 runs-before / runs-after 关系 |
| 两个触发点 | **进入播放模式时**、**构建化身时** |
| 临时资产的生命周期 | 生成/保存临时资产，并在构建结束后清理 |
| 动画路径修正 | 物体被移动后，自动修正动画路径 |

这两个触发点值得记住：NDMF 的改动**在播放模式里就能看到**，不必先上传。[FACT-OFFICIAL-REPO]

写一个最小插件的形状（README 原文示例）：[FACT-OFFICIAL-REPO]

```csharp
[assembly: ExportsPlugin(typeof(MyPlugin))]
class MyPlugin : Plugin<MyPlugin> {
    protected override void Configure() {
        InPhase(BuildPhase.Transforming).Run("Do something", ctx => { /* ... */ });
    }
}
```

README 一并交代了 `BuildContext`、`BuildContext.GetState<T>()`、扩展上下文（`TrackObjectRenamesContext`、`ModularAvatarContext`）以及 `RequiredContexts` / `CompatibleContexts` 的用法。[FACT-OFFICIAL-REPO]

> 说明：本环境访问 `raw.githubusercontent.com` 会被 TLS 拒绝（`write EPROTO … TLSV1_ALERT_DECODE_ERROR`），因此上面的 README 正文是通过 GitHub API 的 readme 端点取得的，不是从 raw 域名抓的。[INFERENCE]

---

## 3. 版本策略（为什么"升级小版本"可能改变执行顺序）

官方 versioning policy 页的规则：[FACT-OFFICIAL]（<https://ndmf.nadena.dev/versioning-policy.html>，2026-11-12，HTTP 200）

| 位 | 允许的变更 |
|----|------------|
| major | 破坏性变更（移除 API 成员、给抽象类加抽象方法等） |
| minor | 新增能力，**以及执行顺序启发式（heuristics）的变化** |
| patch | 向后兼容的修复 |

关键在 minor 那一行：**顺序启发式可能在 minor 版本里变**。所以依赖特定执行顺序的插件/教程必须显式声明 runs-before / runs-after，而不能依赖"它应该会先跑"。[INFERENCE]

---

## 4. 安装

NDMF 通过 VPM 仓库分发。2026-11-12 实测：`https://vpm.nadena.dev/vpm.json` → HTTP 200，427998 字节，**声明 4 个包**：`nadena.dev.misc-vrc-tools`、`nadena.dev.modular-avatar`、`nadena.dev.modular-avatar.resonite`、`nadena.dev.ndmf`。

`nadena.dev.ndmf` 就是 NDMF 本体；它与 Modular Avatar 在**同一个**库里，这也是很多人以为"装 MA 就自带 NDMF"的原因。[INFERENCE]

发布附件（1.14.8）：`nadena.dev.ndmf-1.14.8.zip` 与 `package.json`。[FACT-OFFICIAL-REPO]（<https://api.github.com/repos/bdunderscore/ndmf/releases/tags/1.14.8>，HTTP 200）

---

## 5. 1.14.8 修了什么（发布正文原文）

[FACT-OFFICIAL-REPO]（同上）

> Version 1.14.8
> ## Notable changes
> ### Fixed
> - [#832] Fixed regression in resonite builds resulting in `Unable to determine viewpoint position` error.
> - [#826] ErrorReport.CaptureErrors did not correctly capture errors in certain build contexts

两条都是**修复**：Resonite 构建的视角位置回归、以及错误上报在某些构建上下文里抓不到错误。[FACT-OFFICIAL-REPO]

时间线（发布页，2026-11-12 读取）：稳定线 1.14.4（2026-08-09）→ 1.14.6（08-22）→ 1.14.7（08-28）→ 1.14.8（08-29）；此后稳定线停在 1.14.8，预发布线继续推进到 1.15.0-beta.5（2026-11-08）。[FACT-OFFICIAL-REPO]

⇒ 阅读含义：8 月底以后没有新稳定版，不等于项目停更；beta 线仍在动。写"最新版本"时必须带日期与"稳定/预发布"的限定。[INFERENCE]

---

## 6. 生态：VPM 列表地址实测（2026-11-12）

下表是**实测记录**（HTTP 状态与声明的包数），不是厂商声明，因此不带 `[FACT-OFFICIAL*]` 标记；每行给出地址，读者可自行复核。

| 工具线 | 列表地址 | 状态 | 声明包数 |
|--------|----------|------|----------|
| NDMF + Modular Avatar | `https://vpm.nadena.dev/vpm.json` | 200 | 4 |
| AvatarOptimizer 等 | `https://vpm.anatawa12.com/vpm.json` | 200 | 14 |
| TexTransTool 等 | `https://vpm.rs64.net/vpm.json` | 200 | 9 |
| LAC / mono-ui | `https://vpm.limitex.dev/index.json` | 200 | 2 |
| Meshia 等 | `https://ramtype0.github.io/VpmRepository/index.json` | 200 | 2 |
| VRCQuestTools 等 | `https://kurotu.github.io/vpm-repos/index.json` | 200 | 4 |
| puddingkc 系列 | `https://null-k.github.io/vpm-listing/index.json` | 200 | 3 |

两个常见的地址错误，实测记下来：`https://vpm.anatawa12.com/index.json` → 404（正确是 `/vpm.json`）；`https://lilxyzw.github.io/vpm-repos/vpm.json` 与 `/index.json` 都返回 HTML 404 页而不是 JSON，lilToon 的列表地址本页**不确认**。

---

## 7. 本版重写：旧页断言的处理

| 旧页断言 | 今日处理 | 依据 |
|----------|----------|------|
| "NDMF (Non-Destructive Modular Framework)" 这个全称 | 删除；官方只写 "NDM Framework ('Nademof')" | 文档首页原文 |
| "大部分工具依赖 Modular Avatar 运行" | 删除；NDMF 与 MA 是同一库里的两个独立包，框架不依赖 MA | `vpm.nadena.dev` 包清单 |
| "NDMF 只在编译/构建时生效" | 改为官方 README 的两个触发点：播放模式 + 构建 | README |
| 旧页的工具清单以"本地笔记"为准 | 改为 2026-11-12 实测的列表地址与包数 | §6 实测 |
| 旧页 `confidence: High` 与页内标记不一致 | 由记录的置信度规则重算，结果写在 front matter | 规则输出 |

---

## 8. 和 VRCFury 的关系

VRCFury 与 NDMF 是两套各自完整的非破坏性系统：VRCFury 从 `https://vcc.vrcfury.com` 安装，NDMF 从 `https://vpm.nadena.dev/vpm.json` 安装。[INFERENCE]（两处安装说明分别见 <https://vrcfury.com/download> 与 §4）

VRCFury 官方文档在 2026-11-12 核对时**没有**说明它是否通过 NDMF 执行、或是否与 NDMF 插件共存有特殊要求；本页不替它下结论。VRCFury 自身的能力见 [[vrcfury-reference]]。
