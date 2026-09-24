# 核心文献（Core Literature）

本文档用于记录**真正定义本文研究邻域**的文献，以及每篇论文在 Related Work 中承担什么作用。

最终 BibTeX 应优先从出版社、作者主页、项目主页等权威来源整理，不要直接从本文件复制未经核验的元数据。

---

# A. Classical / From-Region Visibility

## Teller & Séquin — *Visibility Preprocessing for Interactive Walkthroughs*（SIGGRAPH 1991）

作用：

- cell/PVS 的源头级工作；
- 建立 offline from-region visibility preprocessing 这条主线。

在本文中用于说明：

> 传统 PVS 很早就可以把昂贵 visibility computation 移到 preprocessing 阶段。

---

## Wonka et al. — *Visibility Preprocessing with Occluder Fusion for Urban Walkthroughs*（2000）

作用：

- 代表面向大型 urban scene 的 visibility preprocessing。

---

## Nirenstein & Blake — *Hardware Accelerated Visibility Preprocessing using Adaptive Sampling*（EGSR 2004）

作用：

- sampling-based aggressive region visibility；
- 可用于说明 PVS 不一定要求 exact solution。

---

## Bittner et al. — *Adaptive Global Visibility Sampling*（SIGGRAPH 2009）

作用：

- 更成熟的 global view-cell visibility preprocessing；
- 利用 spatial coherence；
- 适合代表 classical PVS preprocessing 的高水平方法。

---

# B. Online From-Region PVS

## Koltun, Chrysanthou, Cohen-Or — *Hardware-Accelerated From-Region Visibility Using a Dual Ray Space*（2001）

作用：

- 与 networked walkthrough 直接相关；
- 已经使用 PVS 支持 selective transmission。

对本文最重要的提醒：

> **不要声称 visibility-guided streaming 本身是新的。**

本文真正区别：

- prior work：server / visibility processor 拥有 scene geometry；
- ours：client 只先拿 compact visibility asset，再决定 detailed geometry。

---

## Camera Offset Space — Real-Time PVS for Streaming Rendering（SIGGRAPH Asia 2019 / TOG）

作用：

- modern GPU online from-region PVS；
- 标题和应用本身就直接关联 streaming rendering。

对本文的意义：

> 引用以后不能再写“from-region PVS 都需要昂贵 offline preprocessing”。

---

## Guided Visibility Sampling++（I3D 2021）

作用：

- 代表 modern ray-tracing / sampling-based online PVS；
- 说明硬件 ray tracing 已显著降低 region visibility sampling 成本。

---

## Voglreiter et al. — *Trim Regions for Online Computation of From-Region Potentially Visible Sets*（SIGGRAPH 2023 / TOG）

作用：

- 最重要的 conventional online-PVS comparison 之一；
- 明确讨论 dynamic / streaming applications。

正确对比：

> Trim Regions 解决“scene representation 已可访问时如何在线算 PVS”；本文解决“detailed scene content 尚未 resident 时客户端如何先获得 occlusion-aware relevance”。

---

## Disocclusion Buffer（SIGGRAPH Asia 2025）

作用：

- 代表最新 highly parallel conventional PVS 方向；
- 防止论文形成“online PVS 普遍很慢”的过时叙事。

---

# C. Learned Visibility

## Wang et al. — *NeuralPVS: Learned Estimation of Potentially Visible Sets*（SIGGRAPH Asia 2025）

这是本文最重要的 learned-PVS comparison。

关键点：

- froxelized runtime geometry input；
- sparse neural inference；
- from-region PVS；
- synthetic scenes；
- cross-scene generalization；
- inference cost 与固定 froxel resolution 密切相关。

最重要的区别：

> **runtime geometry-to-PVS inference**  
> vs.  
> **offline geometry-to-compact-visibility compilation + runtime compact query**

Related Work 中应公平承认它的固定 grid 复杂度优势，不要硬写成“我们更快”。

---

## *Neural Visibility of Point Sets*（SIGGRAPH Asia 2025）

作用：

- per-element learned visibility precedent；
- 证明 neural visibility classification 本身不是本文 novelty。

---

## *NVGS: Neural Visibility for Occlusion Culling in 3D Gaussian Splatting*（CVPR 2026）

作用：

- lightweight learned visibility；
- queried before rasterization；
- 与 V5 的“先编码 visibility knowledge，再轻量查询”思想非常接近。

最有价值的对比：

> **visibility-distilled asset**  
> vs.  
> **geometry-compiled visibility asset**

NVGS 的目标是 render-time primitive culling；V5 的目标是 pre-download selection。

---

## *NeRV: Neural Reflectance and Visibility Fields for Relighting and View Synthesis*（CVPR 2021）

作用：

- continuous directional visibility function 的表示先例。

不要展开太多，因为它服务的是：

- relighting；
- neural light transport；

而不是 PVS / streaming。

---

# D. Streaming / Web3D

## *Smart Visible Sets for Networked Virtual Environments*（SIBGRAPI 2002）

作用：

- PVS；
- visual importance；
- network transmission priority。

对 novelty 的关键约束：

> **visibility score / importance → transmission order 早已有先例。**

因此 scheduler 不是本文 headline novelty。

---

## *Streaming QSplat*（I3D 2001）

作用：

- 大模型的 view-dependent progressive transmission；
- 说明“只传当前更有价值内容”是经典问题。

---

## *View-Dependent Streaming of Progressive Meshes*（2004）

作用：

- current-view visual importance 指导 refinement transmission。

---

## *Streaming HLODs*（2004）

作用：

- hierarchy + LOD + priority streaming。

---

## *Fine-Grained Web3D Culling-Transmitting-Rendering Pipeline*（CGI 2023）

作用：

- 直接把 culling、network transmission、browser rendering 统一到 Web3D pipeline；
- 防止本文错误声称“首次把 PVS 与 Web streaming 结合”。

本文区别：

> visibility signal 在 detailed content residency 之前如何得到。

---

## OGC / Cesium 3D Tiles

作用：

- practical metadata-driven pre-content request decision；
- bounding volume；
- hierarchy；
- geometric error / SSE；
- viewer request volume。

本文定位：

> GCOF-PVS 不替代 3D Tiles selection，而是补充 pre-content occlusion relevance。

---

# 后续阅读每篇论文时统一记录的字段

| 字段 | 要回答的问题 |
|---|---|
| Task | 论文真正解决什么问题？ |
| Visibility type | from-point / from-region？ |
| Runtime scene representation | query 时需要什么？ |
| Preprocessing | 是否需要 scene-specific offline processing？ |
| Computation location | server / offline pipeline / renderer / client？ |
| Representation | geometry / proxy / voxel / froxel / PVS table / latent？ |
| Output | PVS / primitive visibility / continuous score？ |
| Learned? | 是否训练模型？ |
| Target-scene labels? | 新场景是否需要 visibility labels / retraining / calibration？ |
| Streaming use | filtering / selective transmission / ordering / none？ |
| Main cost | storage / preprocessing / runtime GPU / candidate count / startup bytes？ |
| 与 V5 区别 | 用一句最精确的话说明 |

这个矩阵比简单文献列表更重要。
