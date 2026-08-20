# 贡献指南 (Contributing)

感谢你愿意为 **VRChat-Agent-Open-Memory-Library** 贡献知识。本库是 **AI Agent 优先 + 人类创作者共用** 的开源 VRChat 技术知识体系——你贡献的资料经审查合入后，既会被人类读者使用，也会成为 AI Agent 的权威知识源。

## 你可以贡献什么

| 类型 | 说明 | 范例 |
|---|---|---|
| 📄 原始知识文件 | 未经加工的资料 | 官方文档、个人笔记、QA 记录、工程资料 |
| 🎯 可信度标注 | 区分「事实」与「推断」 | 按 `high / medium / low` 分级并注明理由 |
| 🗂️ YAML 元数据 | 遵循库内 frontmatter 规范 | `title / type / category / tags / confidence` 等 |
| 🔗 交叉引用 | 补建页面间关联 | 用 `[[wikilink]]` 建立双向链接 |
| 🧹 格式修正 | 统一结构与元数据 | 对齐现有页面的 Markdown 结构 |
| 🔬 审查验证 | 内容正确性核验 | 工程类改动确认在最新 SDK 可运行 |

## 提交流程

1. **打包资料**：在仓库根的 `Training data/` 目录打包上传原始资料，或加入 QQ 群 `902222352` 提交。
2. **说明来源**：注明知识的可靠来源（官方文档 / SDK 源码 / 社区讨论 / 个人实践），以及你的审查结论。
3. **二次审查**：资料采用后由 **维护者（Master）二次审查 + 模型自审**，共同确定置信度。
4. **合入知识库**：审查通过并标注置信度后，纳入 `LLMWikiPage/` 对应领域目录。

## 知识页质量标准

新知识页 / 对现有页面的改动，应满足以下硬性要求：

- **frontmatter 完整且合法**：遵循 `meta/kb-protocol.md` 中的规范；`tags` 必须在标签分类法内（约 150 个已注册标签，未注册者回退到语义相近的既有标签）。
- **置信度诚实**：每页标注 `confidence`；`low` 级内容**必须**带来源与理由，不得以推断冒充事实。
- **正文标注纪律**：关键事实按 `[FACT]`（已验证）/ `[INFERENCE]`（推断）/ `[UNKNOWN]`（未确认）区分。
- **链接合规**：用 `[[wikilink]]` 与相对链接指向库内页面；外部结论引用 `http(s)://` URL；**严禁硬编码本地路径**（示例路径一律参数化）。
- **自包含**：每个知识文件独立可读，不依赖库外文件才能理解；跨页引用通过链接而非粘贴大段副本。
- **术语规范**：首次出现的内部概念给出简短解释（如 LLMWikiPage、frontmatter、wikilink、confidence）。

## 审查与合入规则

- 全部内容遵循库内治理协议 `meta/` 与 SCHEMA 规范（frontmatter / 标签分类法 / 置信度 / 链接规范）。
- 发布前会运行自动化就绪检查（四指标 C1–C4：自包含 / 置信度 / Fact Index / 标签），未通过不发布。
- 历史版本的增删改统一记录在 [CHANGELOG.md](CHANGELOG.md)（Keep a Changelog 格式），README 只预告最新版本。

## 快速查表

想要为某领域补充知识？先读对应域的入口页：

| 领域 | 入口 |
|---|---|
| World 开发 | `entities/world/udon/index.md` |
| Avatar / Shader | `entities/avatar/shader/index.md` |
| 设计模式与硬约束 | `concepts/patterns/`、`concepts/rules/` |
| OSC / AudioLink / VCC | `concepts/hybrid/` |
| 治理协议 | `meta/kb-protocol.md` |

有任何疑问，加入 QQ 群 `902222352` 沟通，或直接提交 Issue / Pull Request。
