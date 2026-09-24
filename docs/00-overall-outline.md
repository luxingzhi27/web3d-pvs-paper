# 论文总体大纲

## 暂定标题

**Pre-Geometry Visibility for Progressive Web3D via Geometry-Compiled Occlusion Fields**

## 核心问题

大型 Web3D 场景需要渐进式传输，但传统遮挡可见性计算通常依赖已经可访问的场景表示。本文研究：

> **如何在 detailed geometry 尚未 resident 于客户端时，为当前局部观察区域预测 unit-level from-region visibility，并将其用于安全剔除与 progressive delivery？**

核心方法是把 scene occlusion context 在离线阶段编译成 compact、client-queryable visibility representation。

---

# 1. Introduction

## 1.1 背景

说明大规模 Web3D 无法在交互开始前完整下载，客户端需要持续进行：

- candidate selection；
- culling；
- progressive content ordering。

现有 pre-download selection 通常依赖：

- frustum；
- distance；
- hierarchy；
- SSE / geometric error。

这些信号能够估计几何重要性，但通常缺少 occlusion-aware relevance。

## 1.2 From-Region Visibility 的意义

说明 from-region PVS 相比单视点 visibility 更适合：

- camera motion；
- prefetching；
- remote rendering；
- progressive rendering。

本文预测的不是当前单一 camera 的瞬时 visible set，而是一个局部 view-cell 内的 potentially visible units。

## 1.3 Pre-Geometry Visibility Gap

已有 precomputed PVS、online PVS 和 learned PVS 已显著降低可见性计算成本，但通常假设 visibility processor 可以访问足够表达遮挡关系的 scene representation。

Web progressive delivery 中存在不同的约束：

~~~text
visibility helps decide what content is useful
                     ↑
detailed scene content may not be resident yet
~~~

据此提出本文的 operating point：

> **pre-geometry from-region visibility**

即在 detailed geometry residency 之前获得 occlusion-aware regional visibility。

## 1.4 核心思想

离线内容构建阶段拥有完整场景，因此可以预先分析 geometry-dependent occlusion context，并将其编译为紧凑表示：

~~~text
scene geometry
      ↓
offline occlusion compilation
      ↓
compact visibility asset
      ↓
client-side view-cell query
      ↓
unit-level visibility score
~~~

运行时客户端无需重新执行完整场景遮挡分析。

## 1.5 方法概览

简要介绍 GCOF-PVS：

1. 编码 unit local surface geometry；
2. 构造 surrounding potential-occluder relations；
3. 聚合 directional occlusion evidence；
4. 编译为 continuous directional survival field；
5. 对当前 horizontal-disk view-cell 做 analytic region query；
6. 结合 local geometry、regional occlusion statistics 和 query geometry 输出 visibility score。

## 1.6 主要贡献

建议最终压缩为三点：

1. **Pre-Geometry From-Region Visibility**  
   提出面向 progressive Web3D 的 visibility operating point，使客户端在 detailed geometry residency 之前获得 unit-level occlusion relevance。

2. **Geometry-Compiled Occlusion Representation**  
   将 local geometry 与 surrounding occlusion context 编译成 compact directional survival field，并支持连续方向和区域查询。

3. **Systematic Evaluation for Progressive Web3D**  
   从 visibility safety/efficiency、representation ablation、cross-scene transfer、runtime/storage cost 与 progressive delivery 评估方法。

---

# 2. Related Work

Related Work 围绕“visibility 在什么时候、依赖什么 representation、由谁计算”组织，而不是按网络组件分类。

## 2.1 Visibility Culling and From-Region PVS

### Classical Runtime Occlusion Culling

简要介绍：

- HZB；
- hardware occlusion query；
- CHC 类方法。

作用是建立标准 geometry-resident runtime visibility 情形。

### Precomputed From-Region PVS

讨论：

- Teller & Séquin；
- visibility preprocessing；
- Adaptive Global Visibility Sampling 等。

强调：

> 传统方法可以把大量 geometry reasoning 移到 offline stage，但通常得到 view-cell-specific PVS 或相关预计算结果。

### Online From-Region PVS

重点讨论：

