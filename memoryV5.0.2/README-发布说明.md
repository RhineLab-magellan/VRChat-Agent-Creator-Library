# VRChat-Agent-Open-Memory-Library · LLMWikiPage 知识发布包（v5.0.2 · 2026-11-10）

## 包内容

- `LLMWikiExample/LLMWikiPage/` —— 416 个 Markdown 知识页（完全自包含：内链为 `[[wikilink]]`
  或相对路径 markdown 链接 + 完整外链 URL；不依赖除 URL 以外的任何本地路径）
- 本说明

> 📋 审查/就绪报告存放于仓库根 `reports/`（`V5独立审查报告-2026-08-20.md`、`发布检查报告-2026-08-20.md`），**不随本发布包分发**。

## 相对 v5.0.0 的变更

- **新增** `entities/avatar/menu-display-name-resolution.md`：LI（lilycalInventory 1.5.2）与 MA（Modular Avatar 1.18.7）菜单显示名的来源与回退链（组件级 2 级 / Costume 级 7 级 / MA label 2 级），含 Inspector 绑定关系、Unity 序列化转义、批量改名的数组索引陷阱与安全写入流程。社区贡献 PR #5，结论已按上游源码逐条复核。
- **索引同步**：新页登记进 `entities/avatar/index.md`（MOC / 核心文档表 / 子域表 / frontmatter related）与顶层 `index.md`（Avatar 域路由表 / 文件清单）。
- **修正**：新页 §2.2 回退链「六级」→「七级」；`sources` 字段与 §2.1 正文的 `Runtime/ObjHelper.Component.cs` → `Editor/Helper/ObjHelper.Component.cs`；另将 `comparisons/index.md`、`queries/index.md` 的标签补足到协议下限（2 → 3）。
- **目录版号对齐**：知识根 `memoryV5.0.0/` → `memoryV5.0.2/`；页数口径校正为实际值 414 → 416。

## 就绪判定（本次发布前复算）

> 仓库内的 `publish-readiness-check.py`（v5.0.0 报告所引用）未随仓库入库，本次按同一口径用独立脚本对全库 416 页复算，逐项结果如下。

| 指标 | 判据 | 本次复算 |
|---|---|---|
| C1 自包含 | wikilink 死链 = 0；相对链接死链 = 0；站点相对 URL = 0；绝对本地路径 = 0 | ✅ 0 / 0 / 0 / 0 |
| C2 置信度 | 缺失或非法 = 0 | ✅ 0（High 205 / Medium 209 / Low 2，大小写合并统计） |
| C3 Fact Index | 孤页（既无入链也无出链）= 0；index 死链 = 0 | ✅ 0 / 0（另有 2 页无入链但有出链，见下） |
| C4 标签 | 每页 3–8 个；本批未引入新分类法外标签 | ✅ 0 违例 |

页数分布：`entities/` 353（其中 `avatar/` 101）、`concepts/` 47、`comparisons/` 6、`meta/` 6、`queries/` 1、根 3（`index.md` / `FACT.md` / `_always-load.md`）。包内无二进制资产，全部为 Markdown。

## 已知保留项（非本批缺陷）

- **2 页无入链（tail）**：`entities/world/development/index.md`、`entities/world/udon/networking/patterns.md` —— 均主动引用其他页，符合 C3「孤页」口径（既无入链也无出链才算），自 v5.0.0 起既存。
- **38 页 `confidence` 使用小写 `high` / `medium`**：语义与 `High` / `Medium` 等价，自 v5.0.0 起既存，本批未批量改写（改写会使本次发布 diff 覆盖 38 个无关文件）。
- **248 个标签只出现一次**：C4 口径允许单次使用标签，仅限制「分类法外标签被复用 ≥2 次」。
- **2 页 Low 置信**（定性参考 / 草稿骨架）按 SCHEMA §9 诚实标注。
- 正文路径示例保持**参数化**（`<用户目录>\...`、`<安装盘>:\...`），无具体盘符、用户名或工作区路径。
