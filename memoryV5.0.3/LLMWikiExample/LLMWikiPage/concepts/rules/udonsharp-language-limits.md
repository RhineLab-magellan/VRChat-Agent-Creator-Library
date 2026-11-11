---
title: "Rule Set: UdonSharp Language Limits"
category: rules
knowledge_level: applied
status: active
source: "UdonSharp 官方文档 + VRChat 社区实践 + VRChat 官方字节码规范 + SDK release note 3.10.4/3.10.5/3.10.6"
source_type: official
version: 1.1
last_review: 2026-11-10
confidence: High
tags:
  - rules
  - rules
  - event
  - udonsharp
  - reflection
aliases:
  - "Rule Set： UdonSharp Language Limits"
  - "Rule Set: UdonSharp Language Limits"
related:
  - multi-vm-rules.md
  - udon-vm-architecture.md
  - udonsharp-deep-pitfalls.md
  - ../../entities/api/not-exposed.md
  - networking-rules.md
type: concept
created: 2026-06-20
sources: "UdonSharp 官方文档 + VRChat 社区实践 + VRChat 官方字节码规范 + SDK release note 3.10.4/3.10.5/3.10.6"
updated: 2026-11-10
---
# Rule Set: UdonSharp Language Limits

> SDK Version: VRChat SDK 3.x（编译器行为核证至 **3.10.6**）
> UdonSharp Version: 1.x
> Last Verified: 2026-11-10
> **字节码级参考**:[[entities/world/udon/vm-and-assembly]] - PUSH/COPY/EXTERN 实现的底层细节,所有 UdonSharp 限制的最终约束来源

---

## RULE-LL-01: UdonSharp Is Not Full C#

### Rule
UdonSharp 使用 C# 语法作为前端表达，但编译目标是 Udon Assembly/Udon Bytecode，运行在 Udon VM 上。不得默认任何完整 C# 特性可用。

### Reason
Udon VM 是一个受限的解释执行环境。UdonSharp 编译器不支持大量 C# 特性。编译后的产物不是 IL，而是 Udon Assembly。

### Impact
- 使用不支持特性会导致编译错误或运行时崩溃
- 部分特性可能"静默失败"——编译通过但行为异常
- 必须逐一验证每个 C# 特性的可用性

---

## RULE-LL-02: Prohibited C# Features

以下 C# 特性在 UdonSharp 中禁止或应极力避免：

### 集合类
- `List<T>` — 使用数组或 `DataList`
- `Dictionary<TKey, TValue>` — 使用 `DataDictionary`
- LINQ 任何形式 — 使用显式 `for` 循环
- 迭代器 / `yield return`

### 异步 / 异常
- `async` / `await` — Udon VM 不支持
- `try` / `catch` / `throw` / `finally` — Udon VM 不支持异常处理
- Coroutine (`IEnumerator`, `StartCoroutine`) — 使用 `SendCustomEventDelayedSeconds`

### 泛型与委托
- 复杂泛型 — 仅支持简单泛型约束
- `Action<T>`, `Func<T>`, delegate 关键字 — 使用 `SendCustomEvent`
- Runtime `AddListener` / `RemoveListener` on UnityEvent
- Lambda 表达式

### 反射与动态
- `System.Reflection` 任何成员
- `dynamic` 关键字
- `GetType()`, `typeof()` 反射用法
- `Activator.CreateInstance`

### 多线程
- `System.Threading` 任何成员
- `Task`, `Thread`, `ThreadPool`

### 其他
- `switch` on string types — 用 `if/else` 替代
- `String.Format` — 简单拼接可接受，高频场景避免
- `enum.TryParse` — 不可用

---

## RULE-LL-03: Preferred Patterns

### 集合与迭代
```csharp
// ✅ 正确
GameObject[] targets = new GameObject[16];
for (int i = 0; i < targets.Length; i++) { ... }
DataList list = new DataList();
DataDictionary dict = new DataDictionary();

// ❌ 错误
List<GameObject> targets = new List<GameObject>();
var result = targets.Where(x => x.activeSelf);
```

### 延迟调用
```csharp
// ✅ 正确
SendCustomEventDelayedSeconds(nameof(_DelayedAction), 1.0f);

// ❌ 错误
yield return new WaitForSeconds(1.0f);
```