- Camera Offset Space；
- Guided Visibility Sampling++；
- Trim Regions；
- Disocclusion Buffer。

比较维度：

- runtime scene representation；
- preprocessing；
- query region；
- GPU/runtime cost。

本节收束到：

> 现代 PVS 已经可以高效在线计算；本文关注的是 visibility computation 发生在 detailed content residency 之前这一不同 operating point。

## 2.2 Learned Visibility Representations

### NeuralPVS

作为最重要的 learned-PVS comparison。

重点比较：

~~~text
NeuralPVS:
runtime geometry representation
        ↓
neural from-region PVS
~~~

与：

~~~text
GCOF-PVS:
offline scene compilation
        ↓
compact visibility representation
        ↓
runtime region query
~~~

核心区别是 geometry 在 pipeline 中何时参与，而不是简单比较网络速度。

### Neural Visibility of Point Sets

作为 per-element learned visibility 的相关工作，说明 visibility classification 本身不是本文 novelty。

### NVGS

重点比较：

- render-time primitive visibility；
- pre-geometry from-region visibility。

共同点是将 visibility knowledge 编码成轻量可查询表示；主要区别是任务 granularity 和 runtime operating point。

## 2.3 Visibility-Aware Streaming

讨论：

- server-side/networked PVS；
- Smart Visible Sets；
- Streaming QSplat；
- View-Dependent Progressive Meshes；
- Streaming HLODs；
- Fine-Grained Web3D Pipeline；
- 3D Tiles。

说明：

> visibility-guided delivery 本身并不是新的。

本文的研究重点是：

> **如何在 detailed content residency 之前获得 client-queryable occlusion signal。**

---

# 3. Method

本节是论文技术核心。

---

## 3.1 Problem Setting and View-Cell Protocol

### Renderable Unit

场景统一划分为可独立剔除的 renderable units：

$$
\mathcal U=\{u_i\}_{i=1}^{N}.
$$

BIM 场景中 unit 为构件；标准图形学场景使用冻结 partition rule 生成 units。

所有：

- candidate；
- GT；
- prediction；
- culling；

均在 unit level 定义。

### Horizontal-Disk View-Cell

一个 view-cell 由：

- region center；
- fixed camera direction；
- horizontal disk radius；
- projection parameters；

共同定义。

离线数据协议在每个 physical center 上展开：

- yaw：$0^\circ,90^\circ,180^\circ,270^\circ$；
- pitch：$-15^\circ,0^\circ,15^\circ$；

共 12 个数据方向。

这些方向只用于训练与评价覆盖，runtime camera 不需要吸附到这些方向。

### Regional Ground Truth

每个 view-cell 在水平圆盘内使用 32 个 camera positions 做 Color-ID rendering。

regional PVS：

$$
\mathrm{PVS}(\mathcal B)
=
\bigcup_{m=1}^{32}V(c_m).
$$

因此 GT 表示：

> sampled region 内至少一个 camera position 能看到该 unit。

visual weight 使用 32 个 sampled views 中的最大屏幕贡献。

### Candidate Generation

从 region center 沿观察方向后退：

$$
\Delta
=
\frac{r}{\tan 30^\circ},
$$

使用 $66^\circ$ AABB frustum 构造 candidate units。

candidate 必须独立于 GT 生成，并检查 sampled GT visible units 被 candidate set 覆盖。

---

## 3.2 Local Surface Geometry Encoding

每个 unit 使用 256 个表面采样点，每点包含：

$$
(x,y,z,n_x,n_y,n_z).
$$

共享 point encoder 经过 symmetric max/mean pooling 和 unit size ratios，得到：

$$
z_i\in\mathbb R^{32}.
$$

该 descriptor 表示 unit 本身的局部几何特征。

这一节重点说明：

> local geometry 提供 target shape information，但不能单独描述 surrounding occlusion context。

这自然引出下一节。

---

## 3.3 Potential-Occluder Relations

对每个 target unit，在 12 个固定正二十面体 anchor directions 上构造 potential occlusion relations。

### Relation Construction

对于每个 anchor direction：

1. 将 unit AABB 正交投影到垂直于该方向的平面；
2. 检查 projected overlap；
3. 结合 depth ordering；
4. 选取最多 top-8 potential occluders。

