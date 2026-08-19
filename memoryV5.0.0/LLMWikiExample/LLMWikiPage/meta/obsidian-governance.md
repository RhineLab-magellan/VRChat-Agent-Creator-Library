---
title: Obsidian Governance - Memory Vault 分层治理规则
category: meta
knowledge_level: governance
status: active
source: 本地知识库治理
source_type: local
version: 1.0
last_review: 2026-07-14
confidence: High
tags:
  - meta
  - obsidian
  - governance
  - knowledge-base
aliases:
  - Obsidian 分层治理规则
  - Memory Vault Governance
  - Obsidian Governance
related:
  - ../[[../index]]
  - "[[FACT]]"
  - "[[kb-protocol]]"
  - "[[auxiliary-scripts]]"
  # 2026-08-18：_curator_tools/ 已归档至 辅助文件/报告/curator-tools/（不入 Git），原 wikilink 移除
type: governance
created: 2026-07-14
sources: 本地知识库治理
updated: 2026-07-14
---
# Obsidian Governance - Memory Vault 分层治理规则

> 本文件定义 `memory/` Obsidian Vault 的治理边界。目标是让核心知识层完整 Obsidian 化，同时避免工具、备份、历史报告污染知识图谱或被误判为当前事实。
> 2026-08-18 归档整理：``辅助文件``（治理脚本/报告/备份）已整体移入 `辅助文件/`（脚本→`辅助文件/脚本/`，报告→`辅助文件/报告/curator-tools/`，备份→`辅助文件/备份/`），物理保留、不入 Git；下述 `_curator_tools/` 路径在文中作为治理规则描述保留，实际位置以归档后为准。

## Vault Root

```text
C:\Users\59559\Desktop\Agent\UdonSharpAgent\memory
```

`memory/` 是本工作区的 Obsidian Vault Root。Obsidian 写入、索引、链接治理默认限定在该目录内。

## 治理层级

### A. 核心知识层（完整 Obsidian 化）

范围：

```text
api/
avatar/
world/
platform/
hybrid/
patterns/
rules/
vrchatsdk/
misc/
meta/
FACT.md
_always-load.md
index.md
```

治理要求：

- 需要 YAML frontmatter / Obsidian Properties。
- 需要稳定的 `tags`、`aliases`、`related`。
- 应由 `index.md` 或局部 `index.md` 可达。
- 应逐步使用 Obsidian wikilink 建立可点击关系。
- 参与死链、孤立文档、入口可达性、Properties 可解析性审查。

### B. 参考 / 来源层（半 Obsidian 化）

范围：

```text
sources/
references/
references/notes/
reviews/
```

治理要求：

- 需要保留来源、可信度、适用范围和晋升状态。
- 需要父级入口或专题入口。
- 不强制高密度互链。
- 不应直接等同核心事实层。
- 参考资料晋升为核心知识前，应经过验证、拆解和风险标注。

### C. 治理报告层（证据归档，不进入主知识网络）

范围：

```text
_curator_tools/*.md
历史审计报告
验证报告
死链报告
修复报告
```

治理要求：

- 可作为审计证据保留。
- 不要求与核心知识笔记相同的 `aliases` / `related` 完整度。
- 不应作为当前知识事实来源。
- 不参与主知识图谱的质量评分。
- 稳定的人类可读结论未来可迁入 `reviews/`，但迁移前不得删除原始证据。

### D. 工具 / 机器产物层（不 Obsidian 化）

范围：

```text
_curator_tools/*.py
_curator_tools/**/*.py
_curator_tools/**/*.json
JOURNAL.jsonl
*.jsonl
```

治理要求：

- 按普通工程文件管理。
- 不参与 Obsidian 笔记质量统计。
- 不参与 graph / backlink / related 完整性审查。
- 不作为当前知识事实来源。

### E. 备份 / 恢复层（只读排除）

范围：

```text
.backup/
*.bak
_curator_tools/**/*.bak
```

治理要求：

- 只用于完整性比对、回滚和恢复。
- 不进入主索引。
- 不进入主知识图谱。
- 不作为当前事实来源。
- 从备份恢复内容前，必须先进行人工或 Agent 比对。

### F. Obsidian 配置层（默认只读）

范围：

```text
.obsidian/*.json
*.base
*.canvas
```

治理要求：

- `.obsidian/*.json` 默认只读。
- 未经明确授权，不修改 workspace、graph、appearance、app、core plugin 配置。
- `.base` / `.canvas` 属于视图层对象，创建或修改前必须有明确用途和备份。

## 改造前基线要求

任何批量 Obsidian 化改造前，必须生成或更新基线：