### 事件通信
```csharp
// ✅ 正确
SendCustomNetworkEvent(VRC.Udon.Common.Interfaces.NetworkEventTarget.All, nameof(PlayEffect));
SendCustomEvent(nameof(OnStateChanged));

// ❌ 错误
myEvent.AddListener(MyHandler);
Action<int> callback = x => { ... };
```

---

## RULE-LL-04: Conditional Compilation

### Rule
使用 `#if UNITY_EDITOR` 或 `#if UDONSHARP` 等条件编译来区分编辑器代码和运行时代码。

### Reason
某些 Unity Editor API 在 Udon 运行时不可用。条件编译可以保留编辑器工具代码而不影响运行时。

---

---

## RULE-LL-05: GetComponent Restrictions

### Rule
`GetComponent<T>()` 泛型形式在 UdonSharp 中不可用（SDK < 3.8）。必须使用 cast 语法。SDK 3.8+ 对 UdonSharpBehaviour 子类可用泛型形式。

### Pattern
```csharp
// ❌ 错误 (SDK < 3.8)
UdonBehaviour ub = GetComponent<UdonBehaviour>();

// ✅ 正确 (所有版本)
UdonBehaviour ub = (UdonBehaviour)GetComponent(typeof(UdonBehaviour));

// ✅ 正确 (SDK 3.8+)
MyScript s = GetComponent<MyScript>();
```

---

## RULE-LL-06: Struct Mutation Requires Assignment

### Rule
结构体方法不修改原始值。必须使用返回值。

### Pattern
```csharp
// ❌ 错误: v 未被修改
v.Normalize();

// ✅ 正确: 使用返回值
v = v.normalized;
```

---

## RULE-LL-07: Field Initializers Are Compile-Time

### Rule
字段初始化器在编译时求值。场景相关引用必须在 `Start()` 中获取。

### Pattern
```csharp
// ✅ OK: 编译时常量
private int maxPlayers = 10;

// ❌ 错误: 运行时值在字段初始化器（所有实例共享同一值）
// private int rng = Random.Range(0, 100);

// ✅ OK: Start() 中初始化
private int rng;
void Start() { rng = Random.Range(0, 100); }
```

---

## RULE-LL-08: UdonSharpProgramAsset Requirement

### Rule
每个 `.cs` UdonSharpBehaviour 需要对应的 `.asset`（UdonSharpProgramAsset）文件。创建新脚本后必须确认 `Assets/Editor/UdonSharpProgramAssetAutoGenerator.cs` 存在，否则需要安装。

### Impact
没有 `.asset` → "The associated script cannot be loaded"，脚本不编译到 Udon。

---

## RULE-LL-09: UdonBehaviour Program Source Wiring

### Rule
UdonBehaviour 组件的 `programSource` 字段必须引用匹配的 UdonSharpProgramAsset。推荐使用 `AddUdonSharpComponent<T>()` 自动处理。

### Pattern
```csharp
#if UNITY_EDITOR && !COMPILER_UDONSHARP
using UdonSharpEditor;
    MyScript script = gameObject.AddUdonSharpComponent<MyScript>();
#endif
```

---

## RULE-LL-10: Recursive Methods Need Attribute

### Rule
递归方法必须标记 `[RecursiveMethod]` 属性。

### Pattern
```csharp
[RecursiveMethod]
private int Factorial(int n) { ... }
```

---

## RULE-LL-11: Unity Callbacks Don't Need Override

### Rule
Unity 回调 (`OnTriggerEnter` 等) 不需要 `override`。只有 VRChat 事件需要 `override`。

### Pattern
```csharp
// ❌ 错误: override → CS0115
public override void OnTriggerEnter(Collider other) { }

// ✅ 正确: 无 override
public void OnTriggerEnter(Collider other) { }

// ✅ 正确: VRChat 事件需要 override
public override void OnPlayerJoined(VRCPlayerApi player) { }
```

---

## RULE-LL-12: Method Overloading Not Supported

### Rule
UdonSharp 不支持方法重载。所有方法名必须唯一。

### Pattern
```csharp
// ❌ 错误
void Do(int x) { }
void Do(string x) { }

// ✅ 正确
void DoInt(int x) { }
void DoString(string x) { }
```

---

## RULE-LL-13: Available But Limited C# Features (SDK 3.7.1+)

