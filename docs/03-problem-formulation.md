# 03 — 问题定义（Problem Formulation）

## 本节目标

把任务形式化，使它与以下问题清楚地区分：

- 普通 from-point occlusion culling；
- 离散的 precomputed PVS；
- runtime geometry-to-PVS inference。

## Renderable Units 与 Streaming Resources

定义 renderable units：

[
\mathcal U = \{u_i\}_{i=1}^N.
]

定义 streaming resources / GLBs：

[
\mathcal R = \{r_j\}_{j=1}^M.
]

存在映射：

[
g(u_i)=r_j.
]

一个 GLB/resource 可以包含多个 unit/instance。

## View Region / View-Cell

定义一个观察区域：

[
\mathcal B.
]

当前数据协议中，region 可以是：

- disk；
- oriented box。

## From-Region Ground Truth

当且仅当区域内至少有一个采样相机/子视点可以观察到 unit (u_i) 的有效可见贡献时：

[
y_i(\mathcal B)=1.
]

可以写成：

[
y_i(\mathcal B)=
\mathbf 1
\left[
\exists c\in\mathcal B:
u_i\text{ contributes to the rendered image}
\right].
]

最终论文必须使用数据集真正的 GT 语义，而不是笼统说“object visible”。

## Pre-Geometry Constraint

做下载决策时，客户端还没有详细内容：

[
G_i.
]

它只有一个紧凑的 pre-transmitted / compiled representation：

[
a_i.
]

目标是：

[
f_\theta(a_i,\mathcal B)
\rightarrow z_i,
]

其中 (z_i) 是 visibility logit / score。

设计要求：

[
|a_i|\ll |G_i|.
]

## Conservative Asymmetry

false negative：

- 可见 geometry 被延迟或漏掉；
- 造成 missing geometry / hole / pop-in / delayed correct frame。

false positive：

- 多下载或多保留了一些暂时不可见 geometry。

因此两者成本不对称：

[
C_{FN}\gg C_{FP}.
]

这正是后面 constrained training 和 safety-first evaluation 的动机。

## 两种部署使用方式

### 1. Conservative filtering

固定零边界：

[
\hat y_i=
\mathbf 1[z_i\ge0].
]

用于判断哪些候选可以保留/延后。

### 2. Resource ranking

实例分数聚合到资源：

[
p_g=
\max_{i:g(i)=g}s_i
]

或最终实现采用的精确聚合方式。

论文必须把：

- threshold filtering；
- threshold-free ranking；

作为两个独立任务讨论。
