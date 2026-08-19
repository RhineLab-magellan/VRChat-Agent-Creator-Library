---
title: "Cloth — 世界物理布料组件"
category: world
subcategory: scene-components
knowledge_level: applied
status: active
source: "Unity Manual 2022.3 + VRChat 官方白名单"
source_type: official
version: 1.0
last_review: 2026-08-19
confidence: high
tags:
  - world
  - unity
  - physics
  - collider
aliases:
  - Cloth
  - 布料组件
related:
  - ../whitelisted-world-components.md
  - skinnedmeshrenderer.md
  - meshrenderer.md
  - capsulecollider.md
  - spherecollider.md
type: entity
created: 2026-08-19
sources: "https://docs.unity3d.com/2022.3/Documentation/Manual/class-Cloth.html | https://docs.unity3d.com/ScriptReference/Cloth.html | https://creators.vrchat.com/worlds/whitelisted-world-components/"
updated: 2026-08-19
---
# Cloth — VRChat 原生 Unity 物理布料组件

> **白名单地位**: Unity `Cloth` 在官方 [Allowlisted World Components](../whitelisted-world-components.md)「Unity Components (77) / Physics & Colliders」内，**World 可用**。
> **VRChat 专属限制**: 无官方 VRChat 专页。VRChat 侧没有针对 Cloth 的专属限制说明 → 本页按 **Unity 默认行为**记录；若需 VRChat 侧边界，见「VRChat 特定注意」的 [UNKNOWN] 标注。

---

## 核心功能（官方，[FACT]）

`Cloth`（`UnityEngine.Cloth`）是与 **Skinned Mesh Renderer** 协同的物理布料模拟组件，专为角色衣物设计。官方语义来自 Unity Manual：[FACT] https://docs.unity3d.com/2022.3/Documentation/Manual/class-Cloth.html

- **挂载要求（官方，[FACT]）**：只能挂在与 **SkinnedMeshRenderer** 相关的 GameObject 上；若挂到普通 **MeshRenderer** 的物体上，Unity 会**移除 Mesh Renderer 并添加 Skinned Mesh Renderer**。参见 [[skinnedmeshrenderer.md|Skinned Mesh Renderer]] 与 [[meshrenderer.md|Mesh Renderer]]。
- **碰撞是单向的（[FACT]）**：Cloth 只响应被指定的 collider，不向世界回授力；且只支持 3 种 collider：**SphereCollider、CapsuleCollider**（含由两个球构建的锥形胶囊）。
- **顶点约束（Vertex Constraints，[FACT]）**：通过「Edit cloth constraints」工具对每个顶点设 **Max Distance**（粒子相对顶点位置的最大位移）与 **Surface Penetration**（粒子可渗入网格的深度）。
- **自碰撞与互碰撞（[FACT]）**：自碰撞（Self Collision）防布料自穿，互碰撞（Intercollision）让多块布料粒子彼此碰撞；两者都会显著增加模拟耗时。

## 关键属性/行为

| 属性 | 说明（[FACT] Unity Manual 2022.3） |
|---|---|
| **Stretching Stiffness** | 布料拉伸刚度。 |
| **Bending Stiffness** | 布料弯曲刚度。 |
| **Use Tethers** | 用约束防止运动粒子远离固定粒子，减少过度拉伸。 |
| **Use Gravity** | 是否对布料施加重力加速度。 |
| **Damping** | 运动阻尼系数。 |
| **External / Random Acceleration** | 恒定外部加速度 / 随机外部加速度。 |
| **World Velocity / Acceleration Scale** | 角色世界空间运动/加速度对布料顶点的传导比例。 |
| **Friction** | 布料与角色碰撞时的摩擦。 |
| **Collision Mass Scale** | 碰撞粒子的质量增量。 |
| **Use Continuous Collision / Use Virtual Particles** | 连续碰撞 / 每三角形一枚虚拟粒子，用于提升碰撞稳定性。 |
| **Solver Frequency** | 每秒求解迭代次数。 |
| **Sleep Threshold** | 布料休眠阈值。 |
| **Capsule / Sphere Colliders** | 本 Cloth 实例与之碰撞的胶囊/球体碰撞体数组。 |

## VRChat 特定注意

- **[FACT] 白名单地位**：Cloth 在官方白名单「Physics / Colliders」段内，World 可直接当 Unity 组件使用。
- **[FACT] 无 VRChat 官方专页**：官方文档没有为 Cloth 提供 World 侧专属说明。
- **[UNKNOWN]**：VRChat World 内挂载 Cloth 是否有数量上限、性能预算或与玩家/PhysBones 的具体交互边界——**官方未定论**，本页不臆测；建议在场景内先按 Unity 默认行为小规模实测。若无法从官方文档证实，一律不写具体数值。
- **性能定性（[INFERENCE]）**：布料是逐粒子物理求解，顶点/碰撞体越多成本越高，且碰撞体只能为球/胶囊（性能取向的官方设计）；但无官方/基准定量，不得据此估算具体开销。

## 相关页面

[[skinnedmeshrenderer.md|Skinned Mesh Renderer]] · [[meshrenderer.md|Mesh Renderer]] · [[capsulecollider.md|Capsule Collider]] · [[spherecollider.md|Sphere Collider]] · [[../whitelisted-world-components.md|Allowlisted World Components]] · [[index.md|Scene Components 总览]]

## 源清单

- Unity Manual Cloth: https://docs.unity3d.com/2022.3/Documentation/Manual/class-Cloth.html
- Unity Scripting API Cloth: https://docs.unity3d.com/ScriptReference/Cloth.html
- VRChat Allowlist: https://creators.vrchat.com/worlds/whitelisted-world-components/