### 可用但有约束
| Feature | SDK | Notes |
|---------|-----|-------|
| `StringBuilder` | 3.7.1 | 高效字符串拼接 |
| `Regex` | 3.7.1 | 正则匹配 |
| `System.Random` | 3.7.1 | 确定性随机数 |
| `System.Type` | 3.7.1+ | 运行时类型信息 |
| `GetComponent<T>()` (继承) | 3.8+ | 仅 UdonSharpBehaviour 子类 |

---

## Exceptions
- `DataList` / `DataDictionary` 的性能特性不同于普通 C# 集合，高频读写需实测
- 简单 `string + string` 在低频场景可接受
- `static` 字段在所有实例间共享且不同步，仅用于只读常量

---

## 字节码级参考

所有 UdonSharp 语言限制的**最终原因**是 Udon VM 仅有 9 条指令(PUSH/POP/JUMP/JUMP_IF_FALSE/JUMP_INDIRECT/COPY/EXTERN/ANNOTATION/NOP)且**无局部变量、无异常、无反射**。

如需理解某个限制的底层原因,参考:

- **[[entities/world/udon/vm-and-assembly]]** - 完整 9 Opcodes 说明、Udon Assembly 语法、Extern 签名格式
- **[[concepts/rules/udon-vm-architecture]]** - Udon VM 内部实现(强类型堆、LightweightStack、10 秒硬限制)
- **[[entities/api/udon-type-exposure]]** - 哪些 C# 类型/成员被白名单暴露

> **关键洞察**:本规则集(RULE-LL-01 ~ RULE-LL-13)的每条规则都对应 Udon VM 的一种能力缺失。例如:
> - **RULE-LL-02 禁用 `List<T>`** ↔ Udon VM 无泛型集合运行时
> - **RULE-LL-02 禁用 `async/await`** ↔ 无协程指令
> - **RULE-LL-02 禁用 `try/catch`** ↔ EXTERN 指令无异常处理
> - **RULE-LL-12 不支持重载** ↔ Extern 签名按方法名定位,重载会产生冲突

---

## RULE-LL-14: VRC Graphics API 限制(2026-06-15 新增)

> 详细文档: [[entities/world/udon/vrc-graphics/index]]、`asyncgpureadback.md`、`vrchat-shader-globals.md`

### 14.1 VRCShader 属性名 `_Udon` 前缀限制 🔴

**Rule**: `VRCShader.PropertyToID()` 接受的属性名必须以 `_Udon` 为前缀,或使用字面量 `_AudioTexture`。

**Reason**: VRChat 引擎只将 `_Udon*` 前缀的全局属性路由到 Udon 系统,其他属性会被过滤。

**Impact**:
- 无 `_Udon` 前缀 → `PropertyToID` 仍返回 ID(不报错),但 `SetGlobal*` **静默失败**(不报错但不生效)
- 必须使用 `_Udon` 前缀才会影响世界/Avatar 的所有 Shader

**Pattern**:
```csharp
// ✅ 正确
int id = VRCShader.PropertyToID("_UdonMyColor");
VRCShader.SetGlobalColor(id, Color.red);

// ❌ 错误 - 静默失败
int id = VRCShader.PropertyToID("MyColor");
VRCShader.SetGlobalColor(id, Color.red);  // 不生效但不报错
```

### 14.2 VRCGraphics.Blit Quest GPU 限制 🔴

**Rule**: `VRCGraphics.Blit` 在 Quest GPU 上**不会工作**,除非满足以下条件之一:
1. 在 Shader Pass 中添加 `ZTest Always`
2. 关闭目标 RenderTexture 的 depth(`renderTexture.depth = 0`)

**Impact**: 未处理会导致操作静默失败(不报错但 Blit 不生效)。

### 14.3 VRCAsyncGPUReadback 接口差异

**Rule**: `VRCAsyncGPUReadback` 与 Unity `AsyncGPUReadback` 接口不同:
- 接收 `IUdonEventReceiver` 而非 `Action<...>` 回调
- 通过 `OnAsyncGpuReadbackComplete` 方法接收完成消息
- 使用 `TryGetData(array)` 而非 `GetData<T>()`

**Impact**:
- 回调可能在非主线程,需通过 `SendCustomEvent` 跳转到主线程
- 每次请求有 GPU 同步成本,避免每帧调用
- Texture 必须 `isReadable = true`

