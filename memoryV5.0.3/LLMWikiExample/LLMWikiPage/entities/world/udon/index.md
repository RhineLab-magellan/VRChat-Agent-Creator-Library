---
title: "Udon 总览"
category: world
subcategory: udon
knowledge_level: applied
status: active
source: 本地知识库整理
source_type: community
version: 1.0
last_review: 2026-06-20
confidence: Medium
tags:
  - misc
  - index
  - navigation
aliases:
  - "Udon 总览"
related:
  - ai-navigation.md
  - animation-events.md
  - avatar-events.md
  - debugging-udon-projects.md
  - event-execution-order.md
  - image-loading.md
  - udonsharp-linter.md
  - input-events.md
  - persistence/index.md
  - string-loading.md
  - udon-moderation-tool-guidelines.md
type: entity
created: 2026-06-20
sources: 本地知识库整理
updated: 2026-06-20
---
# Udon 总览

> 来源: https://creators.vrchat.com/worlds/udon/
> 抓取日期: 2026-06-15
> 状态: ✅ FACT (官方入口文档)

---

## 概述

**Udon** 是 VRChat 世界的**官方编程语言**,允许玩家与世界物体交互。脚本可与场景物体、玩家、同步网络变量等交互。

> **关键事实**: Udon 在 **VRChat 客户端** 和 **Unity Editor** 中**都能运行**,这意味着你可以**无需 Build & Test 即可在 Editor 中测试和调试脚本**。

---

## Udon 是什么(What is Udon?)

> **底层定义**: **VRChat Udon 是一个虚拟机(VM)**,运行从 **Udon Assembly** 编译的字节码(bytecode)。

技术说明(给高级用户):

- VRChat Udon 是个 VM,跑的是 Udon Assembly 编译出的字节码
- Udon Assembly 可由 **3 种方式**生成:
  1. **VRChat Udon Node Graph** UI(内置可视化编程)
  2. 手动编写 Udon Assembly(极少用)
  3. 写自己的编译器生成 Udon Assembly 或直接生成字节码(技术专家选项)

---

## 3 种创建 Udon 脚本的方式

### 方式 1: Udon Node Graph(可视化)

- **特点**: 节点 + 连线的可视化编程
- **类比**: Unity Animator、Blender Shaders、Geometry Nodes、Unreal Blueprints
- **优势**: VRChat SDK 内置,**不需要第三方工具**
- **适用**: 完全没编程经验,或只做简单脚本
- **文档**: [[entities/world/udon/graph/index]]

### 方式 2: UdonSharp(C# 入口) ⭐ 本 Agent 主战场

