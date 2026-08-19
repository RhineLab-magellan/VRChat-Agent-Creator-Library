---
title: VRChat 创作者备份策略
category: platform
knowledge_level: applied
status: active
source: https://docs.vrcd.org.cn/books/86vrchat
source_type: community
version: 1.0
last_review: 2026-07-04
confidence: High
tags:
  - platform
  - backup
  - storage
aliases:
  - 备份策略
  - 创作者备份
  - VRChat Backup Strategy
  - 项目备份策略
related:
  - ../../concepts/hybrid/vcc.md
  - ../../concepts/hybrid/alcom.md
  - ../world/udon/local-storage.md
type: entity
created: 2026-07-04
sources: https://docs.vrcd.org.cn/books/86vrchat
updated: 2026-07-04
---

# VRChat 创作者备份策略

> Domain: Platform | 适用: VRChat 创作者
> 来源: 86vrchat + VCC 官方 + ALCOM 官方

---

## 1. 三个核心存储位置

**[FACT]** VRChat 创作者涉及三个独立的本地存储位置:

### 1.1 Unity Hub 工程记录

| 平台 | 路径 |
|------|------|
| Windows | `%APPDATA%\UnityHub`(即 `<用户目录>\AppData\Roaming\UnityHub`) |
| macOS | `~/Library/Application Support/UnityHub` |
| Linux | `~/.config/UnityHub` |

### 1.2 VCC/ALCOM 工程记录

**[FACT]** VCC 与 ALCOM **共享同一配置位置**:

- Windows: `%LOCALAPPDATA%\VRChatCreatorCompanion`
- 具体: `<用户目录>\AppData\Local\VRChatCreatorCompanion`
- 关键文件: **`settings.json`**(工程路径 + 第三方包记录)

**[FACT-OFFICIAL]** ALCOM 设计为与 VCC **无缝共存**,使用相同 `settings.json`
(来源: [[concepts/hybrid/alcom]] §4 与 VCC 的关系)

### 1.3 VRChat 本地存储

**[FACT-OFFICIAL]** VRChat 游戏本地数据:

- `%UserProfile%\AppData\LocalLow\VRChat\VRChat\`
- 官方文档: https://docs.vrchat.com/docs/local-vrchat-storage

---

## 2. 备份清单

**[COMMUNITY-86vrchat]** 推荐备份以下内容:
(来源: https://docs.vrcd.org.cn/books/86vrchat)

| 备份目标 | 路径 | 重要性 |
|---------|------|--------|
| Unity Hub 工程记录 | `%APPDATA%\UnityHub` | ⭐⭐⭐ |
| VCC/ALCOM 配置 | `%LOCALAPPDATA%\VRChatCreatorCompanion\settings.json` | ⭐⭐⭐ |
| VRChat 缓存的 Avatar | `%UserProfile%\AppData\LocalLow\VRChat\VRChat\` | ⭐⭐ |

---

## 3. settings.json 损坏应对

**[INFERENCE]** VCC 在异常退出时**有概率**损坏 `settings.json`

- 现象: 工程记录丢失
- 修复: 删除 `settings.json` → 重启 VCC/ALCOM
- 来源: 86vrchat + VCC 官方 FAQ

**[FACT]** ALCOM 同样使用该位置,**比 VCC 更稳定**(社区共识)

---

## 4. 关联文档

- `hybrid/vcc.md` — VCC 详细文档
- `hybrid/alcom.md` — ALCOM 详细文档
- `world/udon/local-storage.md` — VRChat 本地存储

---

## 相关页面

[[../../concepts/hybrid/vcc.md]] · [[../../concepts/hybrid/alcom.md]] · [[../world/udon/local-storage.md]]
