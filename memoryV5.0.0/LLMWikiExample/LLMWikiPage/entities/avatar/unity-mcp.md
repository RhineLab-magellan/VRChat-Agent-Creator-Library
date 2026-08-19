---
title: "Unity MCP (MCP for Unity) — AI 助手与 Unity Editor 的 MCP 桥接层"
category: avatar
knowledge_level: applied
status: active
source: "github.com/CoplayDev/unity-mcp + coplaydev.github.io/unity-mcp"
source_type: community
version: 1.0
last_review: 2026-08-19
confidence: medium
tags:
  - avatar
  - unity-editor
  - mcp
  - ai-assistant
  - toolchain
aliases:
  - "Unity MCP (MCP for Unity)"
  - unity-mcp.md
related:
  - avatar-audit-methodology.md
  - ndmf-tools.md
  - modular-avatar.md
  - avatar-size-limits.md
  - playable-layers.md
  - expression-menu.md
type: entity
created: 2026-08-19
sources: "github.com/CoplayDev/unity-mcp + coplaydev.github.io/unity-mcp + PyPI mcpforunityserver"
updated: 2026-08-19
---
# Unity MCP (MCP for Unity) — AI 助手与 Unity Editor 的 MCP 桥接层

> 来源: 官方文档站 https://coplaydev.github.io/unity-mcp/ + GitHub https://github.com/CoplayDev/unity-mcp（核实日期 2026-08-19）
> 置信度: medium（官方仓库与文档已核实；部分工具级细节为推断）
> 性质声明: 官方明确 **not affiliated with Unity Technologies**，由 Aura 赞助维护，MIT 许可
> 当前版本: v10.0.0（2026-06-30）

---

## 1. 工具定位

**Unity MCP**（仓库名 MCP for Unity）通过 **Model Context Protocol (MCP)** 把 AI 助手（Claude / Codex / Cursor 等）桥接到 Unity Editor，让助手能够读取并操纵编辑器状态：管理资产、控制场景、编辑脚本、运行测试、做 Profile 与出构建（build）。`[FACT]`

| 维度 | 说明 |
|------|------|
| **设计目标** | 让 LLM 通过统一协议读写 Unity Editor 状态，而非人工复制粘贴 |
| **核心能力域** | 资产管理 / 场景控制 / 脚本编辑 / 运行测试 / Profile / Build |
| **非目标** | 不是 VRChat 专用；不提供运行时、VR、多人或上传证据（见 §6 证据边界） |
| **许可与归属** | MIT，Aura 赞助维护，**与 Unity Technologies 无隶属关系** `[FACT]` |

> 💡 对本 VRChat 知识库而言，Unity MCP 是「Editor 级证据」的**唯一授权通道**：任何 `UNITY_RESOLVED` 层级的查询/变更都必须经由它完成。参见 [[avatar-audit-methodology.md]] 的证据分层纪律。

---

## 2. 架构与工作机制（三层）

`[FACT]` 架构为典型三层桥接：

```
┌─────────────────────┐     MCP     ┌──────────────────────────┐   WebSocket   ┌────────────────────────┐
│  MCP 客户端          │ ──────────▶ │  Python 服务端            │ ────────────▶ │  Unity Editor 内 C# 插件  │
│  Claude / Codex /   │   (stdio /  │  FastMCP + WebSocket hub │  (路由 + 会话) │  MCPForUnity            │
│  Cursor / 本地 LLM  │   HTTP)     │  路由 + 会话隔离          │   隔离        │  主线程执行 Editor API  │
└─────────────────────┘             └──────────────────────────┘               └────────────────────────┘
```

| 层 | 角色 | 说明 |
|----|------|------|
| **MCP 客户端** | 入口 | Claude Desktop / Codex / Cursor / 本地 LLM，发出工具调用 |
| **Python 服务端** | 路由 + 会话隔离 | FastMCP + WebSocket hub，负责把请求路由到正确的 Unity 实例并隔离会话 |
| **Unity C# 插件** | 执行端 | **MCPForUnity**，在 Unity **主线程**执行 Editor API（场景/资产/构建等） |

关键机制 `[FACT]`：
- **多实例路由**：可同时连接多个 Unity 实例，按实例路由请求；
- **per-session 会话隔离**：不同客户端会话互不串扰；
- **主线程执行**：Editor 操作统一回到 Unity 主线程，规避线程安全问题的误报。

