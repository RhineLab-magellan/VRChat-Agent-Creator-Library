# Changelog

## v5.0.0 (2026-08-20) — LLMWikiPage 自包含知识发布包

### 形态转型
- 从「SOUL + Memory 工作库」转为「纯知识发布包」：知识根 = `memoryV5.0.0/LLMWikiExample/LLMWikiPage/`（414 个 Markdown 知识页，约 3.90 MB）
- 目录重构为 `entities`（351）`concepts`（47）`comparisons`（6）`queries`（1）`meta`（6）+ 顶层 `FACT.md`/`index.md`/`_always-load.md`
- 全库自包含：仅依赖可解析 `[[wikilink]]` 与 `http(s)://` URL，**零硬编码本地路径**（正文路径示例已参数化，无个人路径）

### 内容增量（2026-08）
- Unity 场景组件专页约 45 页（AudioSource/Light/NavMeshAgent/ParticleSystem/RigidBody/Collider/Joint/Camera/Canvas/Renderer/VideoPlayer 等）
- 工具页：udonsharp-linter、unity-mcp、dressing-tools、gesture-manager、lilycal-inventory、faceemo、runtime-texture-compression、physbones、avatar-audit-methodology
- API/组件：vrc-billboard、unity-audio-filters、unity-constraints、unity-rendering-aux、vrc-rendertexture

### 质量治理
- 发布就绪四指标全过（`publish-readiness-check.py` 幂等验证）：
- **审查循环修复轮**（修复后全指标复跑归零）：绝对路径示例参数化（10 页→0）；soft raw 锚点改公开 URL/抽象描述（4→0）；vrchatsdk 18 页 `source` 补全真实官方/社区 URL；全库旧工作库层（raw/sources/references/reviews）路径文本清除（44 文件）；obsidian-governance 重写（去个人路径、拓扑对齐）；queries 归档填充 5 条真实问答；重复 tag 清除；库内相对路径提示全量解析校验 0 缺失。复跑：C1–C4 全门 0，factcheck 锚定缺口 0，site-scan 死链 0。
  - C1 自包含：死链文件链接 0 / 站点相对 URL 0 / wikilink 死链 0
  - C2 置信度：非法值 0（high 204 / medium 208 / low 2，low 已带来源+理由）
  - C3 Fact Index：孤页 0 / index 死链 0（FACT.md 结构树 = 44 目录）
  - C4 标签：缺失 0 / 分类法 150 / 分类法外复用（≥2 次）0
- 交叉口径：site-scan 知识域 363 页（死链 0 / frontmatter 100%）；factcheck-audit 411 页

### 结构变更
- 移除旧 `memory v3.0.0/` 工作库目录
- 工作库 `memoryV4.0.0/`（466 文件）归档（不随发布包分发）
- 治理层（reviews/_curator_tools/.obsidian/备份/源注册表）不随发布包携带，治理内容收敛至 `meta/`

### 发布前独立审查（2026-08-20）
- **独立 V5 连接自包含审查**：本地文件连接 = 0（publish-readiness C1–C4 幂等两跑 + 包边界目标级审计 + precise_scan + self_contain 四口径一致）；库内相对路径可接受。
- 四指标独立复跑全门 0（C1 自包含 / C2 置信度 / C3 Fact Index / C4 标签）；factcheck 锚定缺口 0。
- 结构调整：审查报告自发布包移出至仓库根 `reports/`（`V5独立审查报告-2026-08-20.md`、`发布检查报告-2026-08-20.md`），**不随发布包分发**；发布包仅含 `memoryV5.0.0/`，GitHub Release 仅分发该文件夹。
- 零违例，无需修复，版本保持 v5.0.0。

---

## v4.0.0 (2026-07—08) — 工作库阶段（未对外发布）

- 保留 v3 拓扑的持续建设：新容器/工具/组件页、治理沉淀、Obsidian 审计
- 作为 v5.0.0 的数据源工作库（466 文件），未单独打版发布

---

## v3.0.0 (2026-07-01)

### 内容规模化扩张

v3.0.0 是在成熟 SOUL+Memory 架构上进行的内容填充和质量治理

