# 04 — Geometry-Compiled Occlusion Fields（GCOF 方法）

## 本节目标

这一节必须解释清楚：

> V5 不是一个普通 instance classifier，而是先把 geometry-only scene context 编译成一个可传输、可连续查询的结构化 visibility representation，再进行轻量 region query。

整体链路：

```text
local target geometry
+ geometry-only occlusion context
    ↓
structured compact directional field
    ↓
analytic region query
    ↓
tiny visibility head
```

---

# 4.1 Local Surface Geometry Encoder

每个 renderable unit 输入：

- 256 个固定表面采样点；
- 每点 6D：normalized local xyz + normal；
- unit-level 3 个 size ratios。

当前网络：

```text
point MLP: 6 → 32 → 64 → 64, SiLU
pool: max(64) || mean(64) || size ratios(3)
unit MLP: 131 → 64 → 32, SiLU + Tanh
```

输出：

[
z_i\in\mathbb R^{32}.
]

这一节不要把 PointNet-like encoder 本身包装成创新。

真正要强调：

- 所有场景共享参数；
- 不使用 scene ID；
- 不使用 unit embedding；
- 不使用 per-instance learned residual；
- 不使用 scene-normalized world position；
- 新场景只需要 geometry preprocessing，不需要 target-scene visibility label。

这一设计是“scene-independent compilation”的第一块基础。

---

# 4.2 Geometry-Only Potential Occlusion Relations

当前 relation graph 固定为：

- 12 个正二十面体方向；
- 每个 target / anchor 最多 top-8 potential occluders；
- 根据 AABB orthographic overlap 和 depth ordering 构造；
- 不读取 visibility labels。

relation artifact 必须显式声明：

```text
usesVisibilityLabels = false
```

8D edge feature：

1–3. relative direction；  
4. (log(1+d_{ij}/r_i))；  
5. (log(r_j/r_i))；  
6. target projected-overlap ratio；  
7. source projected-overlap ratio；  
8. (log(1+mathrm{gap}/r_i))。

核心解释：

> 一个 target 自身的局部形状无法知道“它周围有哪些可能挡住它的物体”；但 potential occlusion structure 可以在完全不读取 visibility label 的情况下从几何关系构造。

relation graph 不是 learned scene memory，而是：

> deterministic geometry proxy。

---

# 4.3 Single-Layer Relation Field Compiler

每条关系边：

[
[z_i,z_j,e_{ij}]
\in\mathbb R^{72}
]

经过：

[
72\rightarrow64\rightarrow32.
]

得到 edge message：

[
m_{ijk}.
]

在每个 target-anchor group 内做 attention aggregation：

[
h_{ik}
=
\sum_j
\alpha_{ijk}m_{ijk}.
]

同时显式保留：

- (log(1+n_{ik}))；
- projected overlap sum。

为什么要保留这两个统计量：

> softmax attention 会把 neighborhood mass 归一化；如果只看 weighted average，一个强 occluder 与多个同等强度 occluder 的“遮挡证据总量”可能被抹平。

因此 count / overlap sum 恢复“遮挡证据有多少”的信息。

---

# 4.4 Anchor Responses 与固定方向投影

geometry-only base：

[
[z_i,a_k]
:
35\rightarrow32\rightarrow7.
]

relation delta：

[
[h_{ik},
\log(1+n_{ik}),
o_{ik},
a_k]
:
37\rightarrow32\rightarrow7.
]

最终：

[
q_{ik}
=
q^{base}_{ik}
+
\mathbf1[n_{ik}>0]\Delta q_{ik}.
]

12 个 anchor response：

[
Q_i\in\mathbb R^{12\times7}.
]

然后使用固定一阶方向基：

[
[1,d_x,d_y,d_z]
]

的 Moore–Penrose pseudoinverse 投影：

[
Q_i
\rightarrow
C_i\in\mathbb R^{4\times7}.
]

最终 runtime field 只有：

[
4\times7=28
]

个值。

重要点：

> projection matrix 是固定 buffer，不是 learned Parameter。

这使得结构化 field 与普通 latent 有明确区分。

---

# 4.5 Structured Analytic Survival Field

7 个方向参数最终对应：

- one no-hit mass；
- two mixture logits；
- two positive locations；
- two positive scales。

归一化距离：

[
t=
\log(1+d/r_i).
]

该 analytic transform 被构造为满足：

[
S(0)=1
]

并且固定方向上：

[
d_2>d_1
\Rightarrow
S(d_2)\le S(d_1).
]

因此它具有明确的 monotone inductive bias。

但论文里一定要强调：

> (S) 是 structured intermediate occlusion/survival statistic，不是真实 object visibility probability。

最终 PVS visibility 仍由后面的 visibility head 预测。

---

# 4.6 Nine-Support From-Region Query

disk view-cell：

- center；
- 8 个等角 ring points。

oriented-box view-cell：

- center；
- 8 corners。

同一个 (C_i) 在 9 个 support 上解析查询：

[
S_{ik}.
]

最终压成：

[
[
S_{center},
S_{max},
S_{mean},
S_{min}
].
]

需要强调：

> 这不是把一个完整 point-visibility neural model 跑 9 次。

而是：

[
\boxed{
\text{one compiled field}
+
\text{nine cheap analytic evaluations}
+
\text{one tiny head}
}
]

---

# 4.7 Query Geometry 与最终 Visibility Head

当前 query geometry 是 16D，主要包含：

- target → region center 的 world direction；
- region → target 在 camera right/up/forward 中的方向；
- normalized distance / radius；
- normalized region extents；
- FOV；
- region type；
- near / far normalized terms。

Full 输入：

[
32+4+16=52.
]

最终 head：

[
52\rightarrow32\rightarrow1.
]

输出 visibility logit：

[
z_i.
]

到这里，runtime inference method 才完整。

---

# 4.8 两个最关键的 Representation Controls

## Geometry Field

去掉 surrounding relation context，但：

- 保留 32D local geometry；
- 仍生成 4×7 structured field；
- 使用相同 9-point query；
- 使用相同 shared-boundary objective；
- network capacity 尽量匹配。

回答：

> **target 自身 geometry 是否足以推断遮挡？**

如果它很安全但 culling 差，说明：

> local shape 能提供 conservative prior，但 surrounding occlusion context 才带来真正有效的遮挡剔除能力。

## Generic Relation 28

保留：

- relation input；
- 12×7 relation evidence；
- 28D runtime budget；
- 同一 shared-boundary objective。

但把 structured analytic field 替换成 unrestricted 28D latent。

回答：

> **Full 的收益来自 relation information 本身，还是来自 structured monotone field representation？**

这是 V5 最关键的结构消融之一。
