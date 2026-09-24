# 02 — Related Work（相关工作）

## 本节目标

Related Work 不应按照我们自己的网络模块来分类，例如：

- PointNet；
- GNN；
- attention；
- spherical harmonics；
- survival analysis。

这些只是实现组件，不是论文真正所在的研究邻域。

更合适的组织方式是围绕一个问题展开：

> **已有方法是在什么时候、在哪里、依赖什么场景表示，才能获得 visibility？**

建议保留三条主线：

1. Visibility Culling and From-Region PVS
2. Learned Visibility Representations
3. Visibility-Aware 3D / Web Streaming

最终目标不是证明“没人做过”，而是把 V5 所处的 operating point 精确定位出来。

---

# 2.1 Visibility Culling and From-Region PVS

## A. Runtime from-point culling：简短交代即可

代表工作：

- Hierarchical Z-Buffer（HZB）
- hardware occlusion query / CHC 类方法

这一部分的作用只是建立经典 runtime 情形：

> 当 renderer 已经拥有足够的场景表示时，传统 runtime occlusion culling 可以高效剔除当前视点不可见的几何。

它们是很重要的系统 baseline，但不是本文最接近的 research question，因此不需要做长篇历史综述。

这一段最后要引出：

> 本文关心的不是“几何已经在 renderer 中时如何更快剔除”，而是“几何尚未到达客户端时如何获得遮挡相关性”。

---

## B. Classical Precomputed PVS

建议重点引用：

- Teller & Séquin, *Visibility Preprocessing for Interactive Walkthroughs*, SIGGRAPH 1991
- Wonka et al., *Visibility Preprocessing with Occluder Fusion for Urban Walkthroughs*, 2000
- Nirenstein & Blake, *Hardware Accelerated Visibility Preprocessing using Adaptive Sampling*, EGSR 2004
- Bittner et al., *Adaptive Global Visibility Sampling*, SIGGRAPH 2009

这条文献线已经充分证明：

- from-region PVS 是成熟问题；
- 可以把昂贵的 geometry visibility reasoning 移到 offline preprocessing；
- runtime 可以直接查询/读取 PVS，而不重新做完整遮挡计算。

经典流程可以概括为：

```text
完整场景几何
    ↓ offline preprocessing
view-cell-specific PVS database
    ↓ runtime lookup
```

这里要特别注意 novelty：

> **offline preprocessing 本身绝对不是本文的创新。**

我们的区别不是“我们也提前算 visibility”，而是：

> 传统 PVS 通常存储 view-cell-specific 的离散 visibility solution；V5 则把 scene occlusion context 编译成 per-unit、可连续 region query 的 compact representation。

也就是说，Related Work 里要逐步从：

$$
	ext{precompute answers}
$$

过渡到：

$$
	ext{compile a reusable query representation}.
$$

---

## C. Networked PVS and Selective Transmission

必须认真讨论：

- Koltun, Chrysanthou, Cohen-Or, *Hardware-Accelerated From-Region Visibility Using a Dual Ray Space*（2001）

这篇对我们非常重要，因为它已经把：

$$
	ext{PVS}
ightarrow
	ext{selective network transmission}
$$

直接用于 networked walkthrough。

因此不能声称：

> “使用 PVS 指导网络传输是本文首次提出。”

正确的区别应该是 computation placement 和 representation。

传统 server-side networked PVS：

```text
server 拥有完整几何
    ↓
server 计算 PVS
    ↓
client 接收筛选后的内容
```

本文目标：

```text
geometry 在离线阶段编译
    ↓
client 先获得很小的 visibility asset
    ↓
client 自己查询 from-region visibility
    ↓
client 决定 detailed geometry 的优先级
```

因此：

> **创新不在“PVS 可以指导传输”，而在 visibility signal 在详细内容到达之前如何以可查询形式存在于客户端。**

---

## D. Online From-Region PVS

现代 online PVS 需要重点覆盖：

- Camera Offset Space（SIGGRAPH Asia 2019 / TOG）
- Guided Visibility Sampling++（I3D 2021）
- Trim Regions（SIGGRAPH 2023 / TOG）
- Disocclusion Buffer（SIGGRAPH Asia 2025）

### Camera Offset Space

作用：

- 代表 GPU online from-region PVS；
- 而且本身就面向 streaming rendering。

因此引用这篇以后，就不能写：

> “现有 from-region PVS 都依赖昂贵 offline preprocessing。”

这个表述已经过时。

### Guided Visibility Sampling++

作用：

- 代表基于 modern ray-tracing / sampling 的 aggressive online PVS；
- 说明现代硬件已经可以很快地做 region visibility sampling。

### Trim Regions

这篇应该是传统 online PVS 里最重要的比较之一。