**Pattern**:
```csharp
// ✅ 正确
VRCAsyncGPUReadback.Request(texture, 0, (IUdonEventReceiver)this);

public void OnAsyncGpuReadbackComplete(VRCAsyncGPUReadbackRequest request)
{
    if (request.hasError) return;
    _data = new byte[request.layerDataSize];
    request.TryGetData(_data);
    SendCustomEvent(nameof(ProcessOnMainThread));  // 跳主线程
}

public void ProcessOnMainThread()
{
    // 安全访问 Udon API
    myVariable = _data[0];
}
```

### 14.4 VRCShader.SetGlobalInteger 仍按 float 存储

**Rule**: `VRCShader.SetGlobalInteger(int id, int value)` 由于 Unity bug 实际以 `float` 存储值。

**Impact**: 精度损失(超过 2^24 的整数会被截断)。需要精确 int 存储时,使用 `SetGlobalFloat` + 显式转换,或考虑不使用整数类型。

### 14.5 `_VRChat` 前缀是受保护命名空间 🔴

**Rule**: 不要在自定义 Shader 中使用 `_VRChat` 前缀(除了官方文档化的变量),VRChat 可能随时新增未文档化的同名变量。

**Reason**: 保留给 VRChat 引擎注入的全局变量(相机模式、镜子模式、时间等)。

**Impact**: 命名冲突 → 自定义 Shader 行为被覆盖。

**Pattern**:
```hlsl
// ❌ 错误 - 使用 _VRChat 前缀
float4 _VRChatMyCustomVar;  // 可能与未来 VRChat 变量冲突

// ✅ 正确 - 使用 _Udon 前缀
float4 _UdonMyCustomVar;  // 专门用于 Udon SetGlobal
```

### 14.6 详细参考

| 主题 | 文档 |
|------|------|
| VRC Graphics 总览 | [[entities/world/udon/vrc-graphics/index]] |
| VRCAsyncGPUReadback 完整 API | [[entities/world/udon/vrc-graphics/asyncgpureadback]] |
| `_VRChat*` 全局变量清单 | [[entities/world/udon/vrc-graphics/vrchat-shader-globals]] |
| 渲染性能优化 | [[entities/world/performance-guide]] |

---

## RULE-LL-15: 编译器修正不等于语言特性（SDK 3.10.4–3.10.6）

> 来源：https://creators.vrchat.com/releases/release-3-10-4/ 、 https://creators.vrchat.com/releases/release-3-10-5/ 、 https://creators.vrchat.com/releases/release-3-10-6/

这一节记录的是 **UdonSharp 编译器的行为变化**，不是"新增 C# 特性"。每一条都建议当成"重新验证"的触发点。

### 15.1 enum 比较：按枚举值 vs 按整数值（3.10.6 修复）🔴

**Rule**：3.10.6 之前，`enum` 之间的 `>=` / `<=` / `>` / `<` 比较会**按整数值**比较，导致与枚举值语义不一致。

官方例子（release note 原文）：

```csharp
enum Test { A }
Test foo = Test.A, bar = Test.A;
bool result = (foo >= bar);   // 3.10.6 之前返回 false；之后正确返回 true
```

**Impact**：
- 3.10.6 之前的工程里，若写过"绕过 bug"的 `(int)foo >= (int)bar` 强制转换，**升级后行为不变**（仍然正确），可以保留；
- 但**不要**因为"现在对了"就把 `enum` 当序数安全使用——enum 的底层值仍由声明顺序决定，跨版本增删枚举成员会改变比较结果；
- 该修复是**编译器一致性修复**，官方原文措辞是 "Fixed an Udon compiler inconsistency between comparing enums by enum value vs. by integer value"，**没有**说放宽对 enum 的其他限制。

### 15.2 命名参数跳过可选参数（3.10.6 修复）🔴

**Rule**：3.10.6 之前，调用时用命名参数**跳过中间的可选参数**会**错位落参**（参数被静默丢弃 / 位置错乱）。

官方例子（release note 原文）：

```csharp
void MyMethod(int a, int b = 0, int c = 0)
MyMethod(1, c: 3);   // 3.10.6 之前实际编译成 MyMethod(1)，c 被静默丢掉
```

