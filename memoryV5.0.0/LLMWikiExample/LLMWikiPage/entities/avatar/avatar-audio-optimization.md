---
title: VRChat Avatar 音频优化
category: avatar

knowledge_level: applied
status: active

tags:
  - avatar
  - audio
  - audioclip
  - compression
  - load-type
  - sample-rate
  - optimization

aliases:
  - "Avatar Audio Optimization"
  - "音频优化"
  - "Audio Clip 配置"
  - "音频压缩"
  - "Avatar 音频"

related:
  - optimization-guide.md
  - avatar-size-limits.md
  - performance-rank.md

source: 18-vrchat-CI4 (VRCD) + Unity 官方 Audio Clip Manual (2022.3)
source_type: community
version: 1.0
last_review: 2026-07-04
confidence: High
type: entity
created: 2026-07-04
sources: 18-vrchat-CI4 (VRCD) + Unity 官方 Audio Clip Manual (2022.3)
updated: 2026-07-04
---

# VRChat Avatar 音频优化

> **来源**:
> - Unity 官方 Audio Clip 文档 (2022.3): https://docs.unity3d.com/2022.3/Documentation/Manual/class-AudioClip.html
> - VRCD 18-vrchat-CI4 §音频方面: https://docs.vrcd.org.cn/books/vrchat-CI4
> - 知识库沉淀日期: 2026-07-04
> **状态**: ✅ FACT (Unity 官方 + VRCD 社区)
> **关键 Insight**: 知识库中**音频部分长期空白** — 本文档填补此缺口，整合 Unity 官方对 Audio Clip 4 压缩格式 + 3 加载类型的完整描述。

---

## 1. 音频压缩格式 ⭐K05

> [FACT] **Unity 支持的音频压缩格式**: PCM、ADPCM、Vorbis、MP3（仅 Android）

### 1.1 PCM（无压缩）

| 维度 | 说明 |
|------|------|
| **压缩** | 无（原始音频数据） |
| **体积** | **最大** |
| **CPU 消耗** | **最低**（无需解压） |
| **适用场景** | **短、播放频繁**的音频 |

> [FACT] **Unity 官方描述**:
>
> "PCM: higher quality at the expense of larger file size and is best for short sound effects."

**典型应用**: 脚步声、撞击声、武器声音

### 1.2 ADPCM

| 维度 | 说明 |
|------|------|
| **压缩** | 有损（PCM 差分编码） |
| **体积** | **固定为 PCM 的 27%**（约 3.5 倍压缩） |
| **CPU 消耗** | 低（解压快速） |
| **适用场景** | **短、播放频繁、音质要求不高** |

> [FACT] **Unity 官方描述**:
>
> "ADPCM: useful for sounds that contain a fair bit of noise and need to be played in large quantities, such as footsteps, impacts, weapons."

> ⚠️ **质量警告**: 由于 ADPCM 的编码特性，**不同音频内容的压缩效果不一**。高频率音频使用 ADPCM 压缩质量相对较差 → **应始终检查 ADPCM 压缩后的音频质量**

**典型应用**: 噪声、爆炸声、脚步细节

### 1.3 Vorbis（Ogg Vorbis）

| 维度 | 说明 |
|------|------|
| **压缩** | 有损 |
| **体积** | **最小**（压缩率优秀） |
| **CPU 消耗** | **最高**（解压开销大） |
| **适用场景** | **长、播放不频繁**的音频 |

**关键特性**:
- 使用 Ogg 作为容器（"Ogg Vorbis"）
- 只能进行**可变比特率（VBR）**编码 → **不同音频压缩率不同**
- **质量**（Quality）参数可调整压缩率与音质的平衡

> ⚠️ **性能权衡**:
> - PC 平台使用 Vorbis 时，**推荐 Android 平台使用 MP3**（CPU 考虑）

**典型应用**: 音乐、歌曲、舞蹈配乐

### 1.4 MP3（仅 Android 平台可用）

| 维度 | 说明 |
|------|------|
| **压缩** | 有损 |
| **体积** | 比 Vorbis 大（压缩率较低） |
| **CPU 消耗** | 比 Vorbis 小 |
| **平台** | ⚠️ **仅 Android 平台** |

> ⚠️ **不支持无缝循环**（重要限制）

**典型应用**: Quest 平台音乐、歌曲替代 Vorbis

### 1.5 压缩格式选型决策表

| 场景 | 推荐格式 | 理由 |
|------|----------|------|
| 脚步声 / 撞击声 / 武器声 | **ADPCM** | 短+频繁，27% 体积，CPU 低 |
| 噪声 / 爆炸声 | **ADPCM** | 短+频繁，对音质要求不高 |
| 极短音（< 1s） | **PCM** | CPU 消耗最低 |
| 音乐 / 歌曲（PC） | **Vorbis** | 长+不频繁，体积最小 |
| 音乐 / 歌曲（Quest） | **MP3** | CPU 比 Vorbis 低 |
| 频繁播放的长音频 | ❌ 避免 | Vorbis 每次播放都消耗 CPU |

