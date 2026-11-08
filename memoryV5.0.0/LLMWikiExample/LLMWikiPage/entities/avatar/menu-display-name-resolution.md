---
title: "LI 衣柜与 MA 菜单的显示名来源机制（menuName / label 回退链）"
category: avatar
knowledge_level: applied
status: active
source: "lilxyzw/lilycalInventory 1.5.2 与 nadena.dev/modular-avatar 1.18.7 源码实测 + 真实头像批量改名实操（2026-11-09）"
source_type: community
version: 1.0
last_review: 2026-11-09
confidence: High
tags:
  - avatar
  - wardrobe
  - modular-avatar
  - expression-menu
  - animator
aliases:
  - "LI 衣柜显示名机制"
  - "MA 菜单显示名回退链"
  - "菜单名本地化"
related:
  - lilycal-inventory.md
  - modular-avatar.md
  - expression-menu.md
  - avatar-modding-guide.md
  - animator-system.md
type: concept
created: 2026-11-09
sources: "lilxyzw/lilycalInventory Runtime/ObjHelper.Component.cs；nadena.dev/modular-avatar ModularAvatarMenuItem.cs 与 Editor/Inspector/Menu/MenuItemGUI.cs"
updated: 2026-11-09
---

# LI 衣柜与 MA 菜单的显示名来源机制

> 来源：lilycalInventory 1.5.2 与 Modular Avatar 1.18.7 的**源码实测**（不是文档转述），外加一次真实头像的 76 项菜单 + 12 格衣柜批量改名实操验证。
> 置信度：**High** —— 核心结论条条能指到源码行号；规模数据来自单例实测，标为实测值而非普适结论。
> 本页价值：**它决定「改菜单显示名」到底该改哪个字段** —— 改错会静默失效，或断开动画。

---

## 1. 为什么这件事容易做错

给 VRChat 头像做菜单本地化时，最常见的直觉是「把 Hierarchy 里的 GameObject 改成中文不就行了」。

**这个直觉会踩两个坑**：

| 坑 | 后果 |
|---|---|
| 改 GameObject 名期望菜单变中文 | 有时**确实有效**（空 label 会回退到 GameObject 名），让人误以为「这样就是标准做法」 |
| 但 GameObject 名可能被动画路径引用 | 改名后动画片段里的路径对不上，**动画静默失效**——不报错，只是不动了 |

因此正确做法**不是**「改 GameObject 名」，而是**写菜单自己的名字字段**，让 GameObject 名保持原样。

[FACT, 2026-11-09] 两条源码回退链分别见 §2、§3；路径引用风险见 §4.2 的实测扫描结果。

---

## 2. LI（lilycalInventory）的显示名回退链

### 2.1 组件级：MenuFolder / ItemToggler 等

源码 `Runtime/ObjHelper.Component.cs` 的 `GetMenuName(this MenuBaseComponent)`：

```csharp
var name = component.menuName;
if(string.IsNullOrEmpty(name)) name = component.gameObject.name;
return name;
```

**回退链（组件级）**：

```
menuName（组件上手动填的）
   ↓ 为空时
gameObject.name
```

### 2.2 Costume 级：衣柜里每一套衣服

同文件 `GetMenuName(this Costume costume, CostumeChanger changer)` 有**更长的六级回退链**：

| 优先级 | 取值来源 |
|---|---|
| ① | `costume.menuName`（手动填的） |
| ② | 第一个开关对象的 GameObject 名 |
| ③ | 第一个 BlendShape 名 |
| ④ | 第一个替换材质名 |
| ⑤ | 第一个 Float 材质属性名 |
| ⑥ | 第一个 Vector 材质属性名 |
| ⑦ | 第一个动画片段名 |

源码形态（节选，保留结构）：

```csharp
if(!string.IsNullOrEmpty(costume.menuName)) return costume.menuName;        // ①
var obj = costume.parametersPerMenu.objects
    .Select(o => o.obj).FirstOrDefault(o => o && !string.IsNullOrEmpty(o.name));
if(obj && !string.IsNullOrEmpty(obj.name)) return obj.name;                // ②
// ③ BlendShape → ④ MaterialReplacer → ⑤ Float → ⑥ Vector → ⑦ Clip → null
```

**所以衣柜里的衣服显示名，绝大多数情况来自 ② —— 那套衣服的 GameObject 名。** [FACT, 2026-11-09]

