# 10 — 标题、摘要与结论规划

## 标题候选

### 候选 1：问题优先

**Pre-Geometry Visibility for Progressive Web3D via Geometry-Compiled Occlusion Fields**

优点：

- 一眼说明 research problem；
- 不在标题里过度承诺 generalization；
- “Pre-Geometry Visibility”有机会成为论文自己的术语。

这是当前最推荐的方向。

---

### 候选 2：方法名优先

**GCOF-PVS: Geometry-Compiled Occlusion Fields for Pre-Geometry Visibility in Progressive Web3D**

优点：

- 建立 GCOF-PVS 方法名；
- 方便之后引用/复现。

缺点：

- 标题略长；
- 如果论文最终最强的是问题定义而非方法品牌，候选 1 更自然。

---

### 候选 3：更一般的 graphics 表述

**Geometry-Compiled From-Region Visibility Before Content Residency**

优点：

- 方法定位更 general。

缺点：

- Web3D / streaming motivation 弱化。

---

# Abstract 的六步逻辑

摘要不要按照：

> 网络 A → 模块 B → loss C → 实验 D

流水账写。

建议固定成六步。

## 1. Context

大型 Web3D 场景需要 progressive delivery。

## 2. Problem

现有 visibility algorithms 通常假设 scene representation 已经可访问；但 streaming client 恰恰希望在 detailed geometry residency 之前就知道哪些 geometry 值得优先请求。

## 3. Formulation

提出：

> pre-geometry from-region visibility。

## 4. Method

一句话概括：

> shared geometry encoder + geometry-only potential-occluder relations → compact structured directional field → lightweight region query。

不要在摘要里塞：

- 256；
- 12×8；
- 4×7；
- 52→32→1。

## 5. Training

强调：

> shared conservative zero boundary across heterogeneous domains。

这是 V5 与普通 classifier 很不一样的部分。

## 6. Evidence

摘要最后只写最终已经完成的 headline evidence：

- safety；
- useful culling；
- cross-scene transfer；
- runtime / asset cost；
- streaming benefit。

在实验没完成前不要预填数字。

---

# Conclusion 的逻辑

结论不要复述所有网络层。

应该重新回到最开始的系统矛盾：

> Conventional visibility becomes available after scene geometry is present, whereas progressive delivery benefits from visibility before that point.

然后用一句 pipeline 总结：

```text
geometry-only scene context
    ↓
compact compiled visibility asset
    ↓
shared conservative query
    ↓
progressive delivery
```

结尾可以考虑更 general 的一句：

> visibility can be treated as a streamable scene representation in its own right.

但只有在最终资产大小、runtime 和 streaming 实验都足够强时再保留。
