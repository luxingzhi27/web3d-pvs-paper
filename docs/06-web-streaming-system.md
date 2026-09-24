# 06 — Progressive Web3D Integration（统一为 Unit-Level Visibility）

## 本节目标

这一节只回答：

> **unit-level regional visibility prediction 如何在浏览器端参与剔除与渐进式内容选择？**

论文核心方法始终以 renderable unit 为统一对象：visibility prediction、regional PVS、culling 与 progressive ordering 都在 unit level 定义。

---

# 6.1 Offline Scene Compilation

对一个新场景：

```text
scene geometry
   ├─ local surface sampling
   │      ↓
   │     z_i
   │
   └─ unit AABBs
          ↓
   geometry-only proxy relation graph
          ↓
   frozen shared compiler
          ↓
      field C_i
          ↓
 compact visibility asset
```

核心 deployment property：

> visibility asset 由 geometry-only preprocessing + frozen shared model 得到，不需要 target-scene visibility labels，也不需要 target-specific fine-tuning。

---

# 6.2 Runtime View-Cell Establishment

前端输入：

- current camera position；
- current camera orientation；
- 场景登记的 horizontal-disk radius；
- camera projection parameters。

以当前相机建立 view-cell anchor。

论文统一采用：

> **固定方向 + 水平圆盘**。

只要相机仍位于该圆盘内，且方向与投影参数仍满足当前 region contract，可以复用该次 regional prediction。

当：

- 相机走出圆盘；
- orientation 改变到需要重新建立 region；
- projection contract 改变；

则重新 query。

---

# 6.3 Candidate Generation

从当前 view-cell center 沿当前观察方向后退：

[
\Delta
=
\frac{r}{\tan30^circ}.
]

在后退位置建立：

[
66^circ
]

candidate frustum，并对 unit AABB 做 frustum test。

得到 candidate units：

[
\mathcal C(\mathcal B).
]

candidate generation 与 model prediction 分开：

> 模型只对 candidate units 做一次 batched query。

---

# 6.4 One Batched Regional Visibility Query

前端不会：

- 渲染离线 32 个 GT cameras；
- 为 9 个 V5 support points 分别运行完整网络。

实际 runtime 是：

```text
candidate units
    ↓
one batched V5 query
    ↓
per-unit visibility scores
```

V5 内部的 9 个 points 只是：

> 对 structured field 的 analytic support evaluations。

因此：

```text
32 = offline GT rendering samples
 9 = V5 internal analytic supports
 1 = runtime batched model query
```

---

# 6.5 Regional PVS 与 Instantaneous Rendering

模型预测的是：

> 当前 horizontal-disk view-cell 的 regional PVS / visibility scores。

使用冻结 decision rule 后得到 predicted potentially visible units：

[
\widehat{\mathrm{PVS}}(\mathcal B).
]

然后使用当前真实：

[
60^circ
]

display frustum 进一步过滤当前帧需要处理的 units。

因此 runtime 语义是：

```text
66° regional candidate / PVS
        ↓
predicted potentially visible units
        ↓
current real 60° display frustum
        ↓
instantaneous units to render/process
```

这里要强调：

> regional visibility 与 current-frame visibility 不是同一个集合。

regional PVS 是为了在一段局部相机运动范围内保持保守与可复用。

---

# 6.6 用于 Progressive Content Selection

每个 candidate unit 得到一个 visibility score：

[
s_i.
]

该信号可以用于：

### Conservative culling / defer

根据冻结边界判断：

- keep；
- defer / cull。

### Progressive ordering

在 candidate units 中优先处理：

> visibility relevance 更高的单位。

论文的核心结论只需要证明：

> **pre-geometry unit-level visibility signal 能改善 progressive delivery。**

至于具体工程系统最终如何把 units 打包成文件，不属于方法定义。

---

# 6.7 与现代 Web Selection 的关系

GCOF-PVS 不替代：

- frustum culling；
- spatial hierarchy；
- SSE / geometric error；
- cache policy；
- streaming scheduler。

它补充的是：

> **occlusion-aware unit relevance before detailed content residency**

更合理的系统关系是：

```text
camera
+ hierarchy / frustum / coarse metadata
        ↓
candidate units
        ↓
GCOF-PVS
        ↓
unit-level occlusion-aware relevance
        ↓
culling / progressive ordering
```

因此本文的系统贡献应表述为：

> 为 Web progressive rendering 提供一个 unit-level、pre-geometry、from-region visibility signal。

而不是：

> 设计一个新的文件级调度框架。