- **特点**: 用 C# 编写 Udon 脚本
- **类比**: Unity 内置 C# 脚本系统
- **IDE 推荐**:
  - [Visual Studio](https://visualstudio.microsoft.com/vs/unity-tools/) — 免费
  - [Rider](https://www.jetbrains.com/rider/) — 非商业用途免费
- **适用**: 已熟悉编程,或需要写复杂脚本
- **文档**: [[entities/world/udon/udonsharp/index]](本目录)

### 方式 3: 直接编写 Udon Assembly

- **特点**: 手动写 Udon Assembly 字节码
- **使用频率**: 极少见,通常只在 UdonSharp/Graph 编译器还不够灵活时

---

## Udon 与底层 VM 关系

> **本节是 Udon 性能/能力的事实基础,所有 UdonSharp 行为都受此约束**

### Udon 字节码 = 9 条指令

| Opcode | 值 | 含义 |
|---|---|---|
| `NOP` | 0x0 | No operation,只步进 PC |
| `PUSH` | 0x1 | 将堆地址推入栈 |
| `POP` | 0x2 | 栈顶弹出(空栈 → 程序停止) |
| `JUMP_IF_FALSE` | 0x4 | 出栈 bool,false 时跳转 |
| `JUMP` | 0x5 | 无条件跳转(带耗时检查) |
| `EXTERN` | 0x6 | **外部函数调用(最昂贵指令)** |
| `ANNOTATION` | 0x7 | 注解(可能是调试信息) |
| `JUMP_INDIRECT` | 0x8 | 从堆读取目标地址后跳转 |
| `COPY` | 0x9 | 出栈两次,源→目标堆复制 |

**UdonSharp `int a = b + c;` 编译后 = 至少 4-6 条 VM 指令(每次都是装箱/拆箱)**。

### 整型栈作为 "Extra Parameters"

- Udon 调用外部方法时,栈上的"参数"实际上是**堆地址**,不是值本身
- `delegate.Invoke(_heap, parameterAddresses)` 是 VM 访问数据的核心模式

### 局部变量不存在

- Udon 没有"局部变量"概念,所有变量都在 **Udon Heap** 上
- 递归函数需谨慎,深度过深会触发 **10 秒硬限制**
- 每次函数调用 = 多次 PUSH/POP/COPY + EXTERN 或 JUMP

### `JUMP, 0xFFFFFFFC` = 返回 Udon 代码

- 0xFFFFFFFC 是 Udon VM 的"返回地址"特殊值
- 类似于 native code 中的 `RET` 指令

### `this` 的特殊含义

`this` 在 Udon 中可指代 3 种不同类型,需要明确:

| 上下文 | `this` 实际指向 | 用途 |
|---|---|---|
| 默认 | `UdonBehaviour` | Udon 脚本主类 |
| GameObject 上下文 | `GameObject` | 通过 `GetComponent<UdonBehaviour>(gameObject)` |
| Transform 上下文 | `Transform` | 父子级位置/旋转 |

### Udon 类型命名规则

> **C# 类型在 Udon 字节码中会被重命名**:

- **`.` 移除**: `System.Int32` → `SystemInt32`
- **`+` 移除**(嵌套类): `Outer+Inner` → `OuterInner`
- **`Array` 追加**(数组): `System.Int32[]` → `SystemInt32Array`

**为什么重要**: 当你在 Udon 日志中看到 `SystemInt32Array` 而不是 `int[]` 时,这是正常现象,不是 bug。

> 详细 VM 逆向工程: [[concepts/rules/udon-vm-architecture]]

---

## Udon 核心子页面索引

| 子页面 | 路径 | 核心内容 |
|---|---|---|
| Udon Node Graph | [[entities/world/udon/graph/index]] | 可视化编程 |
| UdonSharp | [[entities/world/udon/udonsharp/index]] | C# 入口 |
| **Player API** | [[entities/world/udon/players/index]] | 玩家交互 |
| **AI Navigation** | [[entities/world/udon/ai-navigation]] | NavMeshAgent + 寻路 |
| **Animation Events** | [[entities/world/udon/animation-events]] | Animator Event 触发的白名单 |
| **Avatar Events** | [[entities/world/udon/avatar-events]] | OnAvatarChanged 玩家切 Avatar |
| Data Containers | [[entities/world/udon/data-containers/index]] | JSON 序列化 + 位/字节操作 ⭐ |
| **Data Tokens** | [[entities/world/udon/data-containers/data-tokens]] | DataToken 类型系统 + 隐式装箱 + 提取值 |
| **Data Lists** | [[entities/world/udon/data-containers/data-lists]] | DataList 完整 API + 嵌套 + 性能基准 |
| **Data Dictionaries** | [[entities/world/udon/data-containers/data-dictionaries]] | DataDictionary 完整 API + 键值对 |
| **VRCJSON** | [[entities/world/udon/data-containers/vrcjson]] | JSON 序列化/反序列化 + 错误处理 |
| **Debugging Udon** | [[entities/world/udon/debugging-udon-projects]] | Debug 模式 + 断点 |
| **Event Execution Order** | [[entities/world/udon/event-execution-order]] | ⭐ 关键:事件触发时序 |
| **External URLs** | [[entities/world/udon/external-urls]] | VRCUrl + 白名单 |
| **Image Loading** | [[entities/world/udon/image-loading]] | VRCImageDownloader + 4KB 限制 |
| **UdonSharp Linter** | [[entities/world/udon/udonsharp-linter]] | 社区 CLI 静态分析、IDE/CI 诊断 |
| **Input Events** | [[entities/world/udon/input-events]] | Interact/Use/Trigger/Collision |
| Midi in Udon | [[entities/world/udon/midi/index]] | MIDI 钢琴输入 |
| Networking | [[entities/world/udon/networking/index]] | 网络同步（含 `architecture.md` / `patterns.md` 补充架构与工程模式） |
| Persistence | [[entities/world/udon/persistence/index]] | PlayerData + PlayerObject(实战教程 8 文档,2026-06-21) |
| **String Loading** | [[entities/world/udon/string-loading]] | VRCStringDownloader |
| UI Events | [[entities/world/udon/ui-events]] | Unity UI 事件白名单 |
| **Video Players** | [[entities/world/udon/video-players/index]] | 视频播放器生态导航（总览 / 轻量卡片） + 深度入口指向 VVMW / 官方示例 |

| VRCGraphics | [[entities/world/udon/vrc-graphics/index]] | VRCGraphics API |
| **VRCTween** | [[entities/world/udon/vrctween/index]] | ⭐ **官方补间系统**(DOTween 封装,7 大类 + 虚拟补间)NEW 2026-06-21 |
| World Debug Views | [[entities/world/udon/world-debug-views]] | 调试视图 |

---

## Bug 报告和功能请求

VRChat 官方使用 [Canny 反馈板](https://vrchat.canny.io/udon) 收集 Udon 的 bug 和功能请求。

---

## 与知识库互补

- **Udon VM 架构逆向**: [[concepts/rules/udon-vm-architecture]] ⭐ 9 指令详细文档
- **UdonSharp 语言限制**: [[concepts/rules/udonsharp-language-limits]] ⭐ C# → Udon 限制
- **Udon 事件完整参考**: [[entities/api/events-reference]] ⭐ 事件方法签名
- **UdonSharp 运行时系统**: [[entities/api/udonsharp-runtime]] ⭐ GetProgramVariable/SetProgramVariable
- **Udon 编译管线**: [[entities/world/udon/udonsharp/compilation]] ⭐ 四阶段编译

---

## ⚠️ 关键设计约束(全 Agent 适用)

> 🔴 **`_onEnable` → `_start` 无间隔执行,后续事件受此约束**
>
> - Udon 脚本中,**`_onEnable()` 紧接 `_start()` 执行,中间没有任何空隙**
> - 在 `_onEnable` 中**绝对不能**假设其他 UdonBehaviour 已 `_start`
> - 所有需要"其他系统就绪"的初始化,必须放在 `_start` 中
> - 完整事件顺序: [[entities/world/udon/event-execution-order]]
