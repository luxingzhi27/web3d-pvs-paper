# 01 — Introduction 写作方案（按图形学顶会叙事重新整理）

## 为什么要重新整理

当前 V5 代码包含很多重要训练机制，例如：

- fixed-zero boundary；
- domain-robust risk；
- SmoothMax；
- shared dual；
- PBCE control。

这些机制对于模型成立很重要，但**代码里的重要性不等于 Introduction 中的叙事优先级**。

参考 NeuralPVS、Trim Regions、Disocclusion Buffer 等图形学论文的写法，Introduction 更应该围绕：

```text
任务为什么重要
    ↓
现有方法真正缺什么
    ↓
关键 insight
    ↓
方法高层概览
    ↓
贡献与结果
```

而不是把 Method 中的每个模块提前讲一遍。

---

# 推荐的 Introduction 主线

## 第 1 段：从 From-Region Visibility 与 Streaming 需求开始

先建立一个成熟、被图形学社区接受的问题背景：

- visibility determination 是实时图形学中的基础问题；
- PVS 可以避免处理明显不可见的内容；
- from-region PVS 比 from-point visibility 更适合：
  - camera motion prediction；
  - prefetching；
  - remote rendering；
  - progressive / streaming rendering。

这里的写法可以借鉴 Trim Regions 和 NeuralPVS：

> 不需要先讲 Web 技术栈，而是先说明 from-region visibility 为什么天然适合“未来一小段相机运动范围内的内容准备”。

然后再落到 Web3D：

> 对大型 Web 场景而言，客户端尤其需要提前决定哪些资源应该优先到达。

### 这一段的目标

让 reviewer 先接受：

$$
	ext{from-region visibility}
$$

和：

$$
	ext{progressive delivery}
$$

之间是自然关系，而不是我们人为拼接两个课题。

---

## 第 2 段：现有方法的核心 operating assumption

这段不要泛泛写“现有 PVS 太慢”。

更准确地梳理：

### Classical precomputed PVS

把 geometry reasoning 放到 offline，但通常需要：

- scene-specific preprocessing；
- view-cell-specific visibility storage。

### Modern online PVS

如：

- Camera Offset Space；
- Guided Visibility Sampling++；
- Trim Regions；
- Disocclusion Buffer。

它们显著降低了 online from-region PVS 的成本。

### NeuralPVS

进一步把 runtime PVS computation 学习成：

$$
	ext{runtime geometry representation}
ightarrow
	ext{PVS}.
$$

真正要指出的共同点不是“它们都慢”，而是：

> **执行 visibility computation 的一方通常已经能够访问一份足够表达遮挡关系的 scene representation。**

这句话应该成为本文最重要的 literature transition。

---

## 第 3 段：提出 Web Streaming 中的 Bootstrap Gap

现在再把 Web client 的特殊约束说清楚：

> 在 progressive Web streaming 中，detailed geometry 本身就是尚未传输的对象。

因此出现：

$$
	ext{visibility helps decide what to download}
$$

但：

$$
	ext{visibility computation often assumes scene content is already available}.
$$

这才是本文真正的问题。

建议定义：

> **pre-geometry from-region visibility**

一句话研究问题：

> Can a client estimate occlusion-aware from-region relevance before the detailed scene representation used by conventional visibility algorithms is resident?

### 注意

这一段只定义**系统/图形学问题**。

不要在这里提：

- $z=0$；
- calibration；
- dual；
- SmoothMax；
- loss。

---

## 第 4 段：核心 Insight — Compile Visibility-Relevant Scene Context

这一段应该是全文最关键的方法转折。

关键 observation：

> server / content-preparation pipeline 在发布场景前本来就拥有完整 geometry。

所以我们没有必要让 browser 在 runtime 再恢复一份完整遮挡表示。

我们可以问：

> 能否把 geometry-dependent occlusion context 预先编译成一个比 detailed geometry 小得多、但仍可以被任意当前 view-region 查询的 representation？

核心 pipeline：

```text
scene geometry
    ↓ offline geometry compilation
compact visibility representation
    ↓ transfer once / early
client-side from-region query
    ↓
visibility relevance
    ↓
progressive resource delivery
```

这才是 Introduction 中真正应该强调的 method insight。

---

## 第 5 段：方法高层概览

这一段应该类似 NeuralPVS Introduction 里对 froxelization + CNN 的概览，或者 Trim Regions 对 rasterized shrinking 的概览。

只讲**必要概念**，不要讲实现数字。

建议：

> 我们提出 GCOF-PVS。共享模型首先把每个 renderable unit 的局部表面几何及其由纯几何构造的 potential-occluder context 编译为紧凑 directional occlusion field。运行时，Web client 只需对当前 view region 在该 field 上进行少量解析查询，再通过一个轻量预测头得到 per-unit visibility score。该 per-unit visibility score 可直接用于 unit-level filtering，并作为 progressive content ordering 的 occlusion-aware relevance signal。

如果最终 cross-scene evidence 足够强，可以再加一句：

> The same frozen compiler can be applied to new scenes without target-scene visibility fitting.

但在 blind-holdout 结果没有冻结前，不要过度强调。

---

## 第 6 段：Safety / Training 只高层带过，不单独成段

原先这里单独写 shared boundary 是不合适的。

更好的做法是在上一段或这一段用 **1–2 句**说明 PVS 的 asymmetric error：

