---
title: 自身跳 MMD 舞蹈(Self MMD)
category: avatar
subcategory: animator
knowledge_level: applied
status: active
source: yexcadocs §自身跳 MMD 舞蹈(2026-07-04)
source_type: community
version: 1.0
last_review: 2026-07-04
confidence: High
tags:
  - avatar
  - animator
  - mmd
  - dance
  - self-mmd
  - action-layer
  - fx-layer
  - music
aliases:
  - Self MMD
  - 自身跳 MMD
  - 自身跳舞
  - "Avatar MMD Dance"
  - 顺序播放音乐
  - 随机播放音乐
  - 出场动画
related:
  - playable-layers.md
  - animator-system.md
  - expression-menu.md
  - world-fixed-objects.md
  - avatar-dynamic-bone-limits.md
  - avatar-modding-guide.md
type: entity
created: 2026-07-04
sources: yexcadocs §自身跳 MMD 舞蹈(2026-07-04)
updated: 2026-07-04
---

# 自身跳 MMD 舞蹈(Self MMD)

> 来源:yexcadocs §自身跳 MMD 舞蹈 / 音乐播放 / 出场动画(P1/P2)
> 适用场景:在 Avatar 自己身上播放 MMD 舞蹈(非世界固定)
> 本文档为社区实操经验,非 VRChat 官方文档。

---

## 1. 概述

**自身跳 MMD** = Avatar 在玩家自己身上播放 MMD 舞蹈动画,通常配合音乐播放。

### 与"固定世界 MMD"的区别

| 维度 | 自身跳 MMD | 固定世界 MMD |
|------|----------|------------|
| 位置 | 跟随 Avatar | 世界固定 |
| 应用场景 | 玩家自己/好友 Avatar | 场景装饰 |
| 音频源 | 跟随 Avatar(可邦骨到嘴) | 固定在世界 |
| 复杂度 | 中(Action + FX 双层) | 高(5 空物体结构) |

---

## 2. 文件准备

### 2.1 舞蹈动画文件

- 来源:VRC 工具箱(如梦 VRC 工具箱 v1.2.5)转换 MMD 舞蹈
- ⚠️ **动画方向调整**:点击动画文件 → "跟变换旋转-依据" → 选择 "原始"
- 转换步骤详见 `world-fixed-objects.md` 的相关章节

### 2.2 音乐

- 格式:已压缩(Vorbis 格式)
- 大小:计入 Avatar 200MB 总限制

---

## 3. 音频源配置

### 3.1 创建空物体

```
1. 在模型上创建空物体(命名如 "Audio_Music")
2. 添加 Audio Source 组件
3. 配置参数:
   - AudioClip: 音乐文件
   - 空间混合 (Spatial Blend): 1
   - 多普勒级别 (Doppler Level): 0
   - 音量衰减 (Volume Rolloff): 线性衰减
   - 最大距离 (Max Distance): 20(可调)
```

### 3.2 音频邦骨(可选)

- 作者习惯:音频邦骨到嘴部(Head 骨骼)
- 实现:把 AudioSource 拖到 Head 骨骼下作为子集

### 3.3 音频开关

- 录制开关参考 FX 层物品开关流程
- 通过 Animator 动画控制 AudioSource.enabled

---

## 4. Animator 配置(核心)

### 4.1 Action 层配置

进入 Action 层,跟随 `Entry → WaitForActionOrAFK → Prepare Standing`

```
1. "Prepare Standing" 后连接 8 个过渡
2. 任选一个过渡后的状态(命名如 "Dance_State")
3. 将 Motion 替换为舞蹈动画
4. **添加组件 "Animator Locomotion Control"**
   - Locomotion Control = Disable
   - ⚠️ 不设置 → 游戏中转动或移动时模型也会动
5. 检查前后过渡条件
   - 进入条件: 如 VRCEmote == 1
   - 退出条件: 如 VRCEmote != 1
   - 关闭退出时间
6. 下一状态(BlendOut Stand)添加 Animator Locomotion Control
   - Locomotion Control = Enable
   - ⚠️ 不设置 → 跳舞结束后再也无法移动或转动视角
```