### Relation Features

每条 edge 编码：

- relative direction；
- normalized distance；
- relative scale；
- projected overlap；
- depth gap。

### Relation Aggregation

对于 target $i$、anchor $k$、source $j$：

$$
m_{ijk}
=
\operatorname{MLP}(z_i,z_j,e_{ijk}).
$$

使用 attention 聚合：

$$
h_{ik}
=
\sum_j\alpha_{ijk}m_{ijk}.
$$

并保留 neighbor count 和 overlap sum。

每个 anchor 最终输出一个 7D response：

$$
q_{ik}\in\mathbb R^7.
$$

因此每个 unit 得到：

$$
Q_i\in\mathbb R^{12\times7}.
$$

这一步的作用是：

> 把离散的 surrounding geometry relations 编译成不同观察方向上的遮挡 response。

---

## 3.4 Geometry-Compiled Directional Survival Field

这一节需要作为 Method 的重点解释。

### 3.4.1 Continuous Directional Field

12 个 anchor responses 只存在于离散方向，而 runtime camera direction 是连续的。

使用一阶方向 basis：

$$
\phi(d)
=
[1,d_x,d_y,d_z].
$$

12 个 anchor basis 构成：

$$
A\in\mathbb R^{12\times4}.
$$

使用固定 pseudoinverse：

$$
C_i
=
A^{+}Q_i,
$$

得到：

$$
C_i\in\mathbb R^{4\times7}.
$$

因此任意方向 $d$ 都可以查询：

$$
r_i(d)
=
\phi(d)C_i
\in\mathbb R^7.
$$

这一步把：

$$
12\times7=84
$$

个 directional responses 压缩为：

$$
4\times7=28
$$

个 continuous-field coefficients。

### 3.4.2 Survival Interpretation

把沿方向 $d$ 遇到有效遮挡的距离记为：

$$
T_i(d).
$$

定义：

$$
S_i(d,t)
=
P(T_i(d)>t).
$$

直观含义是：

> 从 target 沿方向 $d$ 走到距离 $t$ 时，遮挡事件尚未发生的程度。

需要明确：

> $S_i$ 是 structured occlusion statistic，而不是经过概率校准的最终 visibility probability。

### 3.4.3 Seven-Parameter Survival Model

方向查询得到的 7 个参数对应：

- no-hit mass $p_\infty$；
- mixture weights $\pi_1,\pi_2$；
- locations $\mu_1,\mu_2$；
- scales $s_1,s_2$。

normalized distance：

$$
t
=
\log\left(1+\frac{d}{r_i}\right).
$$

完整 survival 由两个 truncated logistic components 构成。

representation 满足：

$$
S(0)=1,
$$

并在固定方向上满足：

$$
d_2>d_1
\Rightarrow
S(d_2)\le S(d_1).
$$

因此 structured field 注入：

- directional continuity；
- distance monotonicity；
- scale awareness。

### 3.4.4 为什么需要 Structured Field

这一节明确论文 hypothesis：

> 对遮挡这样的 direction-distance phenomenon，具有连续方向与单调距离结构的 representation，可能比同容量 unrestricted latent 更适合作为 compact runtime visibility representation。

该 hypothesis 由 GENERIC_RELATION_28 消融验证。

---

## 3.5 From-Region Survival Query and Visibility Prediction

### Analytic Support Query

对于 horizontal-disk view-cell，runtime 在 field 上查询：

- disk center；
- 8 个圆周点。

对于 support point $x_k$：

$$
d_{ik}
=
\frac{x_k-c_i}{\|x_k-c_i\|},
$$

$$
\ell_{ik}
=
\|x_k-c_i\|.
$$

得到：

$$
S_{ik}
=
S_i(d_{ik},\ell_{ik}).
$$

### Regional Statistics

9 个 values 汇总为：

$$
[
S_{\mathrm{center}},
S_{\max},
S_{\mathrm{mean}},
S_{\min}
].
$$

这些 statistics 提供区域级 occlusion evidence。

尤其：

