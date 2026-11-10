---
title: World 模式 — 动态生成玩家传送按钮
category: world
subcategory: patterns
knowledge_level: applied
status: active
source: 本地知识库整理 + VRChat 官方文档
source_type: community
version: 1.0
last_review: 2026-07-04
confidence: High
tags:
  - world
  - udonsharp
  - pattern
  - ui
  - player-api
  - canvas
  - button
  - vrc-ui-shape
aliases:
  - 动态生成玩家列表
  - 玩家传送按钮
  - 动态玩家列表
  - Dynamic Player List
  - Player Teleporter Buttons
related:
  - ../scene-components/textmeshpro.md
  - ../udon/ui-events.md
source_origin: 从 07-16790.md §Q "地图有多少人就生成多少按钮按下传送到对应玩家..." 提取 (P2)
type: entity
created: 2026-07-04
sources: 本地知识库整理 + VRChat 官方文档
updated: 2026-07-04
---

# World 模式 — 动态生成玩家传送按钮

> 在 World 中根据当前玩家数量动态生成传送按钮,点击后传送到对应玩家位置。
>
> **源 QA**: 07-16790.md §"地图有多少人就生成多少按钮按下传送到对应玩家"

---

## 0. 速查表

| 问题 | 解答 |
|------|------|
| 怎么让 Canvas 在 World 中可交互? | 添加 `VRC_UIShape` 组件 |
| 怎么让 UI 触发 Udon 事件? | Button.OnClick → UdonBehaviour.SendCustomEvent |
| 怎么在 UI 中显示玩家名? | 使用 `TextMeshPro` (VRC SDK 推荐) |
| 怎么获取当前玩家列表? | `VRCPlayerApi.GetPlayers(players)` |
| 怎么传送到玩家? | `player.TeleportTo(targetPos, targetRot)` |

---

## 1. 需求分析

### 1.1 典型场景

- **RPG World**: 玩家点击列表中的名字,传送到该玩家处
- **社交 World**: 看到所有在线玩家,可快速找到朋友
- **解谜 World**: 列出所有玩家进度
- **Tournament World**: 列出所有参赛者

### 1.2 动态 vs 静态

| 方式 | 优点 | 缺点 |
|------|------|------|
| **静态预制 (8 个 Button)** | 简单,性能可控 | 玩家 >8 时不够用 |
| **动态生成 (推荐)** | 数量自适应 | 实现稍复杂,需注意性能 |

---

## 2. 核心技术点

### 2.1 VRC UI Shape

