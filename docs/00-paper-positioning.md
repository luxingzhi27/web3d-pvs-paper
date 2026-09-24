# 00 — 论文定位（Paper Positioning）

## 一句话研究问题

> **在传统可见性算法所需的详细场景几何尚未下载到 Web 客户端之前，客户端如何估计保守的 from-region 可见性？**

更精确地说：

> **Can a Web client predict occlusion-aware from-region visibility using only compact pre-transmitted scene descriptors, before the detailed geometry required by conventional visibility algorithms is resident?**

## 核心问题

大型 Web3D 场景无法在交互开始前一次性传完，因此浏览器必须持续回答：

> **下一批最值得下载的几何是什么？**

在详细 tile / GLB 尚未到达时，客户端通常可以提前获得并使用：

- frustum；
- distance；
- hierarchy；
- geometric error / SSE；
- cache state。

但这些信息基本不包含“遮挡”这一维。

于是出现一个天然的循环依赖：

```text
为了决定哪些 geometry 值得下载，需要 visibility
                    ↑
                    │
传统 visibility 又往往依赖已经可用的 scene geometry
```

本文将这一问题称为：

> **Pre-Geometry Visibility（几何下载前可见性）**

## 这篇论文不应该被写成什么

不要把论文主线写成：

- “一个神经遮挡剔除网络”；
- “一个更快的 NeuralPVS”；
- “一个新的下载调度器”；
- “一个 PointNet/GNN 架构”；
- “一个 WebGPU 优化工作”。

这些都只是组件或应用结果。

## 最强的论文身份

> **一种面向 pre-geometry Web streaming 的紧凑 geometry-compiled visibility representation，并通过跨场景共享的保守决策边界使其可以直接部署。**

技术主体实际上有两个：

1. **GCOF 表征**：把局部几何和纯几何遮挡上下文离线编译成紧凑、可连续查询的结构化方向场；
2. **Shared conservative boundary learning**：使相同的零 logit 决策边界在不同场景/风险域上保持保守安全语义。

Web streaming 是最能说明这两个性质价值的系统场景。

## Novelty 边界

不要声称以下事情本身是新的：

- visibility-guided streaming；
- neural visibility；
- from-region PVS；
- 基于 lightweight metadata 的 pre-content selection。

已有工作分别覆盖了这些方向。

真正要强调的是它们的交叉：

```text
learned visibility
+ from-region query
+ geometry-only scene compilation
+ detailed geometry residency 之前的 client-side query
+ shared conservative boundary
+ progressive Web delivery
```

## 建议的贡献层级

### Contribution A — 问题定义

提出并系统研究 **pre-geometry from-region visibility**。

### Contribution B — 表征

把 geometry-only local descriptor 与 potential-occluder relations 编译成一个紧凑的 4×7 structured directional survival field。

### Contribution C — 学习目标

设计共享的零决策边界，并通过 domain-robust safety constraint 使其在多个场景/结构域上保持保守。

### Contribution D — 系统验证

将该信号接入资源级 progressive streaming，并从 safety、culling efficiency、资产大小、runtime、streaming utility 等角度验证。

## 整篇论文必须回答的 reviewer 问题

> **为什么客户端要先下载这个 visibility asset，而不是直接下载 coarse/proxy geometry 再运行 HZB 或其他 conventional PVS？**

因此最终实验必须在尽可能公平的前提下比较：

- startup bytes；
- runtime cost；
- candidate-count scaling；
- safety；
- useful culling；
- progressive streaming benefit。
