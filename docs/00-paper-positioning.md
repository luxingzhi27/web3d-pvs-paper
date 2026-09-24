# 00 — 论文定位（重新按图形学论文叙事层级整理）

## 当前最核心的论文问题

> **在详细场景几何尚未 resident 于 Web client 时，如何获得可用于 progressive delivery 的 from-region occlusion visibility？**

本文最重要的不是某个 loss 或某个网络模块，而是一个不同的 **visibility operating point**：

[
\text{before detailed content residency}.
]

---

# 一、Primary Story：Pre-Geometry Visibility

大型 Web3D 场景需要逐步传输。

现有客户端在详细 content 尚未到达时，可以使用：

- frustum；
- bounding volume；
- hierarchy；
- distance；
- SSE / geometric error；

进行 content selection。

但 occlusion-aware relevance 通常依赖额外 scene representation。

因此形成 bootstrap gap：

```text
visibility is useful for deciding what geometry to request
                     ↑
                     │
conventional visibility needs a scene representation
that may itself still be waiting to be transferred
```

这应当是整篇论文的第一层问题。

---

# 二、Primary Insight：Geometry-Compiled Visibility Representation

content-preparation/server side 在发布场景前已经拥有 geometry。

因此本文的核心 idea 不是：

> 在 client 再恢复一份完整 scene representation。

而是：

> **把与遮挡相关的 scene context 在 offline stage 编译为一个远小于 detailed geometry 的 representation，并让 client 可以直接对 view region 查询。**

抽象流程：

```text
geometry
   ↓ offline shared compiler
compact visibility representation
   ↓ pre-transfer
client view-region query
   ↓
visibility relevance
   ↓
resource delivery
```

这才是 GCOF-PVS 的主 identity。

---

# 三、Secondary Technical Idea：Structured Occlusion Field

V5 的具体技术实现是：

- local surface geometry；
- geometry-only potential occluder relations；
- relation compiler；
- compact directional field；
- analytic region query；
- small visibility head。

它们共同服务于一个目标：

> **用很小的 runtime representation 保留足够的 geometry-dependent occlusion context。**

论文 Method 的重点应该是这种 representation design，而不是强调使用了哪一种 NN building block。

---

# 四、Secondary Technical Idea：Conservative Learning

PVS 的 false negative 和 false positive 代价天然不对称：

- false negative → visible content missing / delayed；
- false positive → extra transfer / processing。

因此 safety-aware / conservative learning 是必要的。

但它在论文叙事中的层级应当是：

> **让 compact representation 更适合 PVS deployment 的训练机制。**

而不是和“pre-geometry visibility”并列成为整篇论文的第一主线。

当前 fixed-zero / robust-boundary 设计可以是 Method 中较重要的一节，也可以成为 contribution 的一部分，但不应反过来主导 Introduction。

---

# 五、Scheduler 的定位

visibility-guided transmission 早已有先例。

因此：

> scheduler 本身不是 headline novelty。

本文更准确的系统贡献是：

> **为 progressive Web3D 提供一个 detailed-content residency 之前即可获得的 occlusion-aware relevance signal。**

unit-level filtering 与 progressive content ordering 是这个 signal 的自然应用。

---

# 六、当前建议的 3 个贡献层级

## Contribution 1 — Pre-Geometry Visibility via Geometry Compilation

提出一种面向 progressive Web3D 的 geometry-compilation approach，使 client 在 detailed geometry 到达前即可查询 from-region visibility relevance。

## Contribution 2 — Compact Structured Representation and Query

将 local geometry 与 geometry-only potential-occluder context 编译成紧凑 directional field，并设计轻量 region query；训练采用 conservative objective 适配 PVS 的 asymmetric error cost。

## Contribution 3 — Web3D System Evaluation

在 progressive resource delivery 中验证该 signal，并从 visibility safety/efficiency、asset/runtime cost 与 streaming utility 多维评价。

如果 external blind holdout 最终非常强，可以加入：

> no target-scene fitting

作为 Contribution 1/2 的重要性质。

---

# 七、不要把这些内容抬成 headline

除非最终实验显示它们本身就是 strongest novelty，否则以下内容不应在 Introduction 中占独立故事线：

- fixed zero threshold；
- target calibration protocol；
- 10-domain SmoothMax；
- shared dual；
- PBCE；
- exact 32D / 28D dimensionality；
- WebGPU implementation details。

这些应该分别属于：

- Training / Method；
- Evaluation protocol；
- Implementation。

---

# 八、关键 reviewer 问题

最终论文真正需要说服 reviewer 的不是：

> “这个 neural classifier accuracy 高不高？”

而是：

1. 为什么 pre-geometry visibility 是一个有意义、未被当前 PVS operating point 直接解决的问题？
2. 一个 compact compiled asset 是否真的比 proxy/coarse geometry 更有价值？
3. geometry-only compilation 是否保留了足够的 occlusion information？
4. client query 的 safety / efficiency / runtime 是否足够？
5. 这个 signal 是否真的改善 progressive delivery？

所有 Method 和实验都应该围绕这五个问题组织。
