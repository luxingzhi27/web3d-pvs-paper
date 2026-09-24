# 09 — Figures & Tables 规划（按图形学论文视觉叙事调整）

原则：

> 图形学顶会/TOG 的前几张图通常承担“问题 + 核心 insight + 方法行为”的解释，而不是把论文写作逻辑做成流程框图。

---

# Figure 1 — Teaser：问题、方法行为和最终收益放在一起

之前纯粹画：

```text
Server → visibility asset → browser → scheduler
```

过于概念化，不够像图形学论文 teaser。

更推荐：

## 左侧：同一个 view-region

展示一个有明显遮挡关系的 Web3D scene：

- 当前 camera/view-cell；
- foreground occluder；
- behind-occluder geometry；
- true potentially visible geometry。

## 中间：baseline pre-download selection

例如：

- frustum / projected-size / distance；

把一些 occluded resources 标记为高优先级。

## 右侧：GCOF-PVS

展示：

- compact visibility asset；
- predicted relevance；
- visible resources 更早到达；
- occluded resources 被延后。

如果最终有 streaming data，可以在 Figure 1 下面直接放：

- same downloaded-byte budget；
- baseline rendering；
- ours rendering。

这样 teaser 同时回答：

> 这是个什么问题？  
> 我们的方法改变了什么？  
> 有什么视觉/系统收益？

这更接近 Trim Regions / NeuralPVS 的 Figure 1 风格。

---

# Figure 2 — Method Overview

Figure 2 再画完整 pipeline：

```text
OFFLINE
local geometry + proxy occlusion relations
            ↓
shared geometry compiler
            ↓
compact directional field
================ transfer boundary ================
WEB RUNTIME
view region
    ↓
analytic field query
    ↓
visibility score
    ↓
resource priority
```

不要在 Figure 2 同时塞所有 layer dimensions。

目标是让 reviewer 30 秒理解：

> 哪些 computation offline，哪些 runtime，详细 geometry 在哪里消失。

---

# Figure 3 — GCOF Representation Detail

这里才详细画：

- 256-point local surface；
- 12 anchor directions；
- top-K potential occluders；
- relation aggregation；
- anchor responses；
- fixed directional projection；
- 4×7 field。

这张图回答：

> compact field 到底如何从 geometry-only context 编译出来？

---

# Figure 4 — Region Query / Structured Field Intuition

展示：

- 一个 target；
- 几个 directional survival curves；
- V5 单次 query 内部的 9 个 analytic field support points；
- center / max / mean / min。

作用：

> 解释 structured field 为什么不是普通 latent。

---

# Figure 5 — Shared Boundary（只有结果够强才进正文）

这是 training result figure，不应该排在很前。

每个 scene：

- visible score distribution；
- invisible score distribution；
- $z=0$。

比较：

- Full；
- PBCE。

如果它非常有说服力，放 Evaluation。

如果只是辅助训练分析，移 supplementary。

---

# Figure 6 — Safety–Efficiency Frontier

横轴：

- Useful Cull / CNOR。

纵轴：

- Weighted Recall / Bad Cull。

回答：

> 在 conservative constraint 下，各方法能剔除多少真正冗余 geometry？

---

# Figure 7 — Asset / Runtime Cost

分别画：

- asset bytes vs number of units；
- latency vs candidate count。

这是回答：

> 为什么不传 proxy geometry 直接跑 HZB？

的重要图。

---

# Figure 8 — Progressive Streaming Utility

横轴：

- delivered unit payload / time。

纵轴：

- visible-weight coverage。

比较：

- baseline priority；
- projected area / byte；
- AABB；
- HZB；
- GCOF；
- GCOF / byte；
- oracle。

---

# Main Tables

## Table 1 — Related Work / Operating Point

不要做“我们全勾、别人全叉”的营销表。

建议客观列：

- Method
- From-region?
- Runtime scene representation
- Scene-specific preprocessing
- Learned?
- Client pre-content query?
- Streaming application

---

## Table 2 — Dataset / Scene Summary

- scene；
- units；
- renderable units；
- horizontal-disk radius；
- region-center / direction protocol；
- offline GT positions；
- role。

---

## Table 3 — Main Visibility Result

把 safety 放前面：

- WR；
- LCB；
- Bad Cull；
- Useful Cull；
- CNOR。

然后才：

- PR-AUC；
- accuracy 等。

---

## Table 4 — Representation / Training Ablations

不要把所有 ablation 混在 Main Table。

可以分别回答：

- context necessary? Full vs Geometry Field；
- structure necessary? Full vs Generic-28；
- objective necessary? Full vs PBCE。

---

## Table 5 — Runtime / Asset Cost

这是 system claim 的关键。

---

## Table 6 — Progressive Streaming

Unit delivery Cost/Bytes@95 / 99 / waste / bandwidth time。

---

# 总体规则

前 3 张图优先解释：

1. 问题和视觉/系统收益；
2. offline vs runtime 的核心 pipeline；
3. compact representation 的关键机制。

训练机制相关图应该后移到 Evaluation，而不是主导论文前半部分。
