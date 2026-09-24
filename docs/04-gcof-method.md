# 04 — Geometry-Compiled Occlusion Fields（GCOF 方法）

## 方法概览

GCOF-PVS 将每个 renderable unit 的局部几何和 surrounding occlusion context 离线编译成一个紧凑、连续可查询的方向场。运行时，客户端只需针对当前 horizontal-disk view-cell 查询该场，并结合当前 query geometry 输出 unit-level visibility score。

整体流程为：

~~~text
local unit geometry
+ potential-occluder context
        ↓
relation compiler
        ↓
directional survival field
        ↓
analytic view-cell query
        ↓
regional occlusion statistics
        +
query geometry
        ↓
visibility head
        ↓
unit visibility score
~~~

---

## 4.1 Local Surface Geometry Encoding

对每个 renderable unit 固定采样 256 个表面点。每个点包含归一化局部坐标和法向：

$$
(x,y,z,n_x,n_y,n_z).
$$

共享编码器为：

~~~text
6 → 32 → 64 → 64
max/mean pooling + 3 size ratios
131 → 64 → 32
~~~

得到 unit descriptor：

$$
z_i\in\mathbb R^{32}.
$$

该 descriptor 表示 unit 自身的局部几何特征，并在所有场景和 units 之间共享编码器参数。

---

## 4.2 Potential-Occluder Relations

局部几何只能描述 target unit 自身，无法表达周围哪些物体可能形成遮挡。因此，对每个 target unit $u_i$，在 12 个固定正二十面体方向上构造 potential-occluder relations。

对于方向 $a_k$，将所有 unit AABB 投影到垂直于该方向的二维平面，并结合：

- projected overlap；
- relative depth；
- target/source scale；
- center distance；

选择最多 top-8 potential occluders。

每条 relation 使用 8D target-relative feature：

1. relative direction（3D）；
2. normalized log distance；
3. log radius ratio；
4. target projected-overlap ratio；
5. source projected-overlap ratio；
6. normalized log depth gap。

这 12 个 relation anchors 仅用于模型内部的方向表示，与离线数据协议中的 12 个 yaw/pitch camera directions 无关。

---

## 4.3 Relation Compiler

对 target $u_i$、source $u_j$ 和 relation feature $e_{ijk}$，首先构造 edge message：

$$
m_{ijk}
=
\operatorname{MLP}
\left(
z_i,z_j,e_{ijk}
\right).
$$

在同一 target-anchor group 中，通过 attention 聚合最多 8 个 potential occluders：

$$
h_{ik}
=
\sum_j
\alpha_{ijk}m_{ijk}.
$$

同时保留 neighbor count 与 projected-overlap sum，用于描述遮挡证据的总量。

随后，每个方向产生一组 7D response：

$$
q_{ik}
=
q^{\mathrm{base}}_{ik}
+
\mathbf 1[n_{ik}>0]\Delta q_{ik},
$$

其中 base term 由 target geometry 与方向生成，relation delta 根据 surrounding occluders 对其进行修正。

因此每个 unit 得到：

$$
Q_i\in\mathbb R^{12\times7}.
$$

---

## 4.4 从 Directional Responses 到 Continuous Field

12 组 anchor responses 只定义在离散方向上，而运行时 camera direction 是连续的。为获得可连续查询的方向表示，使用一阶方向 basis：

$$
\phi(d)
=
[1,d_x,d_y,d_z].
$$

对 12 个固定 anchor directions 组成 basis matrix：

$$
A
=
\begin{bmatrix}
1&d_{1x}&d_{1y}&d_{1z}\\
\vdots&\vdots&\vdots&\vdots\\
1&d_{12x}&d_{12y}&d_{12z}
\end{bmatrix}
\in\mathbb R^{12\times4}.
$$

使用固定 Moore–Penrose pseudoinverse 投影：

$$
C_i
=
A^{+}Q_i,
$$

得到：

$$
C_i\in\mathbb R^{4\times7}.
$$

因此，对任意查询方向 $d$，都可以连续得到 7D directional parameters：

$$
r_i(d)
=
\phi(d)C_i
\in\mathbb R^7.
$$

这一过程把原始的 $12\times7=84$ 个方向 responses 压缩为 $4\times7=28$ 个 field coefficients，同时保留连续方向查询能力。

---

## 4.5 Directional Survival Field

### 4.5.1 直观含义

为了描述 unit 与观察位置之间的遮挡关系，可以把“沿某个方向遇到有效遮挡”看作一个随距离发生的事件。

令：

$$
T_i(d)
$$

表示从 unit $u_i$ 沿方向 $d$ 前进时，发生有效遮挡的距离。对应的 survival quantity 可以写成：

$$
S_i(d,t)
=
P(T_i(d)>t).
$$

其直观含义是：

> 从 target 沿方向 $d$ 前进到距离 $t$ 时，遮挡事件尚未发生的程度。

在本文模型中，$S_i$ 用作**结构化的中间 occlusion statistic**，而不是经过概率校准的物理 visibility probability。

### 4.5.2 七参数形式

对任意方向 $d$，field 输出 7 个 raw parameters，经变换后得到：

