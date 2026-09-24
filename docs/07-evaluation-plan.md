# 07 — Evaluation Plan（实验与评价计划）

## 本节目标

实验部分应该围绕明确的 research questions 展开，而不是把仓库里能算出来的所有 metric 都堆进正文。

建议最终按 RQ 组织。

---

# RQ1 — 一个固定零边界能否跨场景保持保守安全？

主协议：

[
\tau=0.
]

必须报告：

- per-scene weighted recall；
- one-sided 95% LCB；
- scene-equal mean；
- worst-scene；
- CNOR；
- Useful Cull；
- Bad Cull。

主要比较：

- Full；
- Geometry Field；
- Generic Relation 28；
- PBCE。

calibrated threshold 只作为 diagnostic，不应该取代 fixed-zero 主结论。

### 推荐图

每个场景分别画：

- visible score distribution；
- invisible score distribution；
- 垂直的 (z=0) 决策线。

这样最直接展示：

> shared boundary 是否真的具有跨场景一致语义。

---

# RQ2 — Surrounding Occlusion Context 是否必要？

比较：

- Full；
- Geometry Field。

控制：

- structured field 不变；
- region query 不变；
- robust objective 不变；
- Geometry Field 尽量做 capacity match。

核心问题：

> **target 自身几何是否足以进行有效的 conservative occlusion prediction？**

希望实验最终区分：

- safety；
- efficiency。

如果 Geometry Field 很安全但 CNOR / Useful Cull 明显下降，应该解释为：

> local target geometry 可以提供 conservative prior，但 surrounding occlusion relations 才提供真正有效的遮挡辨别能力。

---

# RQ3 — Structured Field 是否真的有价值？

比较：

- Full；
- Generic Relation 28。

两者控制：

- 都拥有 relation information；
- runtime context 都是 28 个值；
- 使用相同 robust objective。

区别：

- Full：structured analytic survival field；
- Generic-28：unrestricted 28D relation latent。

核心问题：

> **收益是来自 relation context 本身，还是来自结构化、单调、可解析查询的 field representation？**

这是 V5 最强的 representation ablation 之一。

---

# RQ4 — Shared Conservative Boundary Objective 是否必要？

比较：

- Full + robust-boundary objective；
- PBCE objective。

保持 representation 完全一致。

报告：

- fixed-zero WR；
- WR LCB；
- calibrated diagnostic；
- PR-AUC；
- score distributions；
- robust risk / shared dual dynamics（如果结果足够稳定）。

核心问题：

> **普通 balanced classification 是否会自然产生一个可跨场景部署的 operational boundary？**

如果 PBCE ranking 尚可、但 zero-boundary safety 失败，那么论文要强调：

> 任务不只是学会可见性排序，而是学会一个可以直接跨场景解释的 conservative decision boundary。

---

# RQ5 — 不使用 Target-Scene Visibility Labels 时能否泛化？

建议分两类协议。

## External Blind Holdout

这是最适合正文 headline 的 generalization evidence。

必须先冻结：

- architecture；
- checkpoint；
- hyperparameters；
- (z=0) 决策规则。

然后才：

- 转换 blind scene；
- 只读取 geometry 编译资产；
- 最后读取 GT 做评价。

blind holdout 绝不能用于：

- variant selection；
- checkpoint selection；
- threshold tuning；
- loss design。

## LOSO

作为更系统的 cross-scene secondary evidence。

必须清楚区分：

### representation transfer

是否不重新训练即可得到有效 ranking/score。

### threshold transfer

是否仍需要目标场景 calibration。

如果论文主张 label-free deployment，headline 结果不应依赖 target calibration。

---

# RQ6 — Pre-Geometry Visibility 本身的资产和运行成本是多少？

必须重新针对 V5 测。

报告：

- total runtime visibility asset bytes；
- bytes / unit；
- shared-model bytes；
- offline compilation time；
- WebGPU latency；
- WASM latency；
- candidate-count scaling。

候选规模可以按真实数据分布设置，例如：

- 1k；
- 5k；
- 10k；
- 20k；
- 50k。

公平 baseline：

- Geometry-shell HZB；
- AABB/proxy metadata；
- 其他可以在 geometry residency 前使用的轻量表示。

注意：

> V4 runtime result 不能直接作为 V5 runtime result。

---

# RQ7 — Visibility Signal 是否真正改善 Progressive Delivery？

使用 V5 score sidecar。

threshold-free ranking 至少比较：

- original / baseline order；
- projected area / byte；
- AABB model；
- HZB visible-first；
- V5 neural score；
- V5 cost-aware neural score；
- GT utility / byte oracle。

主要指标：

- Bytes@95；
- Bytes@99；
- Bytes@99.9；
- waste-before-99；
- 固定带宽下的时间换算。

所有 ranking 方法必须共享：

> **同一个 candidate GLB 集合。**

否则结果不可比较。

---

# RQ8 — 在真实 Scheduler 状态机里是否仍然有收益？

要求：

- cold cache；
- 相同 scheduler state machine；
- 相同 bandwidth；
- 相同 pose set；
- 唯一变化是 priority source。

报告：

- time-to-coverage；
- downloaded bytes；
- waste；
- p50 / p95。

必须精确描述测到的是什么。

例如：

> Node 里的真实 GLB 下载 + scheduler replay，不等于完整浏览器 first-frame rendering latency。

如果以后补了真实 browser render path，再单独称：

> end-to-end browser latency。

---

# 指标层级

## 第一层：Safety

正文最优先：

- Weighted Recall；
- one-sided 95% LCB；
- Bad Cull；
- image-visible misses / wrong IDs（如果正式完成）。

## 第二层：Occlusion Efficiency

- CNOR；
- Useful Cull；
- predicted count。

## 第三层：Threshold-Free Ranking Quality

- pose PR-AUC；
- prevalence；
- lift。

## 第四层：System Utility

- startup bytes；
- runtime；
- Bytes@x；
- waste；
- scheduler replay。

不要让：

- Accuracy；
- F1；

成为主叙事。

---

# 当前 V5 结果状态（基于论文规划快照 4faca3c）

当前 validation 已经支持一个较清晰的初步故事：

- Full 在 fixed-zero 下具有较高 safety；
- Geometry Field safety 可以很高，但 culling efficiency 更差；
- Generic-28 是非常有意义的 structured-vs-unstructured control；
- PBCE 并不会自然产生所需的跨场景安全零边界。

但这些仍属于：

> formal validation / training matrix evidence。

在随机重复、blind holdout、frozen test、V5 browser system evidence 完成前，不冻结最终论文数字。
