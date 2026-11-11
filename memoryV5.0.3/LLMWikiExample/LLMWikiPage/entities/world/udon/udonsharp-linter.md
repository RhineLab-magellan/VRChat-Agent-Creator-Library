---
title: "UdonSharp Linter"
category: world
knowledge_level: applied
status: active
source: dchaudhari7177/vrchat-udonsharp-linter
source_type: community
version: 1.0
last_review: 2026-08-13
confidence: Medium
tags:
  - world
  - udonsharp
  - debugging
  - toolchain
  - community
  - troubleshooting
aliases:
  - UdonSharpLinterCLI
  - UdonSharp 静态分析器
related:
  - "index.md"
  - "../../../concepts/rules/udonsharp-language-limits.md"
  - "../../api/not-exposed.md"
  - "../../../concepts/rules/runtime-texture-compression.md"
type: entity
created: 2026-08-13
sources:
  - https://github.com/dchaudhari7177/vrchat-udonsharp-linter
updated: 2026-08-13
---
# UdonSharp Linter

## 1. 定位

**[FACT]** UdonSharp Linter 是社区维护的 CLI 静态分析器，用于在 Unity 编译前检查 VRChat UdonSharp 脚本中常见的语言、API 和跨文件语义限制。

来源：[FACT] [UdonSharpLinter README](https://github.com/dchaudhari7177/vrchat-udonsharp-linter)

- 安装方式：`.NET 6.0+` global tool，包名 `tktco.UdonSharpLinter`。
- 基本命令：`udonsharp-lint Assets`。
- 输出格式兼容多数 IDE/CI 的问题面板，例如 `path(line,column): error UDON###: message`。
- 许可证：MIT。

## 2. 检查范围

### 语言特性

可检查 Async/Await、collection initializer、constructor、generic class/method、local function、multidimensional array、nested type、null conditional operator、object initializer、static field、throw、try/catch 等 UdonSharp 常见限制。

### API 与属性

可检查未暴露 API、interface、method overload、network-callable method、SendCustomEvent 方法、TextMeshPro API 等限制。

### 跨文件与语义

可检查跨文件字段访问、跨文件方法调用、引用类型 static field、static method field access、可序列化 Udon Behaviour 类使用等问题。

## 3. IDE 与 CI 集成

- README 提供 Visual Studio Code `tasks.json` 示例，可把 lint 输出接入 Problems 面板。
- README 提供 GitHub Actions 安装与运行示例，适合在提交或 Pull Request 阶段执行。
- `--exclude-test-scripts` 可排除 TestScripts/Tests/Test 目录。

## 4. 使用边界

- **[FACT]** 该工具是社区静态分析器，不是 VRChat 官方 SDK/UdonSharp 编译器。
- **[INFERENCE]** 它适合作为 CI 的快速前置检查，减少 Unity 编译后才暴露的常见限制，但不能替代 Unity、SDK、UdonSharp 和实际 Build & Test。
- 检查规则可能随 SDK/UdonSharp 版本变化；升级工具后应以实际项目编译结果复核。
- 搜索结果显示项目在 2026-07-20 有更新，但仓库规模和采用度有限，因此本页保持 `confidence: Medium`。

## 5. 与本地规则页的关系

本页是工具实体入口；`../../../concepts/rules/udonsharp-language-limits.md` 描述规则本身，`../../../entities/api/not-exposed.md` 描述 API 暴露边界。三者不能互相替代。

## 相关页面

[[index.md]] · [[../../../concepts/rules/udonsharp-language-limits.md]] · [[../../../entities/api/not-exposed.md]] · [[../../../concepts/rules/runtime-texture-compression.md]]