- no-hit mass $p_\infty$；
- 两个 mixture weights $\pi_1,\pi_2$；
- 两个 positive locations $\mu_1,\mu_2$；
- 两个 positive scales $s_1,s_2$。

其中：

$$
p_\infty=\sigma(r_0),
$$

$$
(\pi_1,\pi_2)
=
\operatorname{softmax}(r_1,r_2),
$$

$$
\mu_j
=
\operatorname{softplus}(r_{\mu_j}),
$$

$$
s_j
=
0.02+\operatorname{softplus}(r_{s_j}).
$$

location $\mu_j$ 控制 survival curve 主要下降的位置，scale $s_j$ 控制下降的平缓程度；两个 mixture components 提供比单一 transition 更灵活的距离结构。

### 4.5.3 归一化距离

世界空间距离通过 target radius $r_i$ 归一化：

$$
t
=
\log\left(
1+\frac{d}{r_i}
\right).
$$

这样相同的世界距离会根据 target 的物理尺度具有不同意义，同时对较大的距离范围进行压缩。

### 4.5.4 Monotone Survival

完整 survival function 为：

$$
S(t)
=
p_\infty
+
(1-p_\infty)
\sum_{j=1}^{2}
\pi_j S_j(t).
$$

各 component 被归一化为：

$$
S_j(0)=1,
$$

因此：

$$
S(0)=1.
$$

同时固定方向上满足：

$$
d_2>d_1
\quad\Rightarrow\quad
S(d_2)\le S(d_1).
$$

因此该 representation 注入了三个重要结构先验：

1. **directional continuity**：相邻方向共享低阶方向函数；
2. **distance monotonicity**：沿固定方向增加查询距离时 survival 不增加；
3. **scale awareness**：距离相对于 target radius 归一化。

这些性质是 structured survival field 与 unrestricted 28D latent 的主要区别。

---

## 4.6 Horizontal-Disk View-Cell Query

论文统一采用 horizontal-disk view-cell。

离线区域 GT 使用 32 个 Color-ID camera positions，但运行时不会重放这 32 个视点。一次 V5 query 仅在 structured field 上使用 9 个 analytic support points：

- disk center；
- 8 个等角圆周点。

对于第 $k$ 个 support point $x_k$，相对于 target center $c_i$：

$$
d_{ik}
=
\frac{x_k-c_i}
{\|x_k-c_i\|},
$$

$$
\ell_{ik}
=
\|x_k-c_i\|.
$$

然后查询：

$$
S_{ik}
=
S_i(d_{ik},\ell_{ik}).
$$

9 个 survival values 被汇总成：

$$
[
S_{\mathrm{center}},
S_{\max},
S_{\mathrm{mean}},
S_{\min}
].
$$

其中：

- $S_{\mathrm{center}}$ 描述 view-cell 中心位置；
- $S_{\max}$ 表示区域中最强的 unoccluded evidence；
- $S_{\mathrm{mean}}$ 描述整体区域状态；
- $S_{\min}$ 描述最强遮挡位置。

这四个量只作为 regional occlusion features，不直接作为最终 PVS prediction。

统一语义为：

~~~text
32 = offline GT camera samples
 9 = one V5 query's analytic field supports
 1 = one batched runtime model query
~~~

---

## 4.7 Final Visibility Prediction

survival field 只描述从 target 到当前观察区域的结构化遮挡信息，最终 visibility 仍然取决于：

- target local geometry；
- regional survival statistics；
- camera / query geometry。

因此最终 head 输入为：

$$
z_i
\oplus
s_i^{\mathrm{region}}
\oplus
q_i^{\mathrm{query}},
$$

其中：

$$
z_i\in\mathbb R^{32},
$$

$$
s_i^{\mathrm{region}}\in\mathbb R^4.
$$

当前实现最终输入维度为 52D，并通过轻量 MLP 输出 visibility logit：

$$
\ell_i^{\mathrm{vis}}.
$$

因此应明确区分：

$$
\boxed{
\text{survival field}
\neq
\text{final visibility probability}
}
$$

而是：

$$
\boxed{
\text{survival field}
=
\text{structured directional occlusion representation}
}
$$

最终 PVS prediction 由 local geometry、occlusion statistics 和 query geometry 联合决定。

---

## 4.8 End-to-End Learning

当前正式 V5 不对 survival field 单独施加 field-NLL 或 blocker-distance supervision。

因此 field parameters 由最终 PVS objective 端到端学习：

~~~text
visibility objective
        ↓
visibility head
        ↓
regional survival statistics
        ↓
survival field
        ↓
relation compiler
~~~

这意味着 survival field 应理解为：

> 具有明确数学结构和几何先验的 learned latent representation。

而不是对真实物理遮挡概率的直接监督拟合。

---

## 4.9 Representation Ablations

### Geometry Field

GEOMETRY_FIELD 去除 surrounding potential-occluder relations，但保留 structured field 和相同 query mechanism，用于验证：

> surrounding occlusion context 是否必要。

### Generic Relation 28

GENERIC_RELATION_28 保留 relation evidence，并使用同样的 28D runtime budget，但用 unrestricted 28D latent 替代 structured survival field，用于验证：

> directional continuity、distance monotonicity 和 analytic query structure 是否提供了超出普通 latent 的价值。