- $S_{\max}$ 对应区域中最强 unoccluded evidence；
- $S_{\min}$ 表示最强遮挡位置；
- $S_{\mathrm{mean}}$ 表示整体区域状态。

### Final Visibility Head

survival statistics 不是最终 PVS。

最终模型联合：

- local geometry descriptor；
- regional survival statistics；
- query geometry；

输出 unit-level visibility logit：

$$
\ell_i^{\mathrm{vis}}.
$$

需要明确：

$$
\text{survival field}
\neq
\text{visibility probability}.
$$

更准确地说：

$$
\text{survival field}
=
\text{structured directional occlusion representation}.
$$

---

## 3.6 Conservative Visibility Learning

PVS 中 false negative 与 false positive 的代价不对称：

- false negative：potentially visible unit 被漏掉；
- false positive：不可见 unit 被额外保留。

因此训练目标不是单纯最大化 classification accuracy，而是在安全约束下提高 culling efficiency。

### Fixed Operating Point

主协议使用：

$$
z=0
$$

作为统一 operating point。

### Efficiency Objective

对 negative units 惩罚额外保留：

$$
J_{\mathrm{extra}}.
$$

### Safety Risk

对 positive units 使用结合 count floor 和 visual importance 的 safety risk：

$$
R_s.
$$

### Domain-Robust Constraint

在多个 real scenes 和 synthetic structure families 上聚合风险，并优化：

$$
\min_\theta J_{\mathrm{extra}}
\quad
\text{s.t.}
\quad
R_{\mathrm{robust}}\le\epsilon.
$$

具体 SmoothMax、EMA 和 shared dual 放在方法细节中描述，不需要在 Introduction 重复。

---

# 4. Progressive Web3D Integration

本节说明方法如何从 offline compiled representation 转化为 Web runtime capability。

## 4.1 Offline Compilation

对场景执行：

~~~text
unit surface geometry
        ↓
local descriptor
        +
potential-occluder relations
        ↓
relation compiler
        ↓
4×7 directional survival field
        ↓
compact visibility asset
~~~

离线阶段负责 scene-context analysis；客户端只接收 runtime 所需的 compact representation。

## 4.2 Runtime View-Cell Query

当前 camera position 和 orientation 建立 horizontal-disk view-cell。

通过后退 $66^\circ$ AABB frustum 获得 candidate units。

随后：

~~~text
candidate units
      ↓
one batched model query
      ↓
regional visibility scores
~~~

需要严格区分：

~~~text
32 = offline GT camera samples
 9 = V5 internal analytic support points
 1 = runtime batched model query
~~~

## 4.3 Regional PVS and Instantaneous Rendering

模型得到 regional PVS：

$$
\widehat{\mathrm{PVS}}(\mathcal B).
$$

随后结合当前真实 $60^\circ$ display frustum 得到 instantaneous units。

因此：

~~~text
66° regional PVS
      ↓
predicted potentially visible units
      ↓
current 60° display frustum
      ↓
instantaneous render/process set
~~~

## 4.4 Progressive Ordering

连续 visibility score 还可以作为 progressive ordering signal。

本节不讨论具体文件打包方式，只研究：

> unit-level occlusion-aware relevance 是否能够让有用内容更早到达。

---

# 5. Evaluation

Evaluation 分为主结果、已有消融、泛化、runtime/storage 和 progressive delivery 五类证据。

## 5.1 Experimental Setup

### Scenes

介绍：

- real scenes；
- synthetic training scenes；
- external holdout。

### View-Cell Protocol

统一使用：

- physical region centers；
- 12 offline directions；
- horizontal disk；
- 32 GT positions；
- $66^\circ$ Color-ID GT；
- independent candidate generation；
- center-grouped data split。

### Baselines

根据最终实验完成情况选择：

- simple geometric heuristics；
- HZB / proxy-based visibility；
- existing learned visibility baseline；
- GT oracle for streaming upper bound。

### Metrics

主 visibility metrics：

- Weighted Recall；
- one-sided 95% LCB；
- Bad Cull；
- Useful Cull；
- CNOR；
- PR-AUC。

---

## 5.2 Main Visibility Results

使用 FULL 模型报告主要 visibility quality。

重点回答：