> 对 streaming 而言，漏掉真正可见的 geometry 比额外保留不可见 geometry 更严重。因此训练和评价均采用 safety-first 的保守目标，在减少冗余内容的同时限制 visible misses。

如果最终 shared-zero-boundary 结果确实成为很强的贡献，可以再加一句：

> We additionally train the predictor so that a common operating boundary remains conservative across heterogeneous source domains.

到此为止。

不要在 Introduction 出现：

- $z=0$ 数学定义；
- target calibration；
- SmoothMax；
- shared dual；
- (ho=0.1)。

这些属于 Method。

---

## 第 7 段：Contributions

参考 Trim Regions / NeuralPVS / Disocclusion Buffer，contributions 应该具体、可验证，而不是“问题定义 + 若干模块”的流水账。

当前更推荐 3 点，而不是 4 点。

### Contribution 1 — Geometry-Compiled Visibility Representation

> 提出一种面向 pre-geometry from-region visibility 的 geometry-compilation framework，将局部几何与纯几何 potential-occluder context 编译成紧凑的 per-unit directional visibility representation，使客户端无需 detailed scene geometry 即可进行 region query。

这一点同时包含：

- research operating point；
- GCOF 的核心思想。

比单列“我们提出新问题”更像顶会 graphics contribution。

### Contribution 2 — Compact Structured Query

> 设计一个结构化低阶 directional field 与轻量 from-region query，在较小 runtime asset 下编码遮挡上下文，并通过 conservative learning objective 适配 PVS 中 false negative / false positive 的非对称代价。

这里可以把 shared-boundary learning 作为**方法组成部分**，而不是 Introduction 的独立故事线。

### Contribution 3 — Progressive Web3D Evaluation

> 将 unit-level visibility signal 接入 progressive Web delivery，并从 PVS safety/efficiency、runtime asset cost、client inference cost 与 streaming utility 多个维度进行评价。

如果最终 blind holdout 很强，可以在 Contribution 1/2 中补：

> without target-scene fitting。

不要在证据不足时先写成 headline。

---

# 更接近顶会图形学论文的 Introduction 节奏

最终正文大概应该是：

```text
P1  PVS / from-region visibility 为什么重要，尤其对 streaming
P2  classical + online + learned PVS 已经做得很好，但通常假设 visibility processor 有 scene representation
P3  Web progressive streaming 的 bootstrap gap：要用 visibility 决定尚未 resident 的 geometry
P4  Insight：把 occlusion context offline compile 成 compact visibility representation
P5  GCOF-PVS 高层 pipeline + unit-level Web query / content priority
P6  conservative PVS requirement + 一句话 training philosophy
P7  contributions + headline results
```

也可以把 P5/P6 合并，使全文只有 6 个核心段落。

---

# Introduction 中应该删掉/弱化的内容

## 删除独立段落

- “为什么 shared boundary 是方法的一部分”
- “为什么 target calibration 会削弱 zero-shot”
- “十个风险域”
- “fixed-zero loss”

## 只保留一句高层描述

- conservative / safety-aware training；
- cross-scene deployment（若实验支持）。

## 完全移到 Method

- $h_{\mathrm{keep}}$、$h_{\mathrm{miss}}$；
- $J_{\mathrm{extra}}$；
- blended safety risk；
- SmoothMax；
- shared dual；
- PBCE control；
- calibration protocol。

---

# 与参考论文的对应关系

## NeuralPVS 的写法启示

NeuralPVS 的 Introduction 先讲：

1. PVS 与 from-region 用途；
2. precomputation / online method 的限制；
3. neural method 的机会；
4. froxel + CNN 的方法概览；
5. synthetic generalization；
6. repulsive loss 一句话；
7. contributions。

它**没有**把 loss 的完整设计搬到 Introduction。

我们的对应写法应该是：

1. PVS / streaming；
2. content-residency assumption；
3. pre-geometry bootstrap gap；
4. geometry compilation insight；
5. GCOF-PVS overview；
6. conservative training 一句话；
7. contributions。

## Trim Regions 的写法启示

Trim Regions 非常典型：

1. 先定义 from-point / from-region；
2. 说明 online from-region 对 streaming 的价值；
3. 指出 existing shrinking 在 general 3D 上为什么难；
4. 提出 rasterization 这一核心 insight；
5. 列具体 contributions。

这说明：

> Introduction 的关键不是把整个系统都讲完，而是找到“为什么已有方法在我们关心的 operating point 上还缺一块”以及“我们的核心 insight 是什么”。

## Disocclusion Buffer 的写法启示

它同样是：

1. PVS utility；
2. 当前 online methods 的真正 bottleneck；
3. 一个反转问题 formulation 的核心 idea；
4. 高层 method；
5. 两个 contributions；
6. headline performance。

这进一步说明：

> 具体训练技巧/实现机制只有在它们就是论文主要 conceptual insight 时，才应该进入 Introduction 主叙事。

---

# 当前结论

**shared boundary 仍然重要，但应该从“Introduction 的一整段核心动机”降级为“Method 中的关键训练设计 + Introduction 中一句 safety-oriented training 概述”。**

论文真正的 Introduction headline 应该集中在：

$$
\boxed{
\text{visibility before detailed geometry residency}
}
$$

以及：

$$
\boxed{
\text{compile occlusion context into a compact client-queryable representation}
}
$$

这两个概念上。
