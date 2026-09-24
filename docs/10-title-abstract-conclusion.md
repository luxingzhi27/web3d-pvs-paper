# 10 — 标题、摘要与结论规划（降低训练机制在主叙事中的权重）

## 标题候选

### 当前首选

**Pre-Geometry Visibility for Progressive Web3D via Geometry-Compiled Occlusion Fields**

为什么更合适：

- headline 是 research operating point；
- 第二部分是核心 method insight；
- 没把某个训练技巧写进标题；
- 不提前过度承诺 generalization。

### 方法名版本

**GCOF-PVS: Geometry-Compiled Occlusion Fields for Pre-Geometry Visibility in Progressive Web3D**

如果后续希望建立方法名，可以使用。

---

# Abstract 的推荐逻辑

参考顶会图形学论文，不要把摘要写成所有实现组件的压缩版。

## 1. 背景 / Task

From-region visibility 对 progressive / streaming rendering 很有价值。

## 2. Gap

当前 PVS 方法通常假设 visibility computation 可以访问一份 scene representation；但在 progressive Web delivery 中，detailed geometry 可能正是尚未 resident 的内容。

## 3. Key Idea

提出：

> geometry-compiled pre-geometry visibility。

把 detailed scene geometry 离线编译成 compact client-queryable visibility representation。

## 4. Method Overview

一句话：

> local geometry + geometry-only occlusion context → compact directional field → lightweight region query.

不要塞所有维度。

## 5. Conservative PVS

只用一句：

> The model is trained with a safety-oriented objective that reflects the asymmetric cost of missing visible content.

除非 shared-zero-boundary 最终成为非常强的结果，否则 Abstract 不需要解释 fixed zero / calibration / SmoothMax。

## 6. Results

最终只写：

- safety；
- culling efficiency；
- asset / runtime cost；
- streaming utility；
- unseen-scene transfer（如果 formal evidence 足够）。

---

# Conclusion 的逻辑

回到最初问题：

> Progressive delivery needs relevance before detailed content residency.

核心 takeaway：

```text
scene geometry
    ↓ offline
compact visibility representation
    ↓ client query
occlusion-aware relevance
    ↓
progressive delivery
```

训练 objective、网络层数、anchor 数都不要在结论里重复。

如果最终 system evidence 足够强，可以用：

> Visibility can itself be treated as a compact streamable scene representation.

作为最后一句。

这句话比：

> “our robust zero-boundary objective works well”

更适合整篇论文的最终 takeaway。