它明确讨论：

- dynamic applications；
- streaming content over variable-bandwidth networks；
- arbitrary region 的 online PVS。

不能攻击它“不适合 streaming”。

更准确的区别是：

> Trim Regions 解决的是**当 visibility processor 已经拥有足够 scene representation 时**，如何实时计算 arbitrary-region PVS；本文解决的是 Web 客户端在 detailed scene content 仍未 resident 时，如何先获得一个可用于下载决策的 occlusion-aware signal。

### Disocclusion Buffer

作用：

- 代表较新的高度并行 conventional PVS；
- 防止论文形成错误叙事：好像 traditional/online PVS 普遍很慢，所以才需要 neural method。

更公平的说法是：

> 传统 PVS 的计算效率仍在持续提高，但其 operating point 通常仍是假设 visibility computation 可以访问一份足够详细的场景表示。

### 2.1 的收束句

建议最终收敛到：

> 预计算、采样式、image-space 与 disocclusion-based 方法不断降低 from-region visibility 的计算成本，但这些方法通常共同假设：执行 visibility computation 的一方能够访问一份足够表达遮挡关系的 runtime scene representation。

注意不要绝对化成：

> “都需要完整原始 mesh”。

---

# 2.2 Learned Visibility Representations

这一节最重要的不是泛泛讲“深度学习在图形学中的应用”，而是直接讨论：

> **visibility 本身如何被学习和压缩成可查询表示。**

---

## A. NeuralPVS：最重要的 learned baseline / conceptual comparison

NeuralPVS 应该是这一节篇幅最大的一篇。

它的 pipeline 可以概括为：

```text
scene geometry
    ↓ froxelization
runtime froxel geometry representation
    ↓ sparse neural network
from-region PVS
```

必须公平承认它的优势：

- learned from-region PVS；
- runtime 性能强；
- 使用 synthetic scenes 支持跨场景泛化；
- inference cost 主要由固定 froxel resolution 决定，而不是直接随原始几何复杂度增长。

不要把我们的工作写成：

> “NeuralPVS but smaller”。

真正区别在于：

> **geometry 在 pipeline 中什么时候出现。**

NeuralPVS：

```text
runtime geometry representation
    ↓
neural PVS
```

GCOF-PVS：

```text
offline geometry
    ↓ shared compiler
compact visibility representation
    ↓ transfer
compact representation + region
    ↓
runtime visibility
```

建议使用这样的句子：

> NeuralPVS 学习的是从 runtime geometric scene representation 到 PVS 的快速映射；GCOF-PVS 则把 scene-context reasoning 前移到 offline geometry compilation，只把紧凑的 per-unit visibility representation 传给客户端。

这不是“哪个网络更快”的区别，而是：

> **visibility computation 与 content residency 的 operating point 不同。**

---

## B. Neural Visibility of Point Sets

作用：

- 证明 per-element、view-dependent visibility 可以被学习；
- 防止我们错误声称 neural visibility classification 本身是新的。

这一篇无需展开太多。

它主要支撑：

$$
(	ext{element},	ext{view})
ightarrow
	ext{visibility}
$$

这个 learned representation 思路已经有 precedent。

---

## C. NVGS（CVPR 2026）

NVGS 与 V5 的“轻量可见性资产”思路很接近，建议认真写。

其大致逻辑：

```text
已有 3DGS asset
    ↓ visibility extraction
visibility samples
    ↓ neural distillation
compact neural visibility
    ↓ runtime query
在 rasterization 之前剔除不可见 Gaussian
```

与我们的共同点：

> 都试图把昂贵的可见性信息提前编码到轻量 representation 中，并在 expensive graphics work 之前查询。

关键区别：

- NVGS 面向已经 resident 的 3D Gaussian asset，在 rasterization 前预测 primitive visibility；
- V5 面向 from-region PVS，在 detailed scene content 尚未 resident 于客户端时查询 per-unit visibility；
- 两者都可以在离线阶段利用完整场景信息，差异主要在 representation granularity、query region 和 runtime operating point。

因此更合适的比较是：

> **render-time primitive visibility vs. pre-geometry from-region visibility**

而不是把“是否使用 target-scene visibility labels”作为主要区别。

---

## D. Neural visibility field / NeRV 类工作

可以简短引用，主要作用是说明：

> continuous directional visibility function 作为 neural/learned representation 已有先例。

但这些工作大多服务于：

- relighting；
- light transport；
- neural rendering；
- active mapping。

不要让这条文献线抢占 Related Work 主体。

---

## 2.2 的真正 literature gap

不要把 gap 写成：

> “Can visibility be learned?”

这个问题已经被回答了。

更准确的问题是：

> **Can scene occlusion context be compiled into a compact transferable representation that supports efficient from-region visibility queries before detailed content residency?**

