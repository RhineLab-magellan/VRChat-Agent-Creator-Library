# VRChat-Agent-Creator-Library

> 🧠 **VRChat 专属 AI Agent 开放记忆库 v5.0.0** — LLMWikiPage 自包含知识包 · 机器可解析 · 多领域 VRChat 创作者知识体系

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version: v5.0.0](https://img.shields.io/badge/Version-v5.0.0-blue.svg)](CHANGELOG.md)
[![Files: 414](https://img.shields.io/badge/Files-414-blue.svg)]()
[![Language: 中文](https://img.shields.io/badge/Language-中文-red.svg)]()
[![Architecture: LLMWikiPage](https://img.shields.io/badge/Architecture-LLMWikiPage-purple.svg)]()
[![Quality: Self-contained](https://img.shields.io/badge/Quality-Self--contained-brightgreen.svg)]()
[![Dead Links: 0](https://img.shields.io/badge/Dead_Links-0-brightgreen.svg)]()

---

## 📖 简介

**VRChat-Agent-Open-Memory-Library v5.0.0** 是以「**自包含知识发布包**」为目标的 VRChat 创作者知识库。它由长期工作库（memoryV4.0.0，已归档）整理为 **LLMWikiPage 结构**：414 个 Markdown 知识页，全部使用 `[[相对wikilink]]`/相对链接/`http(s)://` URL 相互连接，**零硬编码本地路径**（正文路径示例已参数化，无个人路径；包内 soft raw 锚点已改写为公开 URL/抽象描述）。

- **414 个** Markdown 知识页，约 3.90 MB
- **四指标发布就绪**（`publish-readiness-check.py` 复跑验证）：C1 自包含（死链 0 / 站点相对 URL 0）· C2 置信度（非法 0）· C3 Fact Index（孤页 0 / 索引死链 0）· C4 标签（0 缺失 / 分类法 150）
- **100% frontmatter** — 任意页面可被机器解析，可直接喂给 RAG / 向量化
- **0 死链** — wikilink 与相对文件链接全库可解析
- 覆盖 **10+ 领域**：World（202 页）、Avatar（99 页含 Shader 子域）、API、Platform、VRChatSDK、Rules、Patterns、Hybrid、Misc、Comparisons、Queries、Meta

> 相比 v3.0.0 的「SOUL + Memory」工作库形态，v5.0.0 是**对外发布形态**：沉淀为纯知识包，Agent 身份 / 工作模式等治理内容收敛到 `meta/`，源材料（raw/、sources/ 注册表）不随包分发。
> 上一代工作库 `memoryV4.0.0`（466 文件，含治理工具/_curator_tools/.obsidian 等）已归档，需要时从本地历史归档取回。

---

## 🗂️ 目录结构

```
VRChat-Agent-Open-Memory-Library/
│
├── README.md                  # 📖 本文件
├── CHANGELOG.md               # 📝 版本更新日志
├── LICENSE                    # 📜 MIT License
│
├── memoryV5.0.0/              # ⭐ 知识发布包（GitHub Release 仅分发此文件夹）
│   ├── README-发布说明.md       #   发布说明（就绪判定口径）
│   └── LLMWikiExample/
│       └── LLMWikiPage/       # 📚 知识页根（414 md，唯一知识入口）
│           ├── FACT.md            # 知识库结构树 + 核心约束 + 长期事实
│           ├── index.md           # 顶层索引（路由地图 + Obsidian MOC）
│           ├── _always-load.md    # 每会话必载的跨域核心约束
│           ├── entities/    (351) # 实体域：api/ avatar/(含shader/) platform/ vrchatsdk/ world/
│           ├── concepts/    (47)  # 概念域：hybrid/ misc/ patterns/ rules/
│           ├── comparisons/ (6)   # 对比分析：shader/工具/架构
│           ├── queries/     (1)   # 优质问答归档
│           └── meta/        (6)   # 治理协议：working-modes/kb-protocol/obsidian-governance 等
│
├── reports/                   # 📋 审查/发布就绪报告（不随发布包分发）
├── Auxiliary script/          # 🔧 历史维护脚本（治理/审计/迁移工具）
└── Training data/             # 📤 社区贡献入口（贡献指南见下方）
```

### 页面数量明细（实测，2026-08-20）

| 域 | 页数 | 内容 |
|---|---|---|
| `entities/world/` | 202 | Udon 编程、网络同步、Persistence、Scene Components、示例、光照烘焙、工具 |
| `entities/avatar/` | 99 | Animator、改模优化、Shader（lilToon/Poiyomi/ORL/Filamented/UnlitWF）、工具链 |
| `entities/api/` | 19 | Networking/Persistence/Player/动画/音频/UI API 参考 |
| `entities/vrchatsdk/` | 19 | VRChat HTTP API、WebSocket、数据模型 |
| `entities/platform/` | 12 | Quest/Android、跨平台、Unity 工具链 |
| `concepts/patterns/` | 21 | 同步/状态机/消息总线等设计模式 |
| `concepts/rules/` | 10 | UdonSharp 语言限制/网络/性能/VM 硬约束 |
| `concepts/hybrid/` | 12 | OSC、AudioLink、VCC/ALCOM、桥接 |
| `concepts/misc/` | 4 | 后处理、无障碍等 |
| `comparisons/` | 6 | Poiyomi vs lilToon、FIL-ORL-Poiyomi Pro 等对比 |
| `meta/` + 顶层 3 | 9 | 治理协议 + FACT/index/_always-load |

> v5.0.0 相比工作库新增约 50 页 2026-08 知识：Unity 场景组件专页（AudioSource/Light/NavMesh/ParticleSystem/RigidBody/Collider/Joint/Camera/Canvas 等约 45 页）、工具页（udonsharp-linter、unity-mcp、dressing-tools、gesture-manager、lilycal-inventory、faceemo、runtime-texture-compression、physbones、avatar-audit-methodology）、VRCBillboard API、unity-audio-filters/unity-constraints/unity-rendering-aux 专页。

---

## 🏷️ 知识分类体系

### 1. 域分类（物理目录即域）

知识页按 `entity`（实体：API/组件/工具/平台）与 `concept`（概念：规则/模式/混合域）二分，再按域细分；`comparisons` 承载跨实体对比，`queries` 承载问答归档。

### 2. 可信度分级（confidence）

| 级别 | 含义 | v5 分布 |
|------|------|--------|
| `high` | 官方文档 / 官方源码 / 直接实验验证 | 204 页（49.3%）|
| `medium` | 多源交叉一致 / 高质量社区结论 | 208 页（50.2%）|
| `low` | 社区经验 / 推断，**必须带来源与理由** | 2 页（0.5%）|

> low 2 页：`entities/avatar/commission-pricing.md`、`entities/platform/meta-quest-troubleshooting.md`，均已声明推断性质与来源。

### 3. YAML frontmatter（100% 覆盖）

```yaml
---
title: 文档标题
type: entity | concept | comparison | query | summary
category: 域
knowledge_level: applied | theoretical | reference
status: active | draft | deprecated
tags: [tag1, tag2, ...]
aliases: [别名1, 别名2]
related: [ [关联] ]
source: 知识来源
source_type: official | community | inference
confidence: high | medium | low
created: 2026-08-13
updated: 2026-08-20
---
```

---

## 🔌 不同 Agent 系统加载方式

本库为**自包含 Markdown 知识包**，任何可读取 `.md` 的系统均可直接加载：

### 1. Ollama/LangChain/RAG 向量化（推荐）

把 `memoryV5.0.0/LLMWikiExample/LLMWikiPage/` 作为知识根喂给切块管线；每页 frontmatter 提供 `tags`/`type`/`confidence`，链接为 `[[相对wikilink]]` 或 URL（可直接用 Obsidian/graph 渲染，也可用 `[[title]]` 拆分为检索标签）。

### 2. Cursor / Claude Code / Copilot

```yaml
# AGENTS.md / CLAUDE.md / .cursorrules
memory_root: memoryV5.0.0/LLMWikiExample/LLMWikiPage/
boot_files:
  - memoryV5.0.0/LLMWikiExample/LLMWikiPage/_always-load.md
  - memoryV5.0.0/LLMWikiExample/LLMWikiPage/FACT.md
```

### 3. Obsidian（人类阅读 / 图谱导航）

把 `LLMWikiPage/` 作为 Vault 打开即可；`index.md` 是路由地图，`FACT.md` 是知识库结构树，内链与图谱立即可用。

### 4. 通用 LLM Chat（按需加载）

```
1. 附加 _always-load.md（跨域核心约束）
2. 附加 FACT.md（知识库结构 + 核心约束）
3. 按需附加 index.md 路由到的具体域文件
```

> ⚠️ 通用 Chat 上下文窗口有限，建议按需加载具体域文件，不要一次加载全部 414 个文件。

---

## 🚀 快速开始

| 我想做什么 | 从哪里开始 |
|-----------|-----------|
| 学习 Udon 编程 | `entities/world/udon/index.md` → `entities/world/udon/udonsharp/` |
| 解决网络同步问题 | `concepts/rules/networking-rules.md` → `concepts/patterns/manual-sync-state.md` |
| 优化 World 性能 | `concepts/rules/performance-rules.md` → `entities/world/performance-guide.md` |
| Avatar 改模入门 | `entities/avatar/modular-avatar.md` → `entities/avatar/teaching-methodology.md` |
| 选择 Avatar Shader | `entities/avatar/shader/index.md`（lilToon/Poiyomi/ORL/Filamented/UnlitWF 对比矩阵）|
| Poiyomi shader 细节 | `entities/avatar/shader/poiyomi/index.md` + `comparisons/poiyomi-pro-vs-toon.md` |
| 审查 UdonSharp 代码 | `concepts/rules/udonsharp-language-limits.md` → `concepts/rules/udonsharp-deep-pitfalls.md` |
| 开发外部应用 | `entities/vrchatsdk/index.md` |
| Quest 适配 | `entities/platform/android-development.md` |
| 管理 VCC/ALCOM | `concepts/hybrid/vcc.md` → `concepts/hybrid/alcom.md` |
| 查阅 API 签名 | `entities/api/` 目录 grep 关键词 |
| OSC 协议 | `concepts/hybrid/osc-protocol.md` |

> 旧版 `memory/` 路径已随 v5.0.0 迁移：`memory/world/...` → `entities/world/...`，`memory/patterns/` → `concepts/patterns/`，`memory/rules/` → `concepts/rules/`，`memory/hybrid/` → `concepts/hybrid/`，`memory/misc/` → `concepts/misc/`，`memory/reviews/`（治理层）未随发布包携带。

---

## ⚠️ 核心约束速查（工程实践红线）

> VRChat World/Avatar 开发绝对红线，详见 `_always-load.md` 与 `concepts/rules/`：

| # | 约束 | 后果 |
|---|------|------|
| 1 | **只支持 BRP 渲染管线**（禁止 URP/HDRP） | 项目完全无法工作 |
| 2 | Editor 脚本必须放在 `Editor` 文件夹内 | 跨平台构建失败 |
| 3 | Unity 版本锁定 2022.3.22f1 LTS（SDK 3.5.0+） | 版本不兼容 |
| 4 | 禁止 List/Dictionary/LINQ/lambda | Udon VM 不支持 |
| 5 | 禁止 async/await/try-catch/coroutine | Udon VM 不支持 |
| 6 | 非 Owner 禁止写入 UdonSynced 变量 | 同步数据被覆盖 |
| 7 | Manual Sync 必须调 RequestSerialization() | 数据永不发送 |
| 8 | Update 中禁止 GetComponent/Find | 严重性能问题 |
| 9 | 同步 String/DisplayName 需特殊处理 | 带宽浪费 |
| 10 | Enum 比较必须 cast 到 int | 装箱导致 GC |

---

## 📊 知识库数据来源

| 来源类别 | 具体来源 | 对应目录 |
|---------|---------|---------|
| 🏛️ **VRChat 官方** | Creator Docs、SDK 源码、Udon VM 规范、Release Notes | `entities/world/udon/` `entities/api/` `concepts/rules/` |
| 🏛️ **VRChat API** | HTTP API 官方文档、WebSocket 规范 | `entities/vrchatsdk/` |
| 🔧 **核心工具** | Modular Avatar、VRCFury、lilToon、Poiyomi、AAO、ClientSim、VCC/ALCOM | `entities/avatar/` `entities/world/` `concepts/hybrid/` |
| 📦 **开源项目** | Sardinal、ULocalization、UdonVoiceUtils、LuraSwitch2、VizVid | `concepts/patterns/` `entities/world/` |
| 👥 **社区智慧** | VRCD 文档库、DeepWiki、Discord 讨论、创作者笔记 | `concepts/misc/` `comparisons/` |
| ✍️ **开发实践** | 个人笔记、项目经验、审查记录 | `meta/` `concepts/rules/` |

---

## ✅ 发布质量（2026-08-20 复核口径）

检查依据 `LLMWiki/SCHEMA.md`（frontmatter / 标签分类法 §4 / 置信度 §9 / 链接规范 §7），`publish-readiness-check.py --json` 幂等两跑 SHA-256 一致：

| 指标 | 判据 | 结果 |
|---|---|---|
| C1 自包含 | 死链文件链接 = 0；站点相对 URL = 0；wikilink 死链 = 0 | ✅ 0 / 0 / 0 |
| C2 置信度 | high/medium/low 缺失或非法 = 0 | ✅ 0 |
| C3 Fact Index | 孤页 = 0；index 死链 = 0 | ✅ 0 / 0 |
| C4 标签 | tags 缺失 = 0；分类法 150；分类法外复用（≥2 次）= 0 | ✅ 0 |

> 交叉口径：site-scan 知识域 363 页（死链 0 / frontmatter 100%）、factcheck-audit 411 页（sources 结构化锚定率 29.4%，锚定缺口 0）。
> 已知保留项（修复轮后终态，非缺陷）：绝对路径示例已参数化、soft raw 锚点已改公开 URL/抽象描述，全库零硬编码本地路径；2 页 low 置信（定性参考 / 草稿）诚实标注。
>
> 📋 **独立 V5 审查（2026-08-20）**：连接自包含复查确认 **本地文件连接 = 0**（库内相对路径可接受，全库自包含），四指标独立复跑全门 0（幂等 SHA 一致）。审查报告见 [`reports/V5独立审查报告-2026-08-20.md`](reports/V5%E7%8B%AC%E7%AB%8B%E5%AE%A1%E6%9F%A5%E6%8A%A5%E5%91%8A-2026-08-20.md) 与 [`reports/发布检查报告-2026-08-20.md`](reports/%E5%8F%91%E5%B8%83%E6%A3%80%E6%9F%A5%E6%8A%A5%E5%91%8A-2026-08-20.md)；报告存放于仓库根 `reports/`，**不随发布包分发**（Release 仅含 `memoryV5.0.0/`）。

---

## 📈 项目状态

| 指标 | v5.0.0 | v3.0.0 |
|------|--------|--------|
| 📁 知识页 | **414**（自包含包） | 353（工作库）|
| 📂 总大小 | ~3.90 MB | ~3.26 MB |
| 🗂️ 结构 | LLMWikiPage（entities/concepts/comparisons/queries/meta）| SOUL+Memory（12 域）|
| ✅ YAML 覆盖率 | **100%** | 100% |
| 🔗 死链数 | **0** | 0 |
| 🔍 发布口径 | 四指标全过（C1–C4）| 99.6% 搜索可达 |
| 📅 更新日期 | 2026-08-20 | 2026-07-01 |

---

## 📝 版本历史

| 版本 | 日期 | 核心变化 |
|------|------|---------|
| **v5.0.0** | 2026-08-20 | **发布包形态**：LLMWikiPage 自包含知识包（414 页）。目录重构为 entities/concepts/comparisons/queries/meta；新增约 50 页（Unity 组件专页、工具页、VRCBillboard 等）；四指标发布就绪（C1–C4 全过）；工作库（memoryV4.0.0）归档。 |
| **v4.0.0** | 2026-07—08 | 工作库阶段（466 文件，未对外发布）：新容器/工具页与治理沉淀，作为 v5.0.0 的数据源。 |
| **v3.0.0** | 2026-07-01 | 内容规模化扩张：Poiyomi Shaders 8 主题 + 玩家操作知识库 + VCC/ALCOM + OSC + VPM 镜像站。A18/A19/A24 治理闭环。353 篇。 |
| **v2.0.0** | 2026-06-21 | SOUL.md Agent 身份系统 + YAML frontmatter 标准化 + meta/ 治理体系。 |
| **v1.0.0** | 2026-06-20 | 初始版本，295 文件，6 大领域。 |

> 详细变更见 [CHANGELOG.md](CHANGELOG.md)。

---

## 🤝 贡献指南

本库为 **AI Agent 优先 + 人类创作者共用** 的 VRChat 技术知识体系。如希望贡献，请在 `Training data/` 目录打包上传以下资料：

1. **原始知识文件**：官方文档 / 个人笔记 / QA / 工程资料
2. **可信度标注**：按 high/medium/low 分级，区分 FACT 与 推断
3. **YAML 元数据**：遵循 `meta/kb-protocol.md` 的 frontmatter 规范
4. **交叉引用**：在相关页间用 `[[wikilink]]` 建立双向链接
5. **格式规范**：遵循现有页面的 Markdown 结构与元数据格式
6. **审查验证**：资料类确认内容正确性；工程类确认在最新版 SDK 可运行

或者加入 QQ 群聊 `902222352`，打包提供上述文件并说明知识来源的可靠性与您的审查结论。采用后由 Master 二次审查 + 模型自审，确定置信度后再纳入知识库。

---

## 📝 许可

本项目采用 [MIT License](LICENSE) 开源。

**知识共享声明**：本库中的知识来源于 VRChat 官方文档、开源项目文档、社区公开讨论，以及作者的个人实践笔记。所有第三方知识均标注原始来源。如果您发现任何版权问题，请提交 Issue。
