# 11 — 面向 CGF / 图形学论文的写作规则

## 1. 从 Graphics Problem 开始，而不是从 Neural Architecture 开始

优先：

> Progressive Web3D 缺少 detailed content residency 之前的 occlusion signal。

避免：

> We propose a novel neural network with three modules.

---

## 2. 所有 Module 都必须由“设计约束”解释

每个模块都问：

1. 为什么系统问题要求它存在？
2. 更简单的 control 是什么？
3. 消融如何验证这个设计？

例如：

### Relation context

不是因为“GNN 很强”。

而是因为：

> target 自身形状不知道周围谁会挡住它。

### Structured directional field

不是因为“SH 很流行”。

而是因为：

> 需要一个 compact、fixed-size、continuous、cheap-to-query 的 runtime representation。

### Shared zero boundary

不是因为“constrained optimization 更高级”。

而是因为：

> 新场景如果必须有 label 才能重新 calibration，就无法形成真正的 label-free deployment story。

---

## 3. 严格区分 Fact / Hypothesis / Result

### Fact

> 我们使用固定一阶方向 basis。

### Hypothesis

> 结构化 field 可能比 unrestricted latent 更容易跨场景保持稳定的 directional behavior。

### Result

> Full 在某指标上显著优于 Generic-28。

不能把 hypothesis 在 Method 里提前写成已经证明的事实。

---

## 4. 避免 Novelty Inflation

不要写：

- first visibility-guided streaming；
- first neural visibility；
- first online PVS；
- first pre-content selection。

真正创新是：

> 多个已有方向的一个特定交叉 operating point。

---

## 5. 消融必须 Hypothesis-Driven

好的写法：

> Full vs Geometry Field tests whether surrounding occlusion context is necessary.

不够好的写法：

> Removing relation module decreases performance by X%.

前者回答科学问题，后者只是模型拆模块。

---

## 6. Safety-First

对于 conservative PVS：

正文排序：

1. Weighted Recall；
2. LCB；
3. Bad Cull；
4. culling efficiency。

不要让：

- Accuracy；
- F1；

成为主 headline。

---

## 7. “Geometry”必须精确

优先使用：

- detailed streamed geometry；
- runtime scene representation；
- proxy geometry；
- compact metadata。

不要随便写：

> “all prior methods require the full detailed mesh”。

很多现代 PVS 只需要一种足够的 scene representation，不一定是原始完整 mesh。

---

## 8. “Generalization”必须精确

严格区分：

- shared in-domain；
- LOSO；
- external blind holdout；
- target-calibrated diagnostic。

如果使用了 target calibration，就不要叫：

> zero-shot decision boundary transfer。

---

## 9. V4 与 V5 的证据不能混

V4 可以是：

- legacy deployed model；
- historical system baseline；
- engineering reference。

V5 是论文方法。

除非 V5 真正完成对应 runtime / streaming 实验，否则不能把：

- V5 model metrics；
- V4 browser runtime；

拼成同一个“ours”。

---

## 10. 每一节都要自然引向下一节

建议：

### Related Work

收束到：

> missing operating point。

### Problem Formulation

收束到：

> representation requirements。

### GCOF Method

收束到：

> representation 有了，但需要一个无需 target calibration 的 deployment boundary。

### Shared Boundary Learning

收束到：

> 得到可部署 score。

### Web Integration

收束到：

> 需要验证 safety、runtime 和 streaming utility。

这样整篇论文才是一条完整论证链，而不是若干独立章节。