> 📌 实操含义：想给衣柜换中文名，**写 `costume.menuName`**，不要改 GameObject 名。写了 menuName 就命中 ①，后面六条回退全部跳过。

---

## 3. MA（Modular Avatar）的显示名回退链

### 3.1 构建期：label 为空则回退 GameObject 名

源码 `ModularAvatarMenuItem.cs`（`CloneToVRCSDK()` 内）：

```csharp
name = string.IsNullOrEmpty(BackingMenuItem.label)
    ? BackingMenuItem.gameObject.name      // ← label 空：用 GameObject 名
    : BackingMenuItem.label                // ← label 有值：用 label
```

**回退链（MA）** 只有两级，比 LI 简单：

```
label（菜单项上手动填的）
   ↓ 为空时
gameObject.name
```

### 3.2 编辑器期：Inspector 里那个「名称」框显示的到底是谁

源码 `Editor/Inspector/Menu/MenuItemGUI.cs`：

```csharp
_name = gameObjects.FindProperty("m_Name");   // 绑的是 GameObject 的 m_Name

// 依 _useLabel 二选一绘制
if (_useLabel) { EditorGUILayout.PropertyField(_prop_label, G("menuitem.prop.name")); }
else           { EditorGUILayout.PropertyField(_name,        G("menuitem.prop.name")); }

_useLabel |= !string.IsNullOrEmpty(_prop_label.stringValue);
```

**这是最容易误判的地方**：

| 状态 | Inspector 里「名称」框编辑的是 |
|---|---|
| `label` 为空 | **GameObject 名**（`m_Name`） |
| `label` 有值 | **label 本身** |

也就是说——**同一个标题为「名称」的输入框，点开时改的东西可能完全不同**。看到框里写着 `Bag On`，不代表它就是 label；很可能是 GameObject 名。

[FACT, 2026-11-09] 判断方法：看组件是否出现「使用自定义名称」一类勾选项（勾了才是 label 模式），或直接读序列化数据里的 `label` 字段值，**不要靠 Inspector 显示内容反推**。

---

## 4. 正确改法：写字段，不动 GameObject

### 4.1 该改哪个字段

| 目标 | 要写的字段 | 类型 |
|---|---|---|
| LI 菜单文件夹名 | `MenuFolder.menuName` | string |
| LI 单件开关名 | `ItemToggler.menuName` | string |
| LI 衣柜里第 N 套衣服 | `CostumeChanger.costumes[N-1].menuName` | string |
| MA 菜单项 | `ModularAvatarMenuItem.label` | string |

> 注意衣柜那条的 `[N-1]` —— 索引换算陷阱见 §6.1。

### 4.2 为什么不能改 GameObject 名

[FACT, 2026-11-09] **实测**：一次真实头像工程里有 **15 个** 菜单相关 GameObject 名被动画片段的 `path:` 字段直接引用：

```
HairClip   Bag      Foot_R   Foot_L   Hand_L   Hand_R   Ass    Pussy
Chest      Dick     KissSFX  RealKissSystem  Heart_Trajectory  Box2
```

动画片段里的写法（`.anim` YAML）：

```yaml
- curve:
    path: Bag            # ← GameObject 层级路径，改名即失效
    attribute: m_IsActive
```

改名之后 `path: Bag` 找不到对象，**Unity 不报错，动画就是不执行**。这是最难排查的一类问题。

> ⚠️ 注意区分：动画**文件名**里出现名字（如 `SL_Bag_On.anim` 内含 `m_Name: SL_Bag_On`）不算引用，那是片段自己的名字。只有 `path:` 字段里的路径段才是真引用。

### 4.3 引用检查思路

遍历工程内全部 `.anim`，抽取每条 `path:` 字段并按 `/` 拆成路径段去重，再拿待改名的目标逐个比对。命中即**不可改名**。

要点：
- 只认 `path:` 行，不认 `m_Name:` 行
- 路径要拆段比对（`A/B/C` 里任何一段都可能被单独引用）
- 结果去重，同一名字多个片段引用只报一次

---

## 5. Unity 序列化中文的转义坑

改完保存后，想用文本搜索确认「中文写进去了」，会发现**搜不到**。

[FACT, 2026-11-09] Unity 的 YAML 会把非 ASCII 字符转义，且**两套规则混用**：