这句话应该自然过渡到 V5 的：

- compact scene compilation；
- structured field；
- lightweight from-region query。

---

# 2.3 Visibility-Aware 3D and Web Streaming

这一节的目标不是证明：

> “没人用 visibility 做过 streaming”。

恰恰相反，要主动承认已有工作。

---

## A. Visibility-Guided Transmission

重点：

- Smart Visible Sets for Networked Virtual Environments（2002）
- 前面的 server-side networked PVS 工作

Smart Visible Sets 已经建立：

```text
PVS
  ↓ direction / distance / importance
network transmission priority
```

因此：

> **visibility score → download priority 不是本文的核心 novelty。**

正确表述应是：

> scheduler 是 proposed visibility signal 的 application；真正的研究问题是 detailed geometry residency 之前如何获得这个 occlusion-aware signal。

---

## B. View-Dependent Progressive Geometry Streaming

代表工作：

- Streaming QSplat
- View-Dependent Streaming of Progressive Meshes
- Streaming HLODs

这些工作已经证明：

- view-dependent transmission order；
- coarse-to-fine refinement；
- visual importance-guided progressive delivery；

都是成熟思路。

共同点通常是：

> client 已经拥有某种 coarse representation / hierarchy / proxy / refinement structure。

这条线的作用是说明：

> “what should arrive next?” 本来就是一个长期存在的 graphics problem。

---

## C. Web3D Pipeline

重要工作：

- *Fine-Grained Web3D Culling-Transmitting-Rendering Pipeline*（CGI 2023）

它已经直接把：

$$
	ext{culling}
+
	ext{network transmission}
+
	ext{browser rendering}
$$

放到统一 Web3D pipeline 中。

因此不能写：

> “本文首次结合 PVS 与 Web streaming”。

我们的区别仍然是：

> **visibility signal 的来源、大小，以及它是否能在详细内容 resident 之前被客户端直接查询。**

---

## D. 3D Tiles / 现代 Web 内容选择

3D Tiles 是非常重要的 practical background。

它已经解决：

> content 尚未下载时，客户端如何根据 lightweight metadata 决定是否请求/细化？

典型 metadata：

- bounding volume；
- hierarchy；
- geometric error / SSE；
- viewer request volume；
- content URI。

可以抽象为：

```text
metadata + camera
    ↓
request / refinement decision
```

本文希望补充：

```text
compact occlusion metadata + camera region
    ↓
occlusion-aware relevance
```

因此要强调：

> GCOF-PVS 是对 frustum/SSE/hierarchy-based selection 的补充，而不是替代。

---

# Related Work 最终应该证明什么

最后一段建议明确总结三件已有事实：

1. **PVS 可以指导 streaming** —— 早已成立；
2. **visibility 可以 online computation，也可以 learned** —— 已成立；
3. **Web client 可以在 content 到达前根据 lightweight metadata 做 request decision** —— 已成立。

本文研究的是三者交叉后的缺口：

> **在 detailed geometry 尚未 resident 时，客户端能否仅依靠 geometry-compiled compact metadata 查询 learned from-region visibility，并把它直接用于 progressive content delivery？**

---

# 不建议单独建立的 Related Work 小节

不要单独为以下内容开小节：

- PointNet / point encoder；
- Graph Neural Network；
- attention；
- spherical harmonics；
- survival analysis；
- generic deep learning in graphics。

这些是 Method 的实现选择，不是本文真正的 research neighborhood。

---

# 后续逐篇阅读文献时统一记录的字段

| 维度 | 需要回答的问题 |
|---|---|
| Visibility type | from-point 还是 from-region？ |
| Scene availability | query 时需要什么场景表示？ |
| Computation location | server / preprocessing / renderer / client？ |
| Representation | full geometry / proxy / voxel / froxel / PVS table / latent？ |
| Query output | binary PVS / primitive visibility / continuous score？ |
| Generalization | per-scene preprocessing / retraining / shared model？ |
| Streaming use | 无 / filtering / selective transmission / priority ordering？ |
| Main cost | storage / preprocessing / GPU / candidate count / startup bytes？ |
| 与 V5 的差异 | 最准确的一句话是什么？ |

---

# Novelty Claim Discipline

避免绝对化表达：

- “所有现有 PVS 都需要完整详细 mesh”；
- “此前没有 visibility-guided streaming”；
- “NeuralPVS 无法用于 Web streaming”；
- “我们首次学习 visibility”。

更稳妥的表达：

- “requires a sufficiently detailed runtime scene representation”；
- “does not directly address the client bootstrap condition considered here”；
- “operates at a different point in the content-residency pipeline”；
- “we focus on the intersection of learned visibility and pre-content delivery”。
