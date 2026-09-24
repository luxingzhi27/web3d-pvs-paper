# 论文总体大纲

## 暂定标题

**Pre-Geometry Visibility for Progressive Web3D via Geometry-Compiled Occlusion Fields**

---

## 1. Introduction

说明大型 Web3D 场景需要渐进式传输，而现有 pre-download selection 主要依赖视锥、距离、层次结构和 SSE，缺少遮挡相关性。

指出 from-region PVS 与 learned visibility 已能高效估计可见性，但通常假设执行可见性计算的一方已经拥有足够的场景表示。本文关注一个不同的 operating point：**在详细几何尚未到达客户端之前估计区域可见性。**

核心思想是将 geometry-dependent occlusion context 离线编译成紧凑的可见性表示，客户端只需对当前 view-cell 做轻量查询。

主要贡献：

1. 面向 progressive Web3D 的 pre-geometry from-region visibility 方法；
2. geometric occlusion context 到 compact directional field 的编译与查询；
3. 面向 unit-level culling 与 progressive delivery 的系统验证。

---

## 2. Related Work

### 2.1 From-Region Visibility and PVS

介绍 classical precomputed PVS、online PVS、Trim Regions、Camera Offset Space、Disocclusion Buffer 等，重点比较 query 时需要的场景表示和计算位置。

### 2.2 Learned Visibility

重点讨论 NeuralPVS、Neural Visibility of Point Sets、NVGS 等，区分 runtime geometry-to-PVS inference 与本文的 offline geometry-to-compact-visibility compilation。

### 2.3 Visibility-Aware Streaming

介绍 networked PVS、Smart Visible Sets、view-dependent streaming、Web3D/3D Tiles 等，说明 visibility-guided delivery 已有基础，本文重点是 **visibility signal 在 detailed content residency 之前如何获得**。

---

## 3. Method

### 3.1 Problem Setting and View-Cell Protocol

以 renderable unit 为唯一 visibility / culling 单位。

每个 view-cell 定义为固定观察方向下的水平圆盘。离线使用 32 个圆盘内 camera positions 生成 sampled regional GT，candidate set 由独立的 66° AABB frustum 构造。

### 3.2 Local Geometry Encoding

将每个 unit 的局部表面几何编码为共享的紧凑 descriptor，不使用 scene ID 或 per-instance learned memory。

### 3.3 Potential-Occluder Relations

基于 unit AABB、固定方向和 projected overlap 构造 potential-occluder relations，用于描述 target unit 周围的遮挡上下文。

### 3.4 Geometry-Compiled Directional Survival Field

使用 relation compiler 聚合 surrounding occlusion context，并将离散方向 responses 投影为连续可查询的低阶 directional field。该 field 以单调 survival function 表示 target 沿给定方向和距离的结构化遮挡状态，而不是最终 visibility probability。

### 3.5 From-Region Query

在水平圆盘中心与 8 个圆周 support points 上解析查询 survival field，汇总 center/max/mean/min regional occlusion statistics，并与 local geometry 和 query geometry 一起预测 per-unit visibility score。

### 3.6 Conservative Visibility Learning

考虑 PVS 中 false negative 与 false positive 的非对称代价，在减少冗余保留的同时约束 visible misses。

---

## 4. Progressive Web3D Integration

### 4.1 Offline Compilation

对场景执行 local geometry encoding、occlusion-relation construction 和 field compilation，生成 compact visibility asset。

### 4.2 Runtime Query

根据当前相机建立 horizontal-disk view-cell，通过后退 66° AABB frustum 获得 candidate units，并执行一次 batched visibility query。

### 4.3 Unit-Level Culling and Ordering

使用预测结果形成 regional PVS，并结合当前 60° display frustum 完成即时 unit filtering；连续 visibility score 同时用于 progressive content ordering。

---

## 5. Evaluation

### 5.1 Experimental Setup

介绍场景、统一 view-cell/GT 协议、数据划分、baselines 和评价指标。

### 5.2 Main Visibility Results

评价完整 FULL 模型在固定 operating point 下的 safety 与 culling efficiency，重点报告 Weighted Recall、LCB、Bad Cull、Useful Cull 和 CNOR。

### 5.3 Ablation Study

只包含当前已有的三组正式消融：

1. FULL vs GEOMETRY_FIELD：验证 surrounding occlusion context；
2. FULL vs GENERIC_RELATION_28：验证 structured field；
3. FULL vs PBCE_OBJECTIVE：验证 conservative learning objective。

### 5.4 Cross-Scene Generalization

使用 LOSO 与 external blind holdout 评价 frozen model 在未见场景上的迁移能力。

### 5.5 Runtime and Storage Cost

报告 V5 visibility asset 大小、bytes/unit、WebGPU/WASM latency 和 candidate-count scaling，并与可行的 geometry/proxy-based baseline 比较。

### 5.6 Progressive Delivery

在相同 candidate unit set 下比较不同 ordering 策略，报告达到目标 visible coverage 所需的传输成本、时间和冗余；如完成真实 scheduler replay，则作为本节补充系统结果。

---

## 6. Discussion and Limitations

讨论：

- per-unit asset 与 candidate-count scaling；
- AABB proxy 对复杂遮挡体的局限；
- low-order directional field 的表达能力；
- static-scene assumption；
- safety–efficiency trade-off；
- cross-scene generalization 的适用范围。

---

## 7. Conclusion

总结 pre-geometry visibility 问题及 geometry-compilation 思路，强调紧凑 visibility representation 可以在 detailed geometry residency 之前为 Web progressive rendering 提供 occlusion-aware unit relevance。