- 文件清单
- SHA256
- 文件大小
- 修改时间
- 所属治理层
- Markdown frontmatter 状态
- 标题数
- 表格数
- 代码块数
- Markdown link 数
- Wikilink 数

当前阶段 A 基线入口：

```text
_curator_tools/obsidian_upgrade_manifest_2026-07-14.json
_curator_tools/obsidian_upgrade_baseline_2026-07-14.md
```

## 改造后验收要求

后续每一阶段完成后，至少检查：

- 核心知识层无非预期 missing / added。
- `.obsidian/*.json` 未被非预期修改。
- `.backup/`、`*.bak` 未被误改。
- 正文长度、标题数、表格数、代码块数无异常下降。
- YAML frontmatter 可解析。
- 死链不增加。
- Wikilink / backlink / outgoing link 的可用性提高。
- 参考层没有被误晋升为核心事实。

## 禁止事项

- 禁止把整个 `memory/` 无差别套用同一套笔记质量规则。
- 禁止将 `_curator_tools/`、`.backup/`、`.obsidian/` 纳入普通知识图谱治理。
- 禁止无基线执行批量改名、批量删除、批量链接重写。
- 禁止把旧报告、备份文件或机器日志作为当前知识事实。
- 禁止把所有路径无脑替换为裸 `［［index］］` 等可能歧义的 wikilink。

## Properties 字段语义规范

### `related`

`related` 只表示 **Vault 内部笔记关系**，应逐步使用 Obsidian 路径型 Wikilink：

```yaml
related:
  - "[[../entities/world/index.md]]"
  - "[[../entities/api/networking.md]]"
```

规则：

- 只放当前 `memory/` Vault 内真实存在的 Markdown 笔记。
- 跨目录关系必须使用路径型 Wikilink，不使用裸 `［［index］］`、`［［overview］］`、`［［installation］］`。
- 同目录短链接可在试点阶段保留，但新增关系优先写成 Vault-root 相对路径。
- `related` 不承载来源路径、外部 URL、历史参考目录或工作区外文件。

### `source_path` / `external_reference` / `legacy_source`

当原 `related` 中出现以下内容时，不应直接转为 wikilink：

- Vault 外路径，例如 `../../../（见源库 raw/ 目录）参考文献`。
- 官方文档 root-absolute path，例如 `/worlds/udon/string-loading`。
- 历史工作区路径或已迁移目录。
- 不确定是否存在的规划文件。

建议字段：

```yaml
source_path:
  - "../../../（见源库 raw/ 目录）参考文献"
external_reference:
  - "https://creators.vrchat.com/worlds/udon/string-loading/"
legacy_source:
  - "旧路径，仅作追溯"
```

治理要求：来源字段可追溯，但不参与 Obsidian 主知识关系图谱评分。

## 推荐 Wikilink 规则

由于 Vault 内存在大量同名文件，例如 `index.md`、`overview.md`、`installation.md`，后续 Obsidian 化应优先使用路径型 Wikilink：

```md
[[../entities/avatar/index.md]]
[[../entities/world/udon/index.md]]
[[../entities/api/networking.md]]
```

避免裸链接（以下使用全角括号示意，防止被 Obsidian 识别为真实链接）：

```text
［［index］］
［［overview］］
［［installation］］
```

## 当前状态摘要（2026-07-14，E+F 前全库复扫）

> 统计为阶段 D 后、阶段 E+F 前的只读复扫结果。后续精确计数以 `_curator_tools/` 中最新阶段报告为准，避免本治理文件长期硬编码漂移。

- 总文件：465
- Markdown 文件：441
- 有完整 frontmatter：435
- 无有效 frontmatter：6，均位于 `_curator_tools/` 历史治理报告层
- malformed frontmatter：0
- Wikilink 总数：约 380
- Markdown `.md` 链接总数：636
- missing `.md` link candidate：4，均已分类为 Vault 外来源、治理报告历史链接或报告 diff 示例误报
- root-absolute 官方路径：44，作为 `external_reference` / source 映射治理 backlog，不批量替换
- `.base` / `.canvas`：未发现实际文件
- `.obsidian` 配置文件：5 个，默认只读

## 后续阶段入口

当前 A-D 阶段已完成：

- Phase A：安全基线与完整性复核
- Phase B：低风险导航骨架补齐
- Phase C：明确死链与错误相对路径修复
- Phase D：Properties / `related` 字段试点

后续优先维护：

- Phase E：alias 与 Properties 轻治理
- Phase F：治理状态摘要刷新
- Phase G：治理报告层误报降噪
- Phase H：`source_path` / `external_reference` 迁移试点
- Phase I：导航可达性补强
- Phase J：`related` 字段分批 Obsidian 化
- Phase K：内容缺口建设