**Impact**：
- 3.10.4 新增的 `VRCBillboard.Register(transform, allowTilt: true, smoothingFactor: 0.25f)` 这类**重度依赖命名参数**的 API，在 3.10.6 之前**不可靠**——见 [[entities/api/vrc-billboard]]；
- 升级后建议把"跳参调用"全量搜一遍重新验证（历史上它是**静默**错误，编译不报错）；
- 反过来，**不要**据此认为任意 C# 9 语法可用——命名参数本身是老特性，这里修的是编译器的落参 bug。

### 15.3 Assembly Version Defines（3.10.5+）

**Rule**：3.10.5 起 U# 脚本可使用 **Assembly Version Defines**：在 Assembly Definition 的 Version Define Expression 里写版本，例如 `3.10`，即可用 `#if SDK_310_OR_NEWER` 在**编译期**门控代码。

- 官方原文："UdonSharp scripts can now use Assembly Version Defines in their code."（release note 3.10.5）
- ⚠️ **U# Assembly Definition 仍然必需**（不是有了 defines 就不要 asmdef）
- ⚠️ 未知的 `VRC_ENABLE_*` scripting define 会被 SDK **strip**（release note 3.10.5："The SDK now strips scripting define symbols beginning with VRC_ENABLE_ that it doesn't recognize"）——拼错的 define 不会报错，只会消失

### 15.4 C# 9 parser ≠ UdonSharp 支持 C# 9

**Rule**：UdonSharp 用 Roslyn 做前端解析，能"解析"某种 C# 语法**不代表**能编译到 Udon Assembly。

- 官方 U# 文档原文（https://creators.vrchat.com/worlds/udon/udonsharp/ ，Last updated 2026-10-07）："**UdonSharp is not conformant to any version of the C# language specification.** Some C# features are not implemented or will not work."
- 该页仍在列的不可用项包括：**泛型类与非静态泛型方法、接口、方法重载** 等；`List<T>` 等泛型集合仍未支持（"Generic collections such as List<T> are not supported yet"）
- ⚠️ **官方文档自身的一处矛盾（只记录、不下结论）**：同一页的 "Supported C# Features" 明确列出 **"User defined properties"** 可用，而 FAQ 又把 **"Properties"** 列进"常见不可用技巧"。两者指的应该不是同一件事，但官方没有澄清。实践上以 SDK 内实测为准，不要拿 FAQ 这一条反证自定义属性不可用。
- 因此 **RULE-LL-02 / RULE-LL-12 继续有效**，3.10.4–3.10.6 没有任何一条 release note 放宽它们

### 15.5 其他 3.10.4–3.10.6 与语言/API 面相关的变化

| 版本 | 变化 | 会不会让老代码失效 |
|---|---|---|
| 3.10.4 | `UdonSharpBehaviour` 实现 `IUdonEventReceiver`，省一次显式 cast | 不会（只是少写 cast） |
| 3.10.4 | 修"特定成员数量的 enum 用法被错误编译" | 可能（曾被误编译的代码行为会变） |
| 3.10.5 | U# 脚本创建对话框**显式限制在 `Assets/`**；beta.2 起恢复支持 `Packages/` | 不会 |
| 3.10.5 | 未知 `VRC_ENABLE_*` define 被 strip | 可能（依赖拼错 define 的条件编译会失效） |
| 3.10.5 | ClientSim 在 Play→Edit 时**不再**运行 `OnDisable()` 等 Unity 事件 | **会**：不要再把"离开 Play Mode"当作 gameplay 回调的证据 |
| 3.10.6 | sync null `VRCUrl` + persistence 触发的 Udon 异常修复 | 不会（但依赖异常"恰好不崩"的写法要重估） |

> ⚠️ **证据纪律**：以上均为官方 release note 文案，**没有**在 3.10.6 实机复现。若与本地实测冲突，以实测为准并回写本页。

---

## RULE-LL-15 相关页面

- [[entities/api/vrctween]] — 3.10.4 新增的补间 API（命名参数安全性与 15.2 直接相关）
- [[entities/api/dynamics]] — 3.10.6 约束 source API / SmoothingFactor
- [[entities/api/udon-type-exposure]] — 曝光面快照（未随 3.10.4–3.10.6 重新导出）
- [[networking-rules]] — 网络侧规则
- [[vrchat-api-exposure]] — API 可用性判断规则
- [[entities/api/api-checker]] — 静态检查清单