**完整状态链**:
```
Entry → WaitForActionOrAFK → Prepare Standing → Dance_State → BlendOut Stand
                                       ↑ Locomotion = Disable
                                                            ↑ Locomotion = Enable
```

### 4.2 FX 层配置

```
1. 添加 Int 型参数,命名为 "VRCEmote"
2. 新建图层,权重 = 1
3. 创建音乐开关状态:
   - 打开状态条件: VRCEmote == 1
   - 关闭状态条件: VRCEmote != 1
4. 关闭退出时间
5. 录制音频启用/禁用动画
```

---

## 5. 参数与菜单

### 5.1 数值配置

- **VRCEmote** 是默认参数,通常已存在
- 如未添加:`Int` 类型

### 5.2 菜单配置

```
菜单项配置:
- Type: Toggle
- Parameter: VRCEmote
- Value: 1(对应 Action 层条件)
```

---

## 6. ⚠️ 关键技术点

### 6.1 Animator Locomotion Control 配对(关键)

**进 Disable,出 Enable 必须配对**:

| 状态 | Locomotion Control |
|------|-------------------|
| 准备跳舞(Dance_State) | **Disable** |
| 退出(BlendOut Stand) | **Enable** |

**错误后果**:
- 缺失 Enable → 跳舞结束后"再也无法移动或转动视角"
- 缺失 Disable → "游戏中转动或移动时模型也会动"

详见 `playable-layers.md §10.1`。

### 6.2 特效不能含脚本(VRChat 上传规则)

- **出场动画中使用的特效必须不含 Script 组件**
- VRChat 不允许上传脚本文件
- 如删除脚本后特效异常,只能换一个

### 6.3 Unity 默认 60 帧/秒

- 动画时间计算: 帧数 / 60 = 秒数
- 240 帧 = 4 秒
- 可在 Animation 窗口查看帧数

### 6.4 动作不能做进场动画(个人理解)

- 作者原话:"因切换到该模型时会执行追踪器重置"
- 标记为 [待证] — 需查证 VRChat 官方

---

## 7. 顺序/随机音乐播放(扩展)

### 7.1 顺序播放

#### 参数组合

| 参数 | 类型 | 用途 |
|------|------|------|
| `Music_Start` | Bool | 播放开关 |
| `Music` | Int | 选择歌曲 |
| `Next` | Bool(可选) | 下一首按钮 |

#### 菜单设计

```
简单开关:
- Toggle: Parameter = Music_Start

游戏中显示当前歌曲(子菜单):
- Toggle: Parameter = Music_Start
- Toggle: Parameter = Music (值 1-N 对应各歌曲)

下一首:
- Button: Parameter = Next
```

### 7.2 随机播放

#### 参数组合

| 参数 | 类型 | 用途 |
|------|------|------|
| `Music_Start` | Bool | 播放开关 |
| `Music` | Int | 选择歌曲 |

#### 实现

- 顺序播放改为多个 `Music` 值
- 添加一个 `Music = 0` 或其他特殊值用于"随机切歌"
- 游戏中点击任意歌曲 = 切换

### 7.3 通用要点

- 顺序/倒序/随机:通过不同 `Music` 值组合实现
- 可扩展到"动画/招式"的随机播放

---

## 8. 出场动画(Entry Animation)

### 8.1 特效处理

```
⚠️ 特效不能包含 Script 组件
- 删除 Script 组件
- 删除后特效异常则换
```

实装:
1. 关闭特效循环(Looping = false)
2. 放在模型上
3. 上传

### 8.2 物品出场动画

```
动画准备:
1. 将物体隐藏
2. 录制"打开"动画(物体显现)

FX 层配置:
1. 新建图层,权重 = 1
2. 默认状态(物体隐藏) → 拖入隐藏动画
3. 创建空状态(物体出现) → 拖入打开动画
4. 过渡: 关闭退出时间,时间长度自定
```

### 8.3 动作出场动画(复杂)

#### 前提

- 创建 Bool 参数(如 `Start`)

#### Action 层配置

