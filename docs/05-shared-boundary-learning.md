# 05 — Shared Conservative Boundary Learning（共享保守边界学习）

## 本节目标

这一节解释 V5 为什么采用 safety-oriented objective，而不是只优化普通分类准确率或 PR-AUC。

PVS 中 false negative 与 false positive 的代价明显不对称：漏掉真正可见的 unit 会直接影响画面完整性，而额外保留不可见 unit 主要增加冗余计算或传输。因此训练目标需要在提升 culling efficiency 的同时显式约束 visible misses。

当前 V5 进一步使用统一的 $z=0$ operating point，便于在不同场景和实验中采用一致的决策规则；这是一种训练与部署选择，而不是对离线阶段“不能生成 visibility labels”这一条件的依赖。

---

# 5.1 Fixed Zero Decision Boundary

定义：

$$
h_{keep}(z)
=
\frac{
\operatorname{softplus}(z)
}{
\ln2
},
$$

$$
h_{miss}(z)
=
\frac{
\operatorname{softplus}(-z)
}{
\ln2
}.
$$

在：

$$
z=0
$$

处：

$$
h_{keep}(0)
=
h_{miss}(0)
=
1.
$$

最终 deployment decision：

$$
z\ge0
\Rightarrow
\text{keep / potentially visible}.
$$

这里的目标不是学习一个完美 calibrated probability。

目标是：

> **让零点成为跨场景一致、便于比较和部署的 conservative operating point。**

---

# 5.2 Extra-Retention Objective

对于 source scene (s)：

$$
J_{extra}^{(s)}
=
\frac{
\sum_{i:y_i=0}
q_i h_{keep}(z_i)
}{
G_s
}.
$$

其中：

- $q_i$：pose inverse-sampling correction；
- $G_s$：source-train split 的 visible occurrence 总数。

直觉：

> 对真正不可见的候选，如果模型仍给出正 logit，就意味着系统会额外保留/下载它。

因此：

$$
J_{extra}
$$

是 efficiency side。

---

# 5.3 Unified Safety Risk

定义 source scene 中正例的平均 visual weight：

$$
\mu_s
=
W_s/G_s.
$$

把每个 visible unit 的安全权重写成：

$$
\tilde w_i
=
\rho
+
(1-\rho)
\frac{w_i}{\mu_s}.
$$

当前正式配置：

$$
\rho=0.1.
$$

于是 safety risk：

$$
R_s
=
\frac{
\sum_{i:y_i=1}
q_i
\tilde w_i
h_{miss}(z_i)
}{
G_s
}.
$$

这个设计同时保留两层含义：

### Count floor

即使一个 visible unit 屏幕贡献很小，它仍至少获得 (ho) 对应的安全权重。

避免模型为了 weighted recall 漂亮而集中漏掉大量“小物体”。

### Visual importance

visual contribution 大的 visible unit 权重更高。

避免纯 count recall 把所有 visible unit 一视同仁。

因此当前 V5 把：

- count safety；
- visual safety；

统一成一个风险，而不是两个互相竞争的 dual constraints。

---

# 5.4 Domain-Robust SmoothMax

当前训练划分成 10 个风险域：

### 5 个真实场景

- HKUST
- IFCBench
- Sponza
- Viking Village
- Big City

### 5 个 synthetic structure families

总计：

$$
K=10.
$$

每个域维护 detached risk EMA：

$$
m_s
\leftarrow
\beta m_s
+
(1-\beta)R_s,
$$

其中：

$$
\beta=0.99.
$$

根据：

$$
p_s=
\operatorname{softmax}
$\alpha m_s$
$$

分配 worst-domain-sensitive 权重，当前：

$$
\alpha=16.
$$

robust risk 使用 smooth approximation of max：

$$
R_{robust}
\approx
\max_s R_s.
$$

最终优化目标：

$$
\min_\theta
J_{extra}
$$

subject to：

$$
R_{robust}
\le
0.01.
$$

Lagrangian：

$$
\mathcal L
=
J_{extra}
+
\lambda
(
R_{robust}-0.01
).
$$

只维护：

> 一个 shared (lambda)。

这样训练目标直接对应：

> **在所有 source domains 中，让固定零边界都尽量保持安全，而不是只优化平均场景。**

---

# 5.5 当前正式目标与旧 V5 草案的区别

这一点必须在论文仓库里长期记住。

当前 robust-boundary 正式实验：

- **不使用 field NLL**；
- **不读取 external-hit probe**；
- 仍然保留 structured survival field 作为 representation；
- 主评价使用 fixed-zero；
- calibrated threshold 只作为诊断。

因此：

> 旧设计文档里写的 $L_{\mathrm{vis}}+L_{\mathrm{surv}}$、field-NLL 等内容，不应该再定义最终论文的主训练目标。

最终论文必须服从：

- 当前代码；
- current robust-boundary experiment；
- frozen run config。

---

# 5.6 PBCE Objective Control

PBCE control 保留：

- Full architecture；
- relation compiler；
- structured field；
- query head。

只把训练目标换成：

> pose-balanced BCE。

同时：

- 不增加 field supervision；
- 不引入额外信息。

因此它回答一个非常干净的问题：

> **普通的 balanced classification objective，是否会自然产生一个跨场景都安全的 $z=0$ operational boundary？**

论文对比重点不应该只是 accuracy。

应该重点展示：

- fixed-zero WR；
- WR LCB；
- calibrated diagnostic；
- PR-AUC；
- positive/negative score distribution；
- training dynamics（如果稳定）。

如果 PBCE ranking 不差、但 fixed-zero safety 明显失败，那么这是非常好的故事：

> **该对照用于验证 safety-oriented objective 是否能在统一 operating point 下取得更好的 safety–efficiency trade-off。**

---

# 5.7 训练协议

当前 formal shared training：

- 5 个 real source scenes；
- 96 个 synthetic train scenes；
- two-real / one-synthetic 调度；
- 每个 real scene 36k updates；
- synthetic 共 90k updates；
- 总计 270k；
- 每步 4 个 pose；
- global yaw augmentation；
- AdamW；
- model LR = 2e-4；
- weight decay = 1e-5；
- dual 参数以最终 frozen run config 为准。

最终论文只写：

> **实际冻结实验配置中的值。**

不要从旧设计稿里拷贝已经废弃的训练超参数。