> 在统一 fixed operating point 下，模型能否保持高 safety，同时有效减少不可见 candidate units？

主要报告：

- per-scene；
- scene-equal；
- worst-scene；

结果。

calibrated threshold 只作为 diagnostic，不替代 fixed operating point 主结果。

---

## 5.3 Ablation Study

只包含当前正式三组消融。

### 5.3.1 Occlusion Context

FULL vs GEOMETRY_FIELD

验证：

> surrounding potential-occluder context 是否提供了超出 local target geometry 的信息。

### 5.3.2 Structured Field

FULL vs GENERIC_RELATION_28

验证：

> structured directional survival field 是否优于同容量 unrestricted latent。

这一组是 survival-field representation 的核心证据。

### 5.3.3 Conservative Objective

FULL vs PBCE_OBJECTIVE

验证：

> safety-oriented learning 是否在统一 operating point 下取得更好的 safety–efficiency trade-off。

---

## 5.4 Cross-Scene Generalization

### LOSO

评价 frozen model 在 held-out real scene 上的迁移能力。

### External Holdout

在模型和实验协议冻结后，对独立场景进行评价。

报告：

- fixed operating point；
- 如需要，单独报告 target-calibrated diagnostic。

本节用于分析 representation/model 的 cross-scene transfer，不把“完全禁止离线 visibility labels”作为方法前提。

---

## 5.5 Runtime and Storage Cost

针对 V5 实际部署测量：

- compact visibility asset size；
- bytes/unit；
- shared model size；
- offline compilation time；
- WebGPU latency；
- WASM latency；
- candidate-count scaling。

核心问题：

> compact pre-geometry visibility representation 的成本是否足够低，能够在 Web 客户端使用？

---

## 5.6 Progressive Delivery

在相同 candidate unit set 下比较不同 ordering signals：

- baseline/original order；
- distance / projected-area heuristic；
- AABB/proxy baseline；
- HZB visible-first；
- V5 visibility score；
- GT visibility oracle。

报告：

- Cost/Bytes@95；
- Cost/Bytes@99；
- Cost/Bytes@99.9；
- waste；
- time-to-coverage。

若完成真实 scheduler replay，则作为同一节的进一步 system result，而不是新的消融实验。

---

# 6. Discussion and Limitations

## 6.1 Per-Unit Representation Cost

runtime asset 和 query cost 随 unit 数增长，需要讨论：

- scene granularity；
- candidate filtering；
- memory/runtime trade-off。

## 6.2 AABB Proxy Approximation

Potential-occluder construction 基于 AABB projected overlap，可能难以精确表达：

- thin geometry；
- porous structures；
- foliage；
- highly non-box-like occluders。

## 6.3 Low-Order Directional Field

一阶方向 basis 提供 compactness 与 continuity，但限制 angular frequency。

需要讨论：

> structured inductive bias 与 representation capacity 之间的 trade-off。

## 6.4 Static Scene Assumption

当前 compiled visibility representation 主要针对静态遮挡结构。动态 occluders 需要：

- local recompilation；
- runtime residual visibility；
- hybrid HZB 等扩展。

## 6.5 Safety–Efficiency Trade-Off

Conservative PVS 天然存在：

$$
\text{safety}
\leftrightarrow
\text{culling efficiency}
$$

的 trade-off。

本文目标不是最大化单一 classification metric，而是在高 safety 下尽可能减少冗余 units。

## 6.6 Generalization Scope

LOSO / external holdout 只能证明 tested distributions 下的 cross-scene transfer，不能泛化为 arbitrary open-world geometry。

---

# 7. Conclusion

总结三点：

1. **问题**：progressive Web3D 需要 detailed geometry residency 之前的 occlusion-aware relevance；
2. **方法**：把 surrounding occlusion context 离线编译成 compact directional survival field，并用轻量 region query 得到 unit-level PVS score；
3. **意义**：visibility 可以作为一种独立、紧凑的 scene representation，在 detailed content arrival 之前服务于 culling 和 progressive delivery。

最终结论应回到：

> **pre-geometry visibility 是一种新的 visibility operating point，而 geometry-compiled occlusion field 提供了实现这一能力的紧凑表示。**