**核心新增内容**：
- 🎨 **Poiyomi Shaders 完整入库**：8 主题知识文档 + 65 篇原始参考文档（10.0 版本），覆盖五大变体、AudioLink 集成、Modular Shader System、9 种 Lighting Type、Quest 优化策略
- 👤 **玩家操作知识库导入**：从 VRChat 官方 user-guide 导入 6 篇新建文档（Trust Rank、Safety System、Skeletal Input、标准手姿、Expression Menu、Launch Options）+ 2 篇增量修改
- 🔧 **VCC/ALCOM 系统化**：VCC 架构/工作流/VPM 格式 + ALCOM 开源替代品（Rust+Tauri, MIT）的完整参考文档
- 📡 **OSC 协议重大更新**：从 23KB 扩展到 32KB (+40%)，修正 P0 Chatbox 3 参数签名冲突，新增 10+ 核心补充、6 行为细节、8 附录含资源分类索引
- 📦 **VPM 镜像站完整数据**：vcc.vrczh.org 全量 57 个 VPM 仓库元数据，含 8 大核心包详细分析 + 镜像/上游对比
- 🌍 **Community Labs / Companions / Steam Audio**：World 发布流程（Stage 2）、Items vs Props 分裂、2025.4.2+ 音频系统迁移
- 📹 **VRCCameraSettings / VRCQualitySettings**：屏幕/手持相机 + VR 双眼模式、阴影距离覆盖 + 质量只读

**规模变化**：
- 文件从 313 增至 351（+38），总规模从 2.8MB 增至 3.26MB
- World 域：149→154 篇 | Avatar 域：51→75 篇（+47%，Shader 子域大幅扩展）
- Sources 域：14→25 篇（VPM 镜像站数据）| Hybrid 域：4→6 篇（VCC/ALCOM）

### 质量治理闭环（A18/A19/A20）

**A18 — V3.1 正式发布**：
- 数据基线修正：351/351 验证通过（从 92% 提升至 100%）
- 26 个死链清零
- 27 个 frontmatter 失败修复
- 8 个并行 Agent 加速比 2.9x

**A19 — 全面审计**：
- T1 搜索可达性：981 个测试问题，整体可达率 99.6%
- T2 框架合理性：识别 212 个 aliases 不足、123 个 related 缺失
- T3 元文件审查：修复 4 项 P0 问题（SOUL last_review、路径错误等）
- SOUL.md 非人格内容移除，数据基线声明准确

**A20 — 搜索可达性实测**：
- 12 个并行 Agent 覆盖 346/351 文档 (98.6%)
- 1 处路径笔误修复，3 处迁移断链标记
- 确认知识库无内容缺失

### 知识库治理指标

| 指标 | v2.0.0 | v3.0.0 |
|------|--------|--------|
| YAML frontmatter 覆盖率 | ~95% | **100%** |
| 死链数 | ~26 | **0** |
| 搜索可达率 | 未测试 | **99.6%** |
| 文档数 | 313 | **351** |
| 总大小 | ~2.8 MB | **~3.26 MB** |

### 总结

v3.0.0 是一次**内容规模化 + 质量工程化**的双重升级。在 v2.0.0 建立的 Agent 身份 + 元数据标准化架构上，完成了 Avatar Shader 领域（Poiyomi 8 主题）、VPM 生态（57 仓库）、OSC 协议（+40%）等关键内容的大规模填充，同时通过 A18-A20 三个治理周期建立了完整的知识库质量保障闭环。

---

## v2.0.0 (2026-06-21)

### 架构升级：新增 Agent 身份系统

根目录新增 `SOUL.md`（v5.0-slim），定义 Agent 为"知识驱动型 VRChat 技术架构师"，包含 6 种工作模式自动选择、6 域路由表和工程人格设定。

### 元数据标准化

所有核心文件新增 YAML frontmatter（title、category、tags、confidence 等），实现可机器解析的知识图谱层。

### 知识库治理体系建立

新增 `meta/` 目录，含知识写入协议、工作模式详细定义、辅助脚本清单三项核心治理文件，从 SOUL.md 迁移瘦身而来。`_curator_tools/` 为未来策展工具预留。

### 内容增量

- 文件从 295 增至 313（+18），总规模从 2.5MB 增至 2.8MB
- 新增 VRCTween 官方补间文档
- 新增 Udon Persistence 实战子目录（8 文档）
- Data Containers 子路径重构
- LuraSwitch2 更新至 v1.06（VN3 License）
- VRCGraphics 重构纳入 udon 子目录

### 总结

v2.0.0 是一次架构性升级，核心变化是 Agent 身份定义 + 元数据标准化 + 治理体系建立，为知识库的可持续维护奠定基础。

---

## v1.0.0 (初始版本)

VRChat 技术知识库初始版本，包含 295 个文件，覆盖 World、Avatar、Hybrid、Platform、VRChatSDK、Sources 六大领域。