> ⚠️ «单实例唯一所有权» 是 **playbook/项目纪律**，不是该工具官方的独占锁——官方明确支持多实例路由 + 会话隔离。二者要区分开看（见 §6）。

---

## 3. 关键组件与能力清单

`[FACT]` 工具面：约 **48 tools / 10 groups**（官方口径注记：README/guide/migration **文本**表述为 47，官方**自动生成** `/reference/tools` 注册表页实际列 48，本文以注册表实数为准），且**按需可见**——如 vfx / animation / ui / testing / probuilder 等组按会话启用，而不是一次性全量暴露。

能力覆盖 `[FACT]`：
- 场景与 GameObject 操作；
- 脚本/C# 代码编辑；
- 资产与 Prefab 管理；
- 材质（Material）与纹理；
- 运行测试（testing group）；
- Profile 与 Build（编译/构建产物）。

**官方工具清单（已核实 HTTP 200，检索日期 2026-08-19）**：`https://coplaydev.github.io/unity-mcp/reference/tools` 为自动生成页（源自 Python 工具注册表 `Server/src/services/tools/`），完整列出 **10 组 / 48 工具**（官方文本口径见 §3 注记：guide/migration 称 47，注册表实数为 48）：

| 分组 | 工具数 | 工具（core 为默认常开，其余按会话按需可见） |
|------|-------|------|
| **`core`** | 30 | **默认常开**：`manage_gameobject` `manage_scene` `manage_asset` `manage_build` `manage_camera` `manage_material` `manage_packages` `manage_components` `manage_prefabs` `manage_physics` `manage_editor` `manage_graphics` `manage_script` `manage_script_capabilities` `manage_tools` `read_console` `refresh_unity` `find_gameobjects` `find_in_file` `get_sha` `execute_menu_item` `execute_custom_tool` `batch_execute` `create_script` `apply_text_edits` `script_apply_edits` `validate_script` `debug_request_context` `set_active_instance` `delete_script` |
| `asset_gen` | 5 | `generate_audio` `generate_image` `generate_model` `import_model` `import_model_file` |
| `vfx` | 3 | `manage_shader` `manage_texture` `manage_vfx` |
| `docs` | 2 | `unity_docs` `unity_reflect` |
| `scripting_ext` | 2 | `execute_code` `manage_scriptable_object` |
| `testing` | 2 | `get_test_job` `run_tests` |
| `animation` | 1 | `manage_animation` |
| `probuilder` | 1 | `manage_probuilder` |
| `profiling` | 1 | `manage_profiler` |
| `ui` | 1 | `manage_ui` |

> 💡 `manage_tools` 可在会话内切换哪些工具组可见——与 §3 顶部「按需可见」机制一致；VRChat Avatar 检查场景下最常用 `core`（场景/层级/资产/Console/构建），配合 `profiling`/`testing` 按需启用。

---

## 4. 版本与仓库迁移

> ⚠️ **关键更正（务必更新旧记录）**：原仓库 `justinpbarnett/unity-mcp` 已 **301 迁移**到 **CoplayDev/unity-mcp**。`[FACT]`

| 项 | 值（核实日期 2026-08-19） |
|----|----|
| **文档站** | https://coplaydev.github.io/unity-mcp/ `[FACT]` |
| **Git 仓库** | https://github.com/CoplayDev/unity-mcp.git `[FACT]` |
| **当前版本** | **v10.0.0**（2026-06-30）`[FACT]` |
| **服务端（PyPI）** | `mcpforunityserver`（pip/uv 安装）`[FACT]` |

> 💡 若旧资料仍然指向 `justinpbarnett/unity-mcp`，应视为过期路由并改用新仓库。

---

## 5. 安装与对接

### 5.1 安装 Unity 插件（注意：是 UPM，不是 VCC/VPM）

> ⚠️ 与大多数 VRChat 工具不同，Unity MCP **不走 VCC/VPM**，而是 **Unity UPM 包**。`[FACT]`

| 方式 | 渠道 | 说明 |
|------|------|------|
| ① **Git URL（UPM）** | `https://github.com/CoplayDev/unity-mcp.git?path=/MCPForUnity#main` | 直接把 MCPForUnity 作为 UPM 包加入 |
| ② **Unity Asset Store** | id **329908** | 商店页面安装 |
| ③ **OpenUPM** | `openupm add com.coplaydev.unity-mcp` | OpenUPM CLI 安装 |