---

## 2. 加载类型 ⭐K06

> [FACT] **Unity 提供了 3 种音频加载类型**，每种都涉及不同的 CPU/内存权衡

### 2.1 Decompress On Load（加载时解压缩）

| 维度 | 说明 |
|------|------|
| **行为** | **加载模型时**将音频解压为 PCM |
| **CPU 消耗** | 加载时一次，**播放时最低** |
| **内存占用** | **极高**（音频必须在内存中保持 PCM） |
| **适用场景** | **频繁播放 + 长度短** |

> [FACT] **Unity 官方描述**:
>
> "Decompress On Load: Audio files are decompressed as soon as they're loaded. Use this option for smaller compressed sounds to avoid the performance overhead of decompressing on the fly. **Be aware that decompressing Vorbis-encoded sounds on load will use about ten times more memory than keeping them compressed (for ADPCM encoding it's about 3.5 times)**, so don't use this option for large files."

**关键数据**:
- Vorbis: 加载时解压 → **~10x** 内存
- ADPCM: 加载时解压 → **~3.5x** 内存

**典型应用**: 频繁触发的音效、脚步声、枪械声

### 2.2 Compressed In Memory（压缩内存）

| 维度 | 说明 |
|------|------|
| **行为** | 加载时**保持压缩状态**，**播放时动态解压缩** |
| **CPU 消耗** | **每次播放消耗 CPU**（Unity 不会缓存 PCM） |
| **内存占用** | **低**（适合大量音频） |
| **适用场景** | **不经常播放 + 长度长** |

> [FACT] **Unity 官方描述**:
>
> "Compressed In Memory: Keep audio compressed in memory and decompress while playing. This option has a slight performance overhead, especially for Ogg/Vorbis compressed files. Use it only for files that consume excess memory for the Decompressed on Load. **The decompression happens on the mixer thread**..."

**关键点**:
- ⚠️ **每次播放都要消耗 CPU 进行解压**（不会替换原始压缩内容）
- 解压发生在 **mixer thread**（独立线程）
- **可监控**: Audio Profiler 窗口的 DSP CPU 部分

**典型应用**: 歌曲、舞蹈配乐

### 2.3 Streaming（流式处理）

| 维度 | 说明 |
|------|------|
| **行为** | 播放时**动态从硬盘读入内存** |
| **CPU 消耗** | **最高**（三种加载类型中） |
| **内存占用** | **最低** |
| **适用场景** | ❌ **Avatar 中绝不使用** |

> [FACT] **Unity 官方描述**:
>
> "Streaming: Decode continuous audio. This method uses a minimal amount of memory to buffer compressed data that's incrementally read from the disk and decoded spontaneously... **Note: Streaming clips have an overhead of approximately 200KB, even if none of the audio data is loaded.**"

**关键数据**:
- **200KB overhead**（即使没数据也有）

> ⚠️ **Avatar 中绝不使用 Streaming 的原因**:
> - VRChat Avatar 是 AssetBundle 的一部分，文件 IO 行为不同
> - 双 CPU + I/O 消耗
> - 操作系统可同时打开文件数有限制
> - 来源: VRCD 18-vrchat-CI4

### 2.4 加载类型选型决策表

| 加载类型 | CPU 周期 | 内存占用 | 适用场景 |
|----------|----------|----------|----------|
| Decompress On Load | **加载时一次 / 播放最低** | **极高（Vorbis 10x, ADPCM 3.5x）** | 短+频繁 |
| Compressed In Memory | **每次播放消耗** | **低** | 长+不频繁 |
| Streaming | **最高** | **最低（200KB overhead）** | ❌ Avatar 禁用 |

---

## 3. 量化数据 ⭐K07

### 3.1 原始测试条件

> [FACT] **来源**: VRCD 18-vrchat-CI4 §附录

| 项目 | 值 |
|------|-----|
| 原始格式 | WAV |
| 长度 | **02:26** |
| 内容 | 歌曲 |
| 采样率 | 48,000 Hz |
| 原始体积 | **26.8 MB** |

### 3.2 压缩效果参考表