```
过渡 1: WaitForActionOrAFK → Prepare Standing
  条件: Start = true
  取消勾选"退出时间"

动作动画状态:
  Motion = 动作动画
  过渡: Prepare Standing → 动作动画 → BlendOut Stand
  - Prepare Standing → 动作动画: 条件 Start = true,无退出时间
  - 动作动画 → BlendOut Stand: 有退出时间 = 1,无条件

BlendOut Stand 添加组件:
  VRC Avatar Parameter Driver
  Parameter Start = false
```

#### FX 层配置

```
参数: Bool Start(与 Action 层同名)
图层: 新建图层,权重 = 1

图层结构: Entry → Default → Hide → Start
- Default: 默认状态(模型可见)
- Hide: 隐藏动画
- Start: 空状态

过渡:
- Default → Hide: 无条件,退出时间 0.1(等待追踪器重置)
- Hide → Start: 无条件,退出时间 0.2(站立 → 动作过渡)

Hide 状态添加组件:
  VRC Avatar Parameter Driver
  Parameter Start = true
```

#### 解释

- `Default → Hide` 过渡时间 = 0.1(等待模型重设追踪器完成)
- `Hide → Start` 过渡时间 = 0.2(模型从站立到动作动画的起始时间)
- 时间可自行调整

---

## 9. VRC 工具箱配置(扩展)

### 9.1 如梦 VRC 工具箱 v1.2.5

- 来源:https://gitee.com/cmoyuer/vrchat-avatar-toolkit/releases
- 作者 B 站:如梦Nya

### 9.2 MMD 转换流程

```
1. 打开 VRC 工具箱
2. 顶部选择 "VRC工具箱 - MMD动作转换"
3. 拖入 MMD 动作文件(vmd)
4. 拖入 MMD 模型(pmx)(可不拖入,建议带上)
5. 点击 "生成配置文件"
6. 等待生成
```

### 9.3 工作流技巧

- **可不带 MMD 模型**转换动作
  - 但有些动作会变怪
  - **建议带上 MMD 模型**
- **复制原模型**再拖入动画(防止错误)
  - 在层级中 `CTRL+D` 复制
  - 隐藏原模型
  - 将动画拖到新模型
  - 点击播放测试

### 9.4 动画方向调整

- 点击动画文件
- "跟变换旋转-依据" 选择 "原始"
- 固定世界模型可选,自身跳 MMD 必须调整

---

## 10. 故障排除

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| 跳舞时角色乱动 | 缺 Locomotion Disable | 在动作状态添加 Locomotion Control = Disable |
| 跳舞后无法移动 | 缺 Locomotion Enable | 在 BlendOut Stand 添加 Locomotion Control = Enable |
| 音乐不播放 | 音频源未启用 | FX 层动画检查 AudioSource.enabled |
| 动画方向错误 | 跟变换旋转未设为"原始" | 修改动画文件设置 |
| 特效失效 | 包含 Script 组件 | 删除 Script 组件 |
| 出场动作失效 | 追踪器重置问题 | 改用特效/物品出场,不动动作 |

---

## 11. 相关文档

- `playable-layers.md §10.1` — Animator Locomotion Control 配对警告
- `animator-system.md` — Animator 系统
- `expression-menu.md` — 菜单 Toggle 控件
- `world-fixed-objects.md` — 固定世界 MMD(不同结构)
- `avatar-dynamic-bone-limits.md §9` — PhysBones 警告
- `avatar-modding-guide.md` — MMD 改模工作流

---

## 12. 文档元信息

- **源文档**:yexcadocs §自身跳 MMD / 音乐播放 / 出场动画
- **本地化时间**: 2026-07-04
- **知识等级**: Applied
- **可信度**: High(社区实操经验)
- **风险点**:
  - yexcadocs 作者已声明不再维护
  - "动作不能做进场动画(追踪器重置)"标记为 [待证]
  - "Locomotion 配对"为社区共识,需结合 VRChat 官方 Animator 文档验证

---

## 相关页面

[[playable-layers.md]] · [[animator-system.md]] · [[expression-menu.md]] · [[world-fixed-objects.md]] · [[avatar-dynamic-bone-limits.md]] · [[avatar-modding-guide.md]]
