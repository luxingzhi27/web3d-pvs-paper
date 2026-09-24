# 07 — Evaluation Plan（实验与评价计划）

## 评价目标

实验部分围绕五类证据组织：

1. 主模型的 visibility quality；
2. 已有三组正式消融；
3. cross-scene generalization；
4. runtime / storage cost；
5. progressive delivery。

其中只有第 2 类属于 **ablation study**。后面三类用于验证方法的泛化性和系统价值，不引入新的模型消融。

---

# 7.1 Experimental Setup

统一说明：

- real / synthetic scenes；
- horizontal-disk view-cell protocol；
- 32 个离线 GT camera positions；
- candidate generation；
- train / calibration / validation / test split；
- baselines；
- metrics。

主评价以 unit-level visibility 为准。

## 主要指标

### Safety

- Weighted Recall；
- one-sided 95% LCB；
- Bad Cull。

### Culling Efficiency

- Useful Cull；
- CNOR；
- predicted count。

### Ranking Quality

- pose PR-AUC；
- prevalence；
- lift。

Accuracy、F1 等只作为补充指标，不作为主结论。

---

# 7.2 Main Visibility Results

使用完整 FULL 模型评价主要 visibility performance。

主 operating point 使用固定边界：

$$
\tau = 0.
$$

报告：

- per-scene Weighted Recall；
- worst-scene / scene-equal LCB；
- Bad Cull；
- Useful Cull；
- CNOR；
- PR-AUC。

calibrated threshold 仅作为诊断，用于分析模型的 ranking / upper-bound behavior，不替代 fixed-zero 主结果。

本节回答：

> 完整模型能否在保守 safety 下有效剔除不可见 units？

---

# 7.3 Ablation Study

消融实验只保留当前已经冻结的三组正式对照。

## 7.3.1 Occlusion Context

比较：

- FULL
- GEOMETRY_FIELD

两者保持 structured field、query head 和训练目标一致，主要区别是是否使用 surrounding potential-occluder relations。

回答：

> target-local geometry 是否足以支持有效的遮挡预测，还是必须显式编码 surrounding occlusion context？

重点比较：

- Weighted Recall / LCB；
- Useful Cull；
- CNOR。

---

## 7.3.2 Structured Representation

比较：

- FULL
- GENERIC_RELATION_28

两者均使用 relation evidence，并保持相同的 runtime context budget；区别在于：

- FULL：structured analytic directional field；
- GENERIC_RELATION_28：unrestricted 28D latent。

回答：

> structured directional field 是否比同容量 generic latent 更适合作为 compact visibility representation？

---

## 7.3.3 Conservative Learning Objective

比较：

- FULL
- PBCE_OBJECTIVE

两者使用相同 Full representation，只改变训练目标。

回答：

> 普通 pose-balanced classification objective 是否能够产生与 conservative PVS 相适应的稳定 operating boundary？

重点报告：

- fixed-zero Weighted Recall / LCB；
- Useful Cull / CNOR；
- PR-AUC；
- calibrated diagnostic。

如有必要，可补充 score distribution 或 training dynamics，但这些不构成新的消融实验。

---

# 7.4 Cross-Scene Generalization

本节独立于消融实验。

## LOSO

使用已有 LOSO protocol，评价 frozen model 在 held-out real scene 上的迁移能力。

关注：

- fixed operating point；
- visibility safety；
- culling efficiency；
- ranking quality。

## External Blind Holdout

在 architecture、checkpoint 和 protocol 冻结后，对未参与设计和训练的 external scene 进行评价。

该场景不得用于：

- architecture selection；
- checkpoint selection；
- hyperparameter tuning；
- threshold tuning。

本节回答：

> geometry-compiled visibility representation 能否在不使用 target-scene visibility fitting 的情况下迁移到未见场景？

---

# 7.5 Runtime and Storage Cost

本节属于系统评价，不是网络消融。

针对 V5 实际 runtime implementation 报告：

- total visibility asset size；
- bytes / unit；
- shared model size；
- offline compilation time；
- WebGPU latency；
- WASM latency；
- latency vs candidate count。

与可行的 geometry/proxy-based baseline 比较时，重点回答：

> 为了在 detailed geometry residency 前获得 visibility，compact visibility asset 的存储与运行代价是否合理？

V4 runtime 结果不能作为 V5 正式结果。

---

# 7.6 Progressive Delivery

本节验证 visibility signal 是否对 progressive Web delivery 有实际价值。

所有方法必须使用相同的 candidate unit set。

可比较：

- baseline / original order；
- distance / projected-area heuristic；
- AABB-based baseline；
- HZB visible-first；
- V5 visibility score；
- GT visibility oracle。

如实验引入单位传输成本，则报告：

- Cost/Bytes@95；
- Cost/Bytes@99；
- Cost/Bytes@99.9；
- waste-before-99；
- 固定带宽下的 time-to-coverage。

如进一步执行真实 scheduler replay，可在本节末报告：

- p50 / p95；
- delivered payload；
- waste；
- time-to-coverage。

需要明确区分 scheduler replay 与完整 browser rendering latency。

---

# 当前实验边界

## 已有正式消融

仅包括：

1. FULL vs GEOMETRY_FIELD
2. FULL vs GENERIC_RELATION_28
3. FULL vs PBCE_OBJECTIVE

不再额外设计新的 architecture / hyperparameter ablation，除非后续审稿或实验结果明确需要。

## 独立评价

以下内容不是消融：

- fixed-zero main results；
- LOSO；
- external blind holdout；
- V5 runtime / storage；
- progressive delivery；
- scheduler replay。

这些分别用于验证主模型、泛化能力和系统价值。