| 压缩格式 | 品质 | 采样率 (Hz) | 文件大小 (MB) |
|----------|------|-------------|---------------|
| PCM      | N/A  | 48,000      | 26.8          |
| ADPCM    | N/A  | 48,000      | 7.5           |
| Vorbis   | 100  | 48,000      | 7.2           |
| Vorbis   | 50   | 48,000      | 2.8           |
| Vorbis   | 1    | 48,000      | **1.3**       |
| PCM      | N/A  | 22,050      | 12.3          |
| ADPCM    | N/A  | 22,050      | 3.5           |
| Vorbis   | 100  | 22,050      | 3.0           |
| Vorbis   | 50   | 22,050      | 1.6           |
| Vorbis   | 1    | 22,050      | 0.7           |

> ⚠️ Vorbis 是 VBR（可变比特率），**不同音频压缩率不同** → **数值仅供参考**

### 3.3 关键数据点

- **Vorbis 1 / 48kHz**: 1.3MB（**4.85% 压缩率**）→ **解压后 26.8MB（膨胀 20.6 倍）**
- **PCM 48kHz → 22.05kHz**: 26.8MB → 12.3MB（54% 体积）
- **Vorbis 100 → 50**: 约 39% 体积
- **Vorbis 50 → 1**: 约 46% 体积

### 3.4 内存膨胀倍数（Unity 官方数据）

> [FACT] **Unity 官方补充数据**（VRCD 文档**未提供**的官方数据点）:

| 格式 | Decompress On Load 内存膨胀 |
|------|------------------------------|
| **Vorbis** | **~10x**（典型值） |
| **ADPCM** | **~3.5x**（典型值） |
| **Streaming** | **~200KB overhead**（即使没数据） |

**与 VRCD 数据点关系**:
- VRCD 文档的 20.6 倍是 **Vorbis Quality=1 的极端压缩**（4.85% 压缩率）
- Unity 官方的 10 倍是 **典型压缩比**（约 10% 压缩率）
- **两者不矛盾** — 不同场景的不同表述

---

## 4. 采样率设置 ⭐K11 (P4 外部验证不完整)

> ⚠️ **本节内容外部验证不完整，待补充**

### 4.1 三种选项

| 选项 | 行为 |
|------|------|
| **Preserve Sample Rate（保持采样率）** | 使用原始音频文件的采样率 |
| **Optimize Sample Rate（优化采样率）** | Unity 根据音频内容自行设置合适采样率 |
| **Override Sample Rate（覆盖采样率）** | 手动设置采样率 |

### 4.2 关键警告

> [FACT] **来源: VRCD 18-vrchat-CI4**
>
> ⚠️ **信息不能无中生有**: 设置为比原始音频文件**更高**的采样率**不能提高音质**，只能**浪费**存储空间
>
> ⚠️ **采样率越高，Unity 的 DSP 负载越高**

### 4.3 体积关系

- **采样率越低** → 体积越小 → 音频质量越差
- 数字化的过程：连续模拟量 → 离散数据格式（采样）

> 📌 **外部验证建议**: Unity 官方 Audio Clip 文档对 Sample Rate 的详细描述（本次未抓取完整）

---

## 5. Load In Background ⭐K12 — VRChat SDK 强制

> [FACT] **VRChat SDK 强制约束**（来源: VRCD 18-vrchat-CI4）
>
> 如果加载类型（Load Type）选择**"加载时解压缩（Decompress On Load）"**，则 **VRChat SDK 要求必须启用"后台加载（Load In Background）"**

### 5.1 Unity 官方描述

> [FACT] **Unity 官方**:
>
> "Load In Background: When this option is enabled, **the loading of the clip will happen at a delayed time on a separate thread, without blocking the main thread**."

### 5.2 强制约束原因

- Decompress On Load 在加载时**同步解压** → 主线程阻塞
- 启用 Load In Background → **独立线程**延迟加载 → 不阻塞主线程
- **VRChat SDK 强制**启用避免上传失败

### 5.3 其他音频属性（简要）

| 属性 | 说明 |
|------|------|
| **Force To Mono** | 打包时将多声道缩减为单声道 |
| **Ambisonic** | 用于 360°/XR 视频，一般不需要勾选 |
| **Preload Audio Data** | 场景加载后加载 Audio Clip，一般打开即可 |

---

## 6. 选型决策树

### 6.1 短音效决策（脚步声、武器声）

```
音频类型？
├─ < 1s + 极频繁播放 → PCM + Decompress On Load
├─ < 5s + 频繁播放 → ADPCM + Decompress On Load
└─ 噪声 / 爆炸 → ADPCM + Decompress On Load
```

### 6.2 长音频决策（音乐、歌曲）

```
平台？
├─ PC → Vorbis + Compressed In Memory
└─ Quest → MP3 + Compressed In Memory
```

### 6.3 Quest 平台决策

```
音频长度？
├─ < 5s → ADPCM + Decompress On Load + Load In Background ✅
├─ 5s - 30s + 频繁 → ADPCM + Compressed In Memory + Load In Background
├─ > 30s + 不频繁 → MP3 + Compressed In Memory
└─ 任何场景 → ❌ 绝不使用 Streaming
```

