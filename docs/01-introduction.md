# 01 — Introduction 写作方案

## 本节目标

Introduction 的任务不是介绍网络，而是让读者接受：

> **“几何尚未到达客户端时的可见性”是一个独立且重要的图形学 / Web streaming 问题。**

## 推荐行文逻辑

### 第 1 段：Progressive Web3D 首先是“优先级问题”

从系统约束开始：

- 大规模 Web3D 场景无法在启动阶段一次性传完；
- 浏览器必须按需、渐进式获取资源；
- 因此系统必须持续回答：**what should be downloaded next?**

先列出现有 pre-download 信号：

- frustum；
- distance；
- hierarchy；
- geometric error / SSE；
- cache state。

不要从“occlusion culling 很重要”起笔。

### 第 2 段：这些 pre-download 信号缺少遮挡信息

用一个极简单的例子：

```text
camera → Building A → Building B
```

A、B 都可能：

- 在 frustum 内；
- 距离很近；
- projected size / SSE 很大；

但 B 完全被 A 遮挡。

因此：

> “几何是否值得优先下载”与“几何是否在视锥内”不是同一个问题。

### 第 3 段：提出 circular dependency

核心矛盾：

> Progressive streaming 希望在传输前知道 visibility；但传统 visibility computation 往往又依赖尚未传输到客户端的 scene representation。

在这里第一次定义：

> **pre-geometry visibility**

并明确区分：

- from-point runtime culling；
- from-region PVS；
- pre-geometry from-region visibility。

### 第 4 段：现有 PVS 为什么没有直接解决 bootstrap 问题

公平承认已有强方法：

- classical precomputed PVS；
- Camera Offset Space；
- Guided Visibility Sampling++；
- Trim Regions；
- Disocclusion Buffer；
- NeuralPVS。

不要写：

> existing PVS is too slow.

更准确：

> 这些方法通常假设执行 visibility 的一方已经拥有足够详细的 scene representation。

### 第 5 段：关键 insight

服务器/内容构建阶段本来就拥有完整几何。

因此问题不应该是：

> 浏览器能否为了 visibility 再恢复一份完整场景？

而应该是：

> 能否把 geometry-dependent occlusion knowledge 离线编译成更小、更适合网络传输的 visibility asset？

引出：

```text
scene geometry
   ↓ offline shared compiler
compact visibility asset
   ↓ transfer
browser region query
   ↓
visibility score
```

### 第 6 段：为什么 shared boundary 是方法的一部分

如果每个新场景都必须：

1. 收集该场景 visibility labels；
2. 重新 calibration threshold；

那么“geometry-only compile + zero-shot deployment”的故事会被削弱。

因此 V5 不只是学 ranking，而是希望：

[
z=0
]

在不同 source/domain 上具有一致的 conservative semantics。

这一点值得出现在 Introduction，而不是只藏在 Training Details。

### 第 7 段：Web integration

说明最终 visibility score 不会再训练一套独立“下载网络”。

而是：

- instance score；
- 聚合到 GLB/resource；
- 注入已有 progressive scheduler。

因此：

> 调度器不是主要 novelty，**pre-content visibility signal 的可获得性**才是。

### 第 8 段：贡献总结

建议只保留 3–4 点：

1. pre-geometry from-region visibility formulation；
2. geometry-compiled compact occlusion field；
3. shared conservative boundary learning；
4. Web3D streaming integration + evaluation。

## Figure 1 / Teaser

第一张图应该解释**问题和系统收益**，而不是画完整网络。

建议：

```text
Server                           Browser

detailed GLBs ───── deferred ───────┐
                                    │
compact visibility asset ───────────┤
                                    ↓
                              current view-cell
                                    ↓
                             visibility scores
                                    ↓
                       resource download priority
                                    ↓
                        useful geometry arrives first
```

至少画出一个：

- frustum 内；
- 但被前景建筑遮挡；

的资源，说明为什么传统 pre-download metadata 不够。

## Introduction 中不宜过早出现

不要在 Introduction 里塞：

- 256 点；
- 4×7；
- 12 anchors；
- top-8；
- SmoothMax 公式；
- 大量 metric 名称。

这些留到 Method / Evaluation。
