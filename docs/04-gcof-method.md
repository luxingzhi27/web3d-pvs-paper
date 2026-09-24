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

$$
z_i\in\mathbb R^{32}.
$$

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

这里必须与数据协议严格区分：

> **模型内部的 12 个正二十面体 relation anchors，不是离线 GT 协议中的 12 个 yaw/pitch 相机方向。**

两者都恰好是 12，但用途完全不同：

- 离线 12 directions：训练/评价的 camera-direction coverage；
- relation 12 anchors：geometry-only occlusion context 的模型内部方向基。
- 根据 AABB orthographic overlap 和 depth ordering 构造；
- 不读取 visibility labels。

relation artifact 必须显式声明：

```text
usesVisibilityLabels = false
```

8D edge feature：

1–3. relative direction；  
4. $\log(1+d_{ij}/r_i)$；  
5. $\log(r_j/r_i)$；  
6. target projected-overlap ratio；  
7. source projected-overlap ratio；  
8. $\log(1+\mathrm{gap}/r_i)$。

核心解释：

> 一个 target 自身的局部形状无法知道“它周围有哪些可能挡住它的物体”；但 potential occlusion structure 可以在完全不读取 visibility label 的情况下从几何关系构造。

relation graph 不是 learned scene memory，而是：

> deterministic geometry proxy。

---

# 4.3 Single-Layer Relation Field Compiler

每条关系边：

$$
[z_i,z_j,e_{ij}]
\in\mathbb R^{72}
$$

经过：

$$
72\rightarrow64\rightarrow32.
$$

得到 edge message：

$$
m_{ijk}.
$$

在每个 target-anchor group 内做 attention aggregation：

$$
h_{ik}
=
\sum_j
\alpha_{ijk}m_{ijk}.
$$

同时显式保留：

- $\log(1+n_{ik})$；
- projected overlap sum。

为什么要保留这两个统计量：

> softmax attention 会把 neighborhood mass 归一化；如果只看 weighted average，一个强 occluder 与多个同等强度 occluder 的“遮挡证据总量”可能被抹平。

因此 count / overlap sum 恢复“遮挡证据有多少”的信息。

---

# 4.4 Anchor Responses 与固定方向投影

geometry-only base：

$$
[z_i,a_k]
:
35\rightarrow32\rightarrow7.
$$

relation delta：

$$
[h_{ik},
\log(1+n_{ik}),
o_{ik},
a_k]
:
37\rightarrow32\rightarrow7.
$$

最终：

$$
q_{ik}
=
q^{base}_{ik}
+
\mathbf1[n_{ik}>0]\Delta q_{ik}.
$$

12 个 anchor response：

$$
Q_i\in\mathbb R^{12\times7}.
$$

然后使用固定一阶方向基：

$$
[1,d_x,d_y,d_z]
$$

的 Moore–Penrose pseudoinverse 投影：

$$
Q_i
\rightarrow
C_i\in\mathbb R^{4\times7}.
$$

最终 runtime field 只有：

$$
4\times7=28
$$

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

$$
t=
\log(1+d/r_i).
$$

该 analytic transform 被构造为满足：

$$
S(0)=1
$$

并且固定方向上：

$$
d_2>d_1
\Rightarrow
S(d_2)\le S(d_1).
$$

因此它具有明确的 monotone inductive bias。

但论文里一定要强调：

> (S) 是 structured intermediate occlusion/survival statistic，不是真实 object visibility probability。

最终 PVS visibility 仍由后面的 visibility head 预测。

---

# 4.6 水平圆盘上的 Nine-Support Analytic Query

论文统一的 view-cell 是：

> **固定观察方向 + 世界 XZ 平面中的水平圆盘。**

离线 GT 使用 32 个真实 Color-ID camera positions，但 V5 runtime **不会重放这些相机**。

对于一次 V5 query，只在已经编译好的 structured field $C_i$ 上使用 9 个 analytic support points：

- 圆盘中心；
- 8 个等角圆周点。

同一个 field $C_i$ 在这 9 个位置上计算 survival statistics，并压缩为：

$$
[S_{center},S_{max},S_{mean},S_{min}].
$$

必须明确：

> 这 9 个点只是**一次模型查询内部的解析 field evaluation points**。

它们不是：

- 9 个 rendered cameras；
- 9 个独立 view-cells；
- 9 次完整 neural inference。

因此论文中统一记成：

```text
32 = offline regional GT camera samples
 9 = one V5 query's analytic field supports
 1 = one batched runtime model query
```

---

# 4.7 Disk-View Query Geometry 与 Visibility Head

论文方法统一以水平圆盘作为 view-cell 空间区域。

query geometry 用于描述：

- target 与当前 region center 的相对方向；
- 相机坐标系中的 target direction；
- target distance / scale；
- 圆盘半径相对于 target distance/size 的归一化关系；
- FOV；
- near / far projection parameters。

当前实现保留了一些通用 region packing 字段，但在论文统一协议中：

> region shape 固定为 horizontal disk，box-specific region-type / 3D half-axis 语义不作为方法定义展开。

最终 visibility head 接收：

- local geometry descriptor；
- 4D region survival statistics；
- compact query geometry；

并输出 unit-level visibility logit：

$$
z_i.
$$

论文正文可在 implementation table 中给出最终冻结维度，但不需要为了兼容历史 box code 把 box-view-cell 重新引入方法定义。

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
