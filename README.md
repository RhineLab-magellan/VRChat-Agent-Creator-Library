# VRChat-Agent-Open-Memory-Library

一套面向 VRChat 开发的中文知识库，以 **Markdown 页 + `[[wikilink]]` 内链**组织，可直接被阅读者翻阅，也可被检索型工具整库读入。

本仓库只有**一个**知识库目录 `memoryV5.0.3/`：知识页全部在里面，外加一份发布说明。没有第二个版本的目录。

## 目录结构

```
memoryV5.0.3/
├── README-发布说明.md          发布说明（页数口径、排除项、复核表）
└── LLMWikiExample/
    └── LLMWikiPage/
        ├── index.md            总入口
        ├── _always-load.md     常驻加载页
        ├── FACT.md             事实速查
        ├── concepts/           hybrid · misc · patterns · rules
        ├── entities/           api · avatar · platform · vrchatsdk · world
        ├── comparisons/        横向对比页
        ├── meta/               库自身的元信息
        └── queries/            查询式入口
```

## 数量

| 项 | 数量 | 数字怎么来的 |
|---|---|---|
| 本包文件合计 | 423 | 由具备 shell 的席位对 `memoryV5.0.3/` 树做递归文件计数，结果逐字引用；本说明的作者没有 shell，不自行推算这个数 |
| 发布说明 | 1 | 位于包根，不计入知识页 |
| 知识页（`*.md`，位于 `LLMWikiPage/` 下） | 422 | 423 − 1，与上一行同一次计数 |

两页第三方工具参考页已按今日核对到的版本重新纳入本包（VRCFury `1.1454.0`，2026-11-11 发布；ndmf `1.14.8`，2026-08-29 发布），路径在 `memoryV5.0.3/LLMWikiExample/LLMWikiPage/entities/avatar/` 下。上表的 422 知识页与 423 文件合计即由此实测，不再按预计值表述。

## 版本与日期

本包版本目录为 `memoryV5.0.3`，发布日 2026-11-11。包内除下列两页以外的每一页，涉及第三方工具版本时，版本与发布日期均在 2026-11-11 核对，此后不再声明任何更新的版本状态；读者若要确认「今天是否仍然如此」，需要自己回查该工具的发布记录。例外是 `memoryV5.0.3/LLMWikiExample/LLMWikiPage/entities/avatar/` 下重新纳入的两页第三方工具参考页（`vrcfury-reference.md`、`ndmf-tools.md`），它们于 2026-11-12 按当天版本核对，核对日期不与 2026-11-11 混同，以那两页自身所载为准。

## 页数口径

包内被写过 414、415、421、422 几个页数，发布说明里把这笔账算到了底：

432 个归档文件 = 422 个发布知识页 + 0 个按裁定不随包发布的页 + 9 个内部策展审计文件 + 1 份发布说明。

加减可以在十秒内重算。发布说明还给出了排除项的逐条理由、发布前在 422 页上重跑的核对表（3378 条 wikilink 全部在包内解析，回退 0 条，未解析 0 条），以及上一轮那处链接缺口已随两页重新纳入而关闭的记录。

## 怎么读

- 想快速进入：`memoryV5.0.3/LLMWikiExample/LLMWikiPage/index.md`。
- 想按主题找：先看 `concepts/`（规律与模式）还是 `entities/`（具体组件与工具）。
- 想横向比较：`comparisons/`。
- 想知道哪些结论有官方出处、哪些只是社区经验：每页正文与置信度标注里写明；发布说明的「两处需要读准的地方」说明了「最新」二字在本库里的含义边界。

## 许可

见仓库根目录 `LICENSE`。