> 来自: [VRChat 官方文档](https://creators.vrchat.com/worlds/components/vrc_uishape/)

**作用**: 让 Canvas 组件可在 World 中被玩家交互

**特点**:
- 玩家可以**指向、点击、滚动** UI (类似 VRChat 菜单)
- 玩家可以**远距离**与 UI 交互(比 Interact 事件更易用)

**配置步骤**:

1. 创建 Canvas
   - 右键 Hierarchy → `UI` → `TextMeshPro (VRC)` (SDK 自动配置)
2. 添加 VRC_UIShape 组件
   - 选中 Canvas GameObject → `Add Component` → `VRC_UI Shape`
3. 配置 Canvas
   - **Render Mode**: 改为 `World Space` (默认 Screen Space 不可用)
   - **Scale**: 改为 `0.01` (默认 1 太大,1 像素=0.01 米)
   - **Layer**: 从 `UI` 改为 `Default` 或其他层
4. 启用 "Allow Focus View"
   - VRC_UIShape 组件参数

### 2.2 UI 事件(Udon)

> 来自: [VRChat 官方文档](https://creators.vrchat.com/worlds/udon/ui-events/)

**UdonBehaviour 的可用事件**:
- `RunProgram` - 运行 Udon 程序
- `SendCustomEvent` - 发送自定义事件
- `Interact` - Interact 调用

**重要约束**:
- ⚠️ `SendCustomEvent` 调用的 U# 方法必须为 **`public`**,否则无效
- ⚠️ UdonBehaviour 出错会"halt"(停止所有功能),需调试
- ⚠️ 事件名必须**完全匹配**(大小写敏感)

### 2.3 TextMeshPro

> 详见: [[entities/world/scene-components/textmeshpro]]

**SDK 推荐**: 使用 TextMeshPro 而非 Unity UI Text

**优势**:
- 高质量字体
- 可缩放不失真
- 性能更好

---

## 3. 实现模式

### 3.1 数据结构

```csharp
[SerializeField] private TextMeshProUGUI[] playerNameTexts;
[SerializeField] private Button[] playerButtons;
[SerializeField] private GameObject buttonPrefab;
[SerializeField] private Transform buttonContainer;
```

### 3.2 核心逻辑

```csharp
using UdonSharp;
using UnityEngine;
using UnityEngine.UI;
using TMPro;
using VRC.SDKBase;
using VRC.Udon.Common.Interfaces;

public class DynamicPlayerList : UdonSharpBehaviour
{
    [SerializeField] private GameObject buttonPrefab;  // 预制 Button + Text
    [SerializeField] private RectTransform buttonContainer;  // Content 父对象
    
    private VRCPlayerApi[] players = new VRCPlayerApi[80];
    private int playerCount;
    
    public override void OnPlayerJoined(VRCPlayerApi player)
    {
        RefreshPlayerList();
    }
    
    public override void OnPlayerLeft(VRCPlayerApi player)
    {
        RefreshPlayerList();
    }
    
    private void RefreshPlayerList()
    {
        // 1. 获取当前玩家
        playerCount = VRCPlayerApi.GetPlayerCount();
        VRCPlayerApi.GetPlayers(players);
        
        // 2. 清除旧按钮
        for (int i = buttonContainer.childCount - 1; i >= 0; i--)
        {
            Destroy(buttonContainer.GetChild(i).gameObject);
        }
        
        // 3. 动态生成按钮
        for (int i = 0; i < playerCount; i++)
        {
            VRCPlayerApi p = players[i];
            if (p == null) continue;
            
            // 实例化按钮
            GameObject btnObj = Instantiate(buttonPrefab, buttonContainer);
            btnObj.SetActive(true);
            
            // 设置玩家名
            TextMeshProUGUI text = btnObj.GetComponentInChildren<TextMeshProUGUI>();
            text.text = p.displayName;
            
            // 绑定点击事件 - 注意:必须用 wrapper
            Button btn = btnObj.GetComponent<Button>();
            // Button.onClick 不能直接传 Udon 事件,需用 wrapper
        }
    }
    
    // 公开方法(供 Button 调用)
    public void TeleportToPlayer(int playerIndex)
    {
        if (playerIndex < 0 || playerIndex >= playerCount) return;
        VRCPlayerApi target = players[playerIndex];
        if (target == null) return;
        
        // 传送到玩家位置
        Vector3 targetPos = target.GetPosition();
        Quaternion targetRot = target.GetRotation();
        Networking.LocalPlayer.TeleportTo(targetPos, targetRot);
    }
}
```

### 3.3 Button 事件绑定(重要)

> ⚠️ **UdonSharp 限制**: `Button.onClick` 不能直接订阅 U# 公共方法

**解决方案 1:每个 Button 实例化时挂载独立的 UdonSharpBehaviour**

```csharp
public class TeleportButton : UdonSharpBehaviour
{
    public int playerIndex;
    public DynamicPlayerList parent;
    
    public void OnClick()
    {
        parent.TeleportToPlayer(playerIndex);
    }
}
```

实例化时:
```csharp
GameObject btnObj = Instantiate(buttonPrefab);
TeleportButton btnLogic = btnObj.AddComponent<TeleportButton>();
btnLogic.playerIndex = i;
btnLogic.parent = this;

Button btn = btnObj.GetComponent<Button>();
// 通过 UnityEvent 绑定(在 Prefab 中预设)
```

**解决方案 2:使用 SendCustomEvent 字符串事件**

```csharp
// 在 Prefab 中预设 Button.onClick → 调用本实例的 OnButtonClicked
// 在 OnButtonClicked 中根据 GetComponent<Button>() 找到是哪个按钮
```

---

## 4. 性能考虑

### 4.1 限制

| 项 | 建议 |
|----|------|
| 按钮数量 | 上限 80 个(VRCPlayerApi 限制) |
| Refresh 频率 | 避免每帧调用,仅在 OnPlayerJoined/Left 触发 |
| 字符串操作 | 避免拼接,使用预分配的 char[] 数组 |
| 销毁 | 用 Destroy 而非 DestroyImmediate |

### 4.2 最佳实践

- ✅ **OnPlayerJoined / OnPlayerLeft** 触发刷新(而非 Update)
- ✅ **对象池** 复用 Button 而非 Instantiate/Destroy
- ✅ **Owner 限制**: 列表生成应在 Master 端进行
- ✅ **同步**: 玩家列表本身无需同步(本地生成即可)

---

## 5. 官方示例参考

> 来自: [Udon Example Scene](https://creators.vrchat.com/worlds/examples/udon-example-scene/)

**VRChat SDK 内置示例**:
- `Packages > VRChat SDK - Worlds > Samples > UdonExampleScene > Prefabs > Udon Variable Sync`
- 包含 `ButtonSyncOwner`、`ButtonSyncAnyone`、`ButtonSyncBecomeOwner` 等模式
- `Canvas` 物件下有多个 UI 元素使用 synced variables

**关键模式**:
- `ButtonSyncOwner`: 玩家点击按钮 → 检查是否是 Owner → 改 count + RequestSerialization
- `ButtonSyncAnyone`: 非 Owner 点击 → 发送 Custom Event 给 Owner
- `ButtonSyncBecomeOwner`: 转移所有权后修改

---

## 6. 常见陷阱

| 陷阱 | 解决方案 |
|------|---------|
| 按钮点击无反应 | 检查 VRC_UIShape 组件是否添加;Allow Focus View 是否开启 |
| Canvas 不显示 | 确认 Render Mode = World Space;Layer 不是 UI |
| SendCustomEvent 无效 | 确认方法为 `public`;事件名拼写完全一致 |
| Text 不显示 | 使用 TextMeshPro 而非 Text;字体资源已导入 |
| 玩家名乱码 | 使用 `displayName` 而非 `name` |
| 列表不更新 | 仅在 OnPlayerJoined/Left 触发,而非每帧 |

---

## 7. 完整检查清单

```
□ Canvas 创建(World Space)
□ VRC_UIShape 组件添加
□ Allow Focus View 启用
□ EventSystem 存在(自动生成)
□ 按钮使用 TextMeshPro 而非 Text
□ Button.onClick → U# 公共方法
□ 方法名为 public
□ 事件名完全匹配
□ 玩家 ID 通过 SetProgramVariable 传递(而非 OnClick 参数)
□ 玩家离开时清理对应按钮
```

---

## 8. 相关文档

- [[entities/world/scene-components/textmeshpro]] — TextMeshPro
- [[entities/world/udon/ui-events]] — UI 事件(待完善)
- [[entities/api/player-api]] — 玩家 API(待定位)
- [[entities/world/sdk-prefabs]] — SDK Prefabs
- [[entities/world/index]] — Udon 示例场景

---

## 9. 引用

- **源 QA**: https://docs.vrcd.org.cn/books/16790
- **VRC UI Shape 官方文档**: https://creators.vrchat.com/worlds/components/vrc_uishape/
- **UI Events 官方文档**: https://creators.vrchat.com/worlds/udon/ui-events/
- **Udon Example Scene**: https://creators.vrchat.com/worlds/examples/udon-example-scene/
- **SDK Prefabs**: https://creators.vrchat.com/worlds/sdk-prefabs/
