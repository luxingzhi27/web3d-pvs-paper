# 09 — Figures & Tables 规划

原则：

> **每一张图、每一张表都必须回答一个 reviewer 问题。**

如果只是展示实现细节，却不支撑任何论文 claim，应放 supplementary。

---

# Figure 1 — Problem Teaser

## 要证明什么

> Web streaming 需要在 detailed geometry residency 之前获得 visibility relevance。

建议画：

- Server 上的大型 GLB resources；
- 一个很小的 visibility asset；
- Browser 当前 view-cell；
- 一个“在 frustum 内但实际被挡住”的资源；
- visibility-aware download priority；
- useful geometry 更早到达。

不要在 Figure 1 里塞完整网络结构。

---

# Figure 2 — 不同方法处在 Content-Residency Pipeline 的哪个位置

可以画三列：

```text
Conventional Online PVS
resident scene representation
        ↓
     visibility
        ↓
      render
```

```text
NeuralPVS
runtime geometry representation
        ↓
    neural PVS
        ↓
      render
```

```text
GCOF-PVS
compact compiled asset
        ↓
     visibility
        ↓
     download
        ↓
     geometry
        ↓
      render
```

这张图主要用于：

> 把我们的 operating point 与 NeuralPVS / online PVS 一眼区分开。

---

# Figure 3 — GCOF Architecture

必须明确标出：

## OFFLINE / CONTENT PREPARATION

- 256-point local surface encoder；
- AABB relation graph；
- 12 anchors × top-8；
- attention relation compiler；
- 12×7 anchor responses；
- fixed first-order directional projection；
- 4×7 compact field。

## TRANSFER BOUNDARY

明确画一条线：

> 哪些东西传到 Web client，哪些不传。

## WEB RUNTIME

- view-cell；
- 9 support points；
- analytic field query；
- center/max/mean/min；
- 16D query geometry；
- tiny visibility head。

---

# Figure 4 — Structured Survival Field Intuition

选一个 target。

画：

- 几个不同方向；
- survival vs normalized distance curve；
- (S(0)=1)；
- monotonicity。

这张图用于解释：

> 为什么这不是普通 28D latent。

---

# Figure 5 — Shared Zero Boundary

这是非常值得做成主图的一张。

每个 scene：

- visible score histogram / KDE；
- invisible score histogram / KDE；
- (z=0) 垂直线。

比较：

- Full；
- PBCE。

它直接回答：

> robust-boundary training 是否真的让零边界跨场景对齐。

---

# Figure 6 — Safety–Efficiency Frontier

横轴：

- Useful Cull；
- 或 CNOR。

纵轴：

- weighted recall；
- 或 miss risk。

展示：

- Full；
- Geometry Field；
- Generic-28；
- PBCE；
- 其他 baseline。

作用：

> 不用一个单指标掩盖 conservative trade-off。

---

# Figure 7 — Asset / Runtime Scaling

建议两张独立图。

### 图 A

横轴：

- number of units。

纵轴：

- visibility asset MiB / bytes。

### 图 B

横轴：

- number of queried candidates。

纵轴：

- latency ms。

分别报告：

- WebGPU；
- WASM。

等 V5 runtime 真正实现后再冻结。

---

# Figure 8 — Progressive Streaming Utility

横轴：

- Downloaded MiB。

纵轴：

- Visible-weight coverage。

方法：

- V5 (p_g)；
- V5 (p_g/B^\alpha)；
- AABB；
- HZB visible-first；
- projected area / byte；
- oracle。

这张图直接回答：

> 同样下载 X MiB，谁更快得到正确可见内容？

---

# Main Tables

## Table 1 — Related Work Operating Point

推荐列：

- method；
- from-region；
- runtime scene representation required；
- learned；
- pre-content client query；
- streaming use。

这张表不能写成“我们全是勾，别人全是叉”的宣传表。

应该保持事实性。

---

## Table 2 — Scene / Dataset Summary

列：

- scene；
- domain type；
- unit count；
- resource / GLB count；
- view-cell type；
- split size；
- 在 shared / LOSO / blind holdout 中的角色。

---

## Table 3 — Fixed-Zero Main Result

行：

- Full；
- Geometry Field；
- Generic-28；
- PBCE。

列重点：

- worst-scene WR；
- worst-scene LCB；
- scene-equal WR；
- CNOR；
- Useful Cull；
- Bad Cull；
- PR-AUC lift。

---

## Table 4 — Generalization

分开：

- shared known scenes；
- LOSO；
- external blind holdout。

一定要有一列：

> 是否使用 target labels / calibration。

否则 reviewer 很难判断真正的 zero-shot 程度。

---

## Table 5 — Runtime Asset

列：

- method；
- startup bytes；
- bytes / unit；
- WebGPU p50/p95；
- WASM p50/p95；
- candidate count。

---

## Table 6 — Streaming

列：

- Bytes@95；
- Bytes@99；
- Bytes@99.9；
- Waste@99；
- 固定带宽时间。

---

## Table 7 — Scheduler Replay

列：

- p50；
- p95；
- downloaded bytes；
- waste；
- startup-asset overhead。

---

# 图表规划的总体原则

每一张图都先写一句：

> **This figure is intended to prove ______.**

如果这句话写不出来，这张图大概率不应该进正文。
