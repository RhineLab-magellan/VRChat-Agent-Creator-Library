# UdonSharpAgent · LLMWikiPage 知识发布包（2026-08-20 就绪版）

## 包内容
- `LLMWikiExample/LLMWikiPage/` —— 414 个 Markdown 知识页（完全自包含：内链为 `[[相对wikilink]]`
  或相对路径 markdown 链接 + 完整外链 URL；不依赖除 URL 以外的任何本地路径）
- `发布检查报告-2026-08-20.md` —— 四指标就绪检查正式报告（C1 自包含 / C2 置信度 / C3 Fact Index / C4 标签）
- 本说明

## 就绪判定（`publish-readiness-check.py --json` 复跑）
| 指标 | 判据 | 结果 |
|---|---|---|
| C1 自包含 | 死链文件链接 = 0；站点相对 URL = 0；wikilink 死链 = 0 | ✅ 0 / 0 / 0 |
| C2 置信度 | high/medium/low 缺失或非法 = 0（low 2 页已可解释） | ✅ 0 |
| C3 Fact Index | 孤页 = 0；index 死链 = 0（FACT.md v3.3 拓扑已更新） | ✅ 0 / 0 |
| C4 标签 | 分类法 150；分类法外复用（≥2 次）= 0 | ✅ 0 |
检查脚本幂等（SHA-256 两次一致）。新增外链 URL 抽验 HTTP 200（creators.vrchat.com / modular-avatar.nadena.dev）。

## 已知保留项（修复轮后终态，非缺陷）
- 正文路径示例已**参数化**（`<用户目录>\...`、`<安装盘>:\...`），无具体盘符/用户名/工作区路径
- 旧 soft raw 锚点已改写为**公开 URL / 抽象描述**，无包外路径
- 2 页 low 置信（定性参考 / 草稿骨架）按 SCHEMA §9 诚实标注（非缺陷）
