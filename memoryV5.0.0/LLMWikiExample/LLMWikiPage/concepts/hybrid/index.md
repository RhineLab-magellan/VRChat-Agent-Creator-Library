---
title: "Hybrid Domain — Knowledge Base"
category: hybrid
knowledge_level: applied
status: active
source: 本地知识库整理
source_type: community
version: 1.3
last_review: 2026-07-12
confidence: Medium
tags:
  - misc
  - index
  - navigation
aliases:
  - "Hybrid Domain — Knowledge Base"
related:
  - "[[osc-protocol.md]]"
  - "[[audio-link.md]]"
  - "[[udon-world-plugins.md]]"
  - "[[avatar-camera-plugins.md]]"
  - "[[vcc.md]]"
  - "[[alcom.md]]"
  - "[[avatar-world-boundaries.md]]"
  - "[[player-observation-boundaries.md]]"
  - "[[player-world-trigger-bridge.md]]"
  - "[[avatar-world-contact-bridge.md]]"
  - "[[avatar-world-sync-model.md]]"
  - "[[FACT]]"
  - ../../entities/api/player-api.md]]
type: concept
created: 2026-07-12
sources: 本地知识库整理
updated: 2026-07-12
---
# Hybrid Domain — Knowledge Base

**Domain**: Hybrid(跨域桥接 / 项目与交互边界)
**Type**: Knowledge Base
**本地化日期**: 2026-07-12
**文件数**: 12

---

## 核心文档

| 文档 | 说明 | 来源 |
|------|------|------|
| **[osc-protocol.md](osc-protocol.md)** | OSC 完整协议数据库、消息格式、路由规则 | VRChat 官方文档 |
| **[audio-link.md](audio-link.md)** | 音频同步系统 API 与使用指南、纹理编码数据传递、时间同步 | 源码验证 |
| **[udon-world-plugins.md](udon-world-plugins.md)** ⭐NEW 2026-06-20 | **推荐 Udon 世界插件索引**(创作者安装 / 玩家识别) | 源码验证 + 案例研究 |
| **[avatar-camera-plugins.md](avatar-camera-plugins.md)** ⭐NEW 2026-07-04 | **Avatar 端摄影工具索引**(玩家安装在 Avatar 上,Flex Fisheye Lens 等) | BOOTH 官方 + VRCD 文档库 |
| **[vcc.md](vcc.md)** ⭐NEW 2026-07-01 | **VRChat Creator Companion 参考** — 架构/工作流/VPM格式/已知问题/CLI | 官方文档全站下载 |
| **[alcom.md](alcom.md)** ⭐NEW 2026-07-01 | **ALCOM 开源替代参考** — 跨平台/并行下载/多语言/vrc-get CLI | 官方仓库 + VRChat Wiki |
| **[avatar-world-boundaries.md](avatar-world-boundaries.md)** ⭐NEW 2026-07-12 | **Avatar↔World 架构边界总纲** — 内部参数层 / 外显观察层 / 桥接层 / 同步层 | 首轮综合入库（以正文分层标记为准） |
| **[player-observation-boundaries.md](player-observation-boundaries.md)** ⭐NEW 2026-07-12 | **玩家/Avatar 外显观察边界** — Position / Tracking / Bone / Eye Height / Scaling | 首轮综合入库（以正文分层标记为准） |
| **[player-world-trigger-bridge.md](player-world-trigger-bridge.md)** ⭐NEW 2026-07-12 | **Player↔World 区域/碰撞/粒子观测桥** — Trigger / Physics Collision / Particle | 首轮综合入库（以正文分层标记为准） |
| **[avatar-world-contact-bridge.md](avatar-world-contact-bridge.md)** ⭐NEW 2026-07-12 | **Avatar↔World Contact / Dynamics 桥接边界** | 首轮综合入库（以正文分层标记为准） |
| **[avatar-world-sync-model.md](avatar-world-sync-model.md)** ⭐NEW 2026-07-12 | **Avatar↔World 同步模型** — Event / Owner / `[UdonSynced]` / Late Joiner | 首轮综合入库（以正文分层标记为准） |

---

## 子域

| 子域 | 内容 | 状态 |
|------|------|------|
| **OSC** | Open Sound Control 协议集成 | ✅ 已完成 |
| **音频同步系统(参考工程)** | 音频可视化核心系统架构 + API 使用指南 | ✅ 已完成 |
| **推荐 Udon 世界插件** | 创作者安装的工业级插件索引(TLP UdonVoiceUtils + VizVid VVMW) | ✅ 已完成 (2026-06-20) |
| **推荐 Avatar 端摄影工具** ⭐NEW 2026-07-04 | 玩家安装在 Avatar 上的摄影增强工具(Flex Fisheye Lens) | ✅ 已完成 (2026-07-04) |
| **项目与依赖管理** ⭐NEW 2026-07-01 | VCC + ALCOM + VPM 分发机制 | ✅ 已完成 |
| **Avatar↔World** ⭐NEW 2026-07-12 | 内部参数边界 + 观察层 + Trigger桥 + Contact桥 + Sync模型 | ✅ 主骨架已建立（首轮入库完成，仍待补证） |
| **Tracking** | 已并入 Avatar↔World 观察层子主题 | ✅ 已并入 |
| **External** | 第三方系统联动（MIDI, Web, etc.） | 待建设 |

---

## 核心设计模式（既有沉淀）

### 1. Shader-Centric 数据传递
**原理**：高频变化数据编码为纹理，Shader 直接采样，零网络同步开销

### 2. Master 权威时间锚点
**原理**：Master 写入实例加入时间，其他客户端本地计算 elapsed time

### 3. 位域压缩同步
**原理**：单个 byte/bit 字段存储多个布尔值，减少序列化带宽

### 4. 漂移校正算法
**原理**：本地时间与网络时间差值缓慢收敛，避免瞬时抖动

---

## 待收录内容

- External 第三方系统集成案例（MIDI、Web 等）
- 多机位导演系统(参考工程)中的 Hybrid 模式参考
- Avatar↔World 主题下更细的 Pickup / Station / 其他交互桥定位

---

## 相关知识

- [[FACT]] → 核心设计模式 section
- （来源：Open Source Projects，本地整理） → 开源参考工程
- [[entities/api/player-api]] → VRCPlayerApi 追踪接口
- [[concepts/hybrid/avatar-world-boundaries]] → Avatar↔World 总纲
