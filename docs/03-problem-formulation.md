# 03 — 问题定义（Problem Formulation）

## 本节目标

形式化本文真正研究的对象：

> **以 renderable unit 为基本剔除单位，在详细场景几何尚未 resident 于客户端之前，对一个水平圆盘 view-cell 预测 from-region potentially visible set。**

具体的数据采样、GT、candidate 与前端查询规则统一见：

- `docs/03a-unified-viewcell-protocol.md`

本文不再使用 oriented-box / cube view-cell，也不把底层文件打包形式纳入核心问题定义。

---

# 3.1 Renderable Units

定义场景中的可独立剔除单位：

[
\mathcal U
=
\{u_i\}_{i=1}^{N}.
]

每个 unit 至少具有：

- unique ID；
- world-space AABB；
- geometry；
- 方法所需的几何特征。

对于：

- BIM 场景：unit 为构件；
- 标准图形学场景：unit 由冻结的 deterministic partition rule 生成。

本文所有：

- visibility label；
- candidate；
- prediction；
- culling；

都统一在 unit level 定义。

---

# 3.2 View-Cell

一个 view-cell 由：

1. 一个合法 region center (c)；
2. 一个固定观察方向 (d)；
3. 一个场景登记的水平圆盘半径 (r)；
4. 固定 camera projection parameters；

共同定义。

空间区域为世界 (XZ) 平面中的水平圆盘：

[
\mathcal B(c,r)
=
\left\{
x:
\|(x-c)_{XZ}\|_2\le r,;
x_y=c_y
\right\}.
]

在一个 view-cell 内：

- camera position 可在圆盘内变化；
- camera orientation 固定；
- camera height 固定。

不同高度由不同 region centers 表示，而不是在一个 view-cell 内沿 (Y) 方向扩张。

---

# 3.3 离线方向覆盖

离线数据集在每个 physical region center 上展开：

## yaw

[
0^circ, 90^circ, 180^circ, 270^circ
]

## pitch

[
-15^circ, 0^circ, 15^circ.
]

因此每个 center 对应：

[
4\times3=12
]

个离线 view-cells。

这些方向只定义：

> 训练与评价的数据覆盖协议。

它们**不限制前端运行时相机方向**。

---

# 3.4 Sampled From-Region Ground Truth

对于一个 view-cell，在水平圆盘内冻结采样：

[
M=32
]

个 camera positions：

[
\{c_m\}_{m=1}^{32}.
]

其中包含圆盘中心，其余位置按冻结的面积均匀规则分布。

所有位置使用相同：

- orientation；
- camera height；
- (66^circ) FOV；
- projection contract。

设：

[
V(c_m)
\subseteq
\mathcal U
]

为第 (m) 个 Color-ID rendering 中的 visible units。

则 sampled regional PVS 定义为：

[
\mathrm{PVS}(\mathcal B)
=
\bigcup_{m=1}^{32}
V(c_m).
]

对应 unit label：

[
y_i(\mathcal B)
=
\mathbf 1
\left[
u_i
\in
\mathrm{PVS}(\mathcal B)
\right].
]

这表示：

> **区域内的 sampled potentially visible set**

而不是：

- center camera 的瞬时 visible set；
- 对连续圆盘所有相机位置的数学精确 PVS。

---

# 3.5 Visual Weight

若 (w_i(c_m)) 表示 unit (u_i) 在第 (m) 个 Color-ID sample 中的屏幕覆盖贡献，则区域权重定义为：

[
w_i(\mathcal B)
=
\max_{m=1,ldots,32}
w_i(c_m).
]

因此训练/评价中的 weighted recall 面向的是：

> unit 在整个 sampled view-cell 中可能达到的最大视觉贡献。

---

# 3.6 Candidate Units

candidate set 独立于 GT 构造。

从 region center 沿固定观察方向向后移动：

[
\Delta
=
\frac{r}{\tan30^circ}.
]

在后退位置建立 (66^circ) candidate frustum，并对 unit AABB 做 frustum test，得到：

[
\mathcal C(\mathcal B)
\subseteq
\mathcal U.
]

数据协议要求：

[
\mathrm{PVS}(\mathcal B)
\subseteq
\mathcal C(\mathcal B).
]

若 sampled GT 中存在 visible unit 不在 candidate set 中，则 candidate protocol 失败。

禁止：

> 将漏掉的 GT unit 人工补回 candidate set。

这一 inclusion test 只保证冻结的 32 个 sampled camera positions 被 candidate frustum 覆盖，不构成连续圆盘上的形式化 coverage proof。

---

# 3.7 Pre-Geometry Visibility Task

运行时客户端尚未拥有每个 unit 的 detailed geometry (G_i)。

客户端只拥有由服务器/离线阶段编译得到的紧凑表示：

[
a_i.
]

目标是在当前 view-cell 下预测：

[
f_\theta
(
a_i,
\mathcal B
)
\rightarrow
z_i,
qquad
u_i\in\mathcal C(\mathcal B).
]

其中：

- (z_i) 是 unit-level visibility score / logit；
- 预测只对 candidate units 执行。

设计目标：

[
|a_i|
\ll
|G_i|.
]

---

# 3.8 Conservative Asymmetry

false negative：

- potentially visible unit 被错误剔除；
- 可能导致 missing geometry、pop-in 或延迟正确画面。

false positive：

- 实际不可见 unit 被额外保留。

因此：

[
C_{FN}
\gg
C_{FP}.
]

这构成 safety-first learning 与 evaluation 的基本动机。

---

# 3.9 Runtime Regional Query

前端运行时并不重放离线 32 个 sampled cameras。

流程统一为：

```text
current camera position + orientation
        ↓
establish current horizontal-disk view-cell
        ↓
backward-shifted 66° AABB frustum
        ↓
candidate units
        ↓
one batched model query
        ↓
regional PVS prediction
        ↓
current real 60° frustum
        ↓
instantaneous units to render/process
```

只要相机仍处于当前圆盘内，并保持该 view-cell 的方向和 projection contract，可复用一次 regional prediction。

走出圆盘或方向发生需要重新锚定的变化时，再重新 query。

---

# 3.10 三个容易混淆的数字

统一记忆：

```text
32 = offline GT camera positions
 9 = V5 单次 query 内部的 analytic field support points
 1 = runtime 每个 view-cell 的 batch model query
```

其中 9 个 support points 不定义新的 view-cell，也不需要额外 rendering。

它们只是 V5 structured field query 的内部计算点。