### 6.4 整体决策矩阵

| 场景 | 格式 | Load Type | Load In BG |
|------|------|-----------|------------|
| 脚步声（PC） | ADPCM | Decompress On Load | ✅ 必开 |
| 脚步声（Quest） | ADPCM | Decompress On Load | ✅ 必开 |
| 武器声（PC） | ADPCM | Decompress On Load | ✅ 必开 |
| 短音效（< 1s） | PCM | Decompress On Load | ✅ 必开 |
| 音乐（PC） | Vorbis | Compressed In Memory | 推荐 |
| 音乐（Quest） | MP3 | Compressed In Memory | 推荐 |
| 任何场景 | - | Streaming | ❌ 禁用 |

---

## 7. FAQ

### 7.1 Vorbis vs ADPCM 选哪个？

> **答**: 看使用场景
> - **短 + 频繁** → ADPCM（27% 体积，CPU 低）
> - **长 + 不频繁** → Vorbis（压缩率最高，但 CPU 高）
> - **Quest 平台** → MP3（Android 平台替代 Vorbis）

### 7.2 为什么 Avatar 不能用 Streaming？

> **答**:
> 1. 双 CPU + I/O 消耗（性能浪费）
> 2. 操作系统可同时打开文件数有限制
> 3. VRChat Avatar 是 AssetBundle 的一部分，文件 IO 行为不同
> 4. 来源: VRCD 18-vrchat-CI4 + Unity 官方"200KB overhead 即使没数据"

### 7.3 音频解压占 VRAM 吗？

> **答**:
> - **Decompress On Load** 解压的 PCM 占用 **RAM**（不是 VRAM）
> - **VRAM 只算贴图**（详见 `avatar-size-limits.md §1.2`）
> - 音频解压产生的 RAM 占用**不计入** "Uncompressed Size" 限制
>   > [FACT] "音频文件解压为 PCM 后的体积不算在模型性能限制中的'未压缩大小'，未压缩大小只计算 AssetBundle 解压后大小"
>   > — VRCD 18-vrchat-CI4

### 7.4 加载时解压缩导致 26.8MB 占用计入模型大小吗？

> **答**: **不计入**。
> - Uncompressed Size 限制只计算 AssetBundle 解压后大小
> - 音频 PCM 在内存中的占用是 **运行时** 占用，不影响上传限制
> - 但仍可能影响玩家 RAM 容量

### 7.5 30 分钟的演唱会 CD 怎么办？

> [FACT] **来源: VRCD 18-vrchat-CI4**
>
> "你真的要随身携带 30 分钟的演唱会 CD？你也不想用 Soundpad？好吧，如果你真的要坚持，你应该选**压缩内存（Compressed In Memory）**并使用 **Vorbis** 压缩格式"
> — VRCD 18-vrchat-CI4

---

## 8. 引用与参考

### 8.1 官方来源

- [Unity 官方 Audio Clip 文档 (2022.3)](https://docs.unity3d.com/2022.3/Documentation/Manual/class-AudioClip.html)
  - PCM/ADPCM/Vorbis/MP3 压缩格式描述
  - Decompress On Load/Compressed In Memory/Streaming 加载类型
  - Load In Background 描述
  - 内存膨胀倍数（Vorbis ~10x / ADPCM ~3.5x）

### 8.2 社区来源

- [VRCD 18-vrchat-CI4](https://docs.vrcd.org.cn/books/vrchat-CI4) — 完整音频优化章节
  - 量化数据表（26.8MB / 02:26 / 48000Hz 测试）
  - 选型决策树
  - 中文工程化建议

### 8.3 知识库内部

- `optimization-guide.md` — 速查要点
- `avatar-size-limits.md` — VRAM vs Uncompressed Size 区分
- `performance-rank.md` — Quest 平台音频标准

### 8.4 知识整理笔记

- `../../../../（见源库 raw/ 目录）参考文献/18-vrchat-CI4-notes/05-audio-compression-formats.md`
- `../../../../（见源库 raw/ 目录）参考文献/18-vrchat-CI4-notes/06-audio-load-types.md`
- `../../../../（见源库 raw/ 目录）参考文献/18-vrchat-CI4-notes/07-audio-compression-reference.md`
- `../../../../（见源库 raw/ 目录）参考文献/18-vrchat-CI4-notes/11-audio-sample-rate.md`
- `../../../../（见源库 raw/ 目录）参考文献/18-vrchat-CI4-notes/12-load-in-background.md`

---

## 相关页面

[[optimization-guide.md]] · [[avatar-size-limits.md]] · [[performance-rank.md]]