**前置条件** `[FACT]`：
- Unity **2021.3 LTS 及以上**；
- Python **3.10+**（推荐用 `uv` 管理运行环境）；
- 一个 **MCP 客户端**（Claude Desktop / Codex / Cursor / VSCode 等）；
- 服务端从 PyPI 安装 `mcpforunityserver`。

> 💡 建议用 `uvx` 直接跑服务端，避免污染全局 Python 环境（见 §5.3 stdio 一行命令）。

### 5.2 对接传输方式

| 传输 | 用途 | 说明 |
|------|------|------|
| **HTTP** | 默认 `http://localhost:8080/mcp` | 供 **Cursor / Windsurf / VSCode / Cline** 等使用 `[FACT]` |
| **stdio** | Claude Desktop | `uvx --from mcpforunityserver mcp-for-unity --transport stdio` `[FACT]` |

**自动配置**：进入 `Window → MCP for Unity → Configure All Detected Clients` 可自动检测并配置已安装的客户端 `[FACT]`。

### 5.3 Claude Desktop（stdio）最小示例

```text
# mcp 客户端配置（示意）
mcp-for-unity:
  command: uvx
  args: ["--from", "mcpforunityserver", "mcp-for-unity", "--transport", "stdio"]
```

---

## 6. 限制、安全与证据边界 ⚠️

### 6.1 安全面

- 默认**无遥测、无账号** `[FACT]`；
- 支持**远程托管 + API key** 方式（把服务端部署到远程/共享环境时用 API key 鉴权）`[FACT]`；
- 部署面自备安全：暴露的是**本地/远程编辑能力**，应只对可信网络开放。

### 6.2 关键证据边界（方法论归因）`[INFERENCE]` / 项目纪律

这是本条目对 [[avatar-audit-methodology.md]] 最有用的部分——**Unity MCP 暴露的是 Editor 状态，不是最终用户可感知结果**：

| 误区 | 正确认知 |
|------|----------|
| 「Console 干净 = 通过」 | ❌ 干净 Console **≠ 通过**。干净 Console 或静态解析成功都不是 `UNITY_RESOLVED` 证据 |
| 「MCP 可查场景 = 能证明运行时表现」 | ❌ MCP 看到的只是 **Editor 状态**（`UNITY_RESOLVED`），**不等于** client 运行时 / VR / 多人 / 上传证据 |
| 「沿用旧 instance = 可靠连接」 | ❌ 每次操作用前**重新发现并证实 instance**；不沿用旧连接、不以 recency 选实例 |
| 「单实例唯一所有权」 | ⚠️ 这是**项目纪律而非工具官方独占锁**（官方支持多实例路由+会话隔离） |

> 💡 结论分层地讲：能用 MCP 证明到哪一层就只声称到哪一层（详见 [[avatar-audit-methodology.md]] 的 7 级证据阶梯）。

### 6.3 【遗留未知项】

| # | 未知项 | 说明 |
|---|--------|------|
| ① | ~~逐工具清单~~ | ✅ 已排除：官方 `/unity-mcp/reference/tools` 页实测 HTTP 200，48 tools / 10 groups 全部列证（见 §3；官方文本口径 47 / 注册表实数 48） |
| ② | Play Mode 显式授权 | "进入 Play Mode / 执行变更需显式授权" 为 playbook/项目纪律，**非工具官方原文**，官方归属未在文档确认 |

> 【UNKNOWN】表内第 ② 项（playbook 纪律的工具官方归属）核实前**不得当作工具官方特性引用**。

---

## 7. 相关文档

- [[avatar-audit-methodology.md]] — Avatar 检查/审计纪律入口（Unity MCP 是 Editor 级证据的授权通道）
- [[ndmf-tools.md]] — NDMF 工具生态（构建层证据）
- [[modular-avatar.md]] — 生成层输出（NDMF_BUILT 证据）
- [[avatar-size-limits.md]] — 体积指标边界（SDK 构建层证据）
- [[playable-layers.md]] — Animator 层级体系（Editor/构建层交叉）
- [[expression-menu.md]] — 表情菜单（菜单→参数→提供者链路）

> 💡 使用顺序建议：先读 [[avatar-audit-methodology.md]] 确认证据分层与授权边界，再决定本工具能/不能用于哪种结论。