| 字符码位 | 存储形式 | 例子 |
|---|---|---|
| ≤ U+00FF | `\xNN`（两位十六进制） | `·` U+00B7 → `\xB7` |
| > U+00FF | `\uNNNN`（四位） | `星` U+661F → `\u661F` |

所以「星空水色」在场景文件里长这样：

```yaml
value: "\u661F\u7A7A\u6C34\u8272"
```

**验证时必须按这两套规则分别转义**，否则会误判「没写进去」：

```
逐字符判断码位：
  码位 > 127 且 ≤ 255  → 输出 \xNN
  码位 > 255           → 输出 \uNNNN
  其余                 → 原样输出
```

> 🔴 **踩过的真实误判**：只用 `\uNNNN` 一种规则验证，含 `·`（U+00B7）的 24 个译名全部「搜不到」，一度以为保存失败；实际 Unity 存成了 `\xB7`，文件完全正确。**先怀疑自己的验证方法，再怀疑保存动作。**

---

## 6. 批量改写时的数组索引陷阱

### 6.1 menuName 展平后有个「幽灵第 0 项」

[FACT, 2026-11-09] 读取 `CostumeChanger` 的序列化属性时，返回的 `menuName` 列表结构是：

```
menuName[0] = ''            ← 组件自身的 menuName（不是第 1 格衣服！）
menuName[1] = '第一套衣服'
menuName[2] = '第二套衣服'
...
```

**组件自身的 `menuName` 与 `costumes[]` 里每格的 `menuName` 是同名字段**，展平后混在一起。

> 🔴 **踩过的真实事故**：按「第 8 格」写入时用了 `costumes.Array.data[8]`，实际写到了第 9 格，**覆盖掉第 9 格原本已有的名字**。
>
> 正确换算：**第 N 格 → `costumes.Array.data[N-1]`**。
> 回读时同样要留意 +1 偏移（`menuName[0]` 是幽灵项）。

### 6.2 安全的批量写入流程

```
1. 先校验一次属性路径合法性（dry run）
2. 只写 1 个，回读确认值正确
3. 再批量写剩下的
4. 全量回读，列出仍不合格的项
5. 保存后按 Unity 转义规则验证磁盘
```

**不要跳过第 2 步** —— 上面那个覆盖事故就是跳过单点验证导致的。

---

## 7. 实测规模数据

[FACT, 2026-11-09] 一次真实头像（Unity 2022.3.22f1，lilycalInventory 1.5.2 + Modular Avatar 1.18.7）：

| 范围 | 数量 |
|---|---|
| LI 服装衣柜（CostumeChanger） | 1 个组件 / 9 格 |
| LI 发型衣柜 | 1 个组件 / 3 格 |
| MA 菜单项（ModularAvatarMenuItem） | 全场景 146 个（该 avatar 下 76 个） |
| 其中 label 为空、靠 GameObject 名显示 | 71 / 76 |
| 被 `.anim` path 引用的候选改名目标 | 15 个（必须避开） |

**结论：绝大多数菜单项的 label 都是空的**，所以看起来「菜单里全是英文 GameObject 名」。给它们补 label 即可全部本地化，GameObject 名一个都不用动。

> 单例实测，规模数字仅供估算参考，不同作者的头像差异可能很大。

---

## 8. 检查清单

改菜单显示名前：

- [ ] 确认目标是 LI 还是 MA（两者回退链不同）
- [ ] 扫描全部 `.anim` 的 `path:` 字段，列出**不可改名**的 GameObject
- [ ] 决定只写字段（推荐）还是连 GameObject 名一起改（有动画风险）

批量写入时：

- [ ] 先校验属性路径合法性
- [ ] 单点写 1 个并回读
- [ ] 注意「第 N 格 = data[N-1]」的索引换算
- [ ] 全量回读验证
- [ ] 保存后按 `\xNN` / `\uNNNN` 两套转义规则验证磁盘

---

## 相关页面

- [[lilycal-inventory.md]]（LI 衣柜本体，本页补它的显示名机制）
- [[modular-avatar.md]]（MA 组件总览）
- [[expression-menu.md]]（表情菜单体系）
- [[avatar-modding-guide.md]]（改模全流程）
- [[animator-system.md]]（动画路径与 Animator 行为）
