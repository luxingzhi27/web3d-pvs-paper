# 07 — 实验与评价

本节与总体大纲第 6 节对应。评价分别刻画可见内容保留、遮挡剔除、排序质量、跨场景迁移以及运行和传输成本。以“可见”为正类，所有方法使用相同候选单元与冻结的数据划分。

## 6.1 实验设置与评价指标

### 6.1.1 评价对象与聚合层级

对观察区域或 pose $p$，记候选集合为 $\mathcal C_p$，区域 GT 为 $\mathcal G_p$，预测保留集为 $\widehat{\mathcal G}_p$。令 $n_p=|\mathcal C_p|$，则：

$$
TP_p=|\widehat{\mathcal G}_p\cap\mathcal G_p|,\quad
FP_p=|\widehat{\mathcal G}_p\setminus\mathcal G_p|,
$$

$$
FN_p=|\mathcal G_p\setminus\widehat{\mathcal G}_p|,\quad
TN_p=|\mathcal C_p\setminus(\widehat{\mathcal G}_p\cup\mathcal G_p)|.
$$

$TN_p$ 是正确剔除的不可见单元数，$FN_p$ 是错误剔除的可见单元数。数据采用第 3 节的后退视锥与区域采样协议，候选集合独立生成，并满足采样 GT 的包含检查。

Aggregate 指标先合并全部 pose 的计数或权重再计算；pose-macro 指标先在每个 pose 内计算，再等权平均。两种口径分别反映整体单元表现和平均观察区域表现，表头明确区分。跨场景均值在各场景指标计算完毕后等权汇总，不直接拼接所有场景的候选。

### 6.1.2 Weighted Recall：视觉贡献加权召回率

单元数量与画面贡献并不等价。普通召回率对占据少量屏幕像素的单元和大面积可见单元赋予相同权重，难以区分二者的遗漏代价。本文以区域采样中单元的最大屏幕覆盖贡献 $w_{pi}$ 衡量其视觉重要性，采用加权召回率（Weighted Recall，WR）：

$$
\operatorname{WR}_{\mathrm{agg}}=
\frac{\sum_p\sum_{u_i\in\mathcal G_p\cap\widehat{\mathcal G}_p}w_{pi}}
{\sum_p\sum_{u_i\in\mathcal G_p}w_{pi}},
\qquad
w_{pi}=\max_{m=1,\ldots,32}w_i(\mathbf c_{pm}).
$$

WR 衡量被保留的可见贡献比例，使大量屏幕贡献很小的遗漏不会与同数量的大面积缺失被等价计分。这里的“小”指当前观察条件下的屏幕贡献，而非物体的世界尺寸；具有语义重要性的小单元也可能不可忽略。因此同时报告普通 Visible Recall 及其对应的 False Occlusion Rate，以保留对遗漏单元数量的诊断。WR 是单元级视觉贡献代理，不等同于最终画面的像素误差。

单侧 95% 置信下界（Lower Confidence Bound，LCB）按冻结协议对 pose 重采样后计算，用于表达 WR 的统计不确定性。LCB 不表示每个 pose 均达到该召回率，也不是连续视域的保守性证明。

### 6.1.3 CNOR：候选归一化遮挡召回率

不同 pose 的候选规模和负样本数量可能相差很大。直接汇总的遮挡召回率 $\sum_p TN_p/\sum_p(TN_p+FP_p)$ 会使负候选数量较多的 pose 获得更大权重，即使其主要区别只是候选集合更大。本文采用**候选归一化遮挡召回率（Candidate-Normalized Occlusion Recall，CNOR）**：

$$
\operatorname{CNOR}=
\frac{\sum_{p:n_p>0}TN_p/n_p}
{\sum_{p:n_p>0}(TN_p+FP_p)/n_p}.
$$

该指标先将各 pose 的正确剔除量和理论可剔除量除以候选数，再计算总体机会利用率，从而消除候选绝对规模对该 pose 贡献的直接放大。令 $N_p=TN_p+FP_p$、$\operatorname{OR}_p=TN_p/N_p$，则：

$$
\operatorname{CNOR}=
\frac{\sum_p(N_p/n_p)\operatorname{OR}_p}
{\sum_pN_p/n_p}.
$$

因此，CNOR 是以“负样本占候选的比例”为权重的 pose 遮挡召回率，**不是所有 pose 严格等权的遮挡召回平均值**。它衡量归一化剔除机会中有多少被模型实现，而不评价可见内容是否被误剔；后者由 WR 和普通召回率共同约束。

空候选 pose 和没有负样本的 pose 不贡献剔除机会。若整个集合均无负样本，当前实现返回 1，并同时报告零剔除机会；该退化值不作为有效剔除能力的证据。

### 6.1.4 互补指标及用途

| 指标 | 定义或计算口径 | 使用目的 |
|---|---|---|
| Visible Recall | $\sum_pTP_p/\sum_p(TP_p+FN_p)$；另报 pose-macro | 保留可见单元数量的诊断，补充视觉权重较低单元的覆盖情况。 |
| False Occlusion Rate | $\sum_pFN_p/\sum_p(TP_p+FN_p)=1-\operatorname{Recall}_{\mathrm{agg}}$ | 直接反映可见单元中被错误剔除的比例。 |
| Useful Cull Ratio | $\sum_pTN_p/\sum_pn_p$；另报 pose-macro | 表示全部候选中真正减少了多少无效处理，补充 CNOR 的机会归一化含义。 |
| Bad Cull Ratio | $\sum_pFN_p/\sum_pn_p$；另报 pose-macro | 将错误剔除量与候选工作负载联系；不能代替以可见单元为分母的误剔率。 |
| PR-AUC（Average Precision，AP） | 按分数排序后计算非插值 $\mathrm{AP}=\sum_k(R_k-R_{k-1})P_k$ | 在不选阈值的情况下评价可见单元的排序能力，与渐进式内容优先级直接相关。 |
| 正类比例与 AP lift | 与 AP 相同聚合口径的正类比例 $\pi$，以及 $\mathrm{AP}/\pi$ | 说明不同候选分布下的排序任务难度，避免只比较跨场景 AP 数值。 |
| 平均预测单元数 | $\operatorname{mean}_p\lvert\widehat{\mathcal G}_p\rvert$ | 给出客户端实际保留工作量，补充相对比率。 |

项目中的 PR-AUC 指非插值 Average Precision，不是 PR 曲线的梯形积分。pose-macro AP 在各 pose 内排序，只对包含正例的 pose 求均值并报告有效 pose 数；aggregate AP 将候选拼接后计算。零 GT pose 仍参与适用的候选与计数指标。Accuracy、Precision、Balanced Accuracy 等列入完整结果表，正文以安全性、CNOR 和 Useful Cull 为主。

## 6.2 主可见性结果

完整模型 FULL 在固定 logit 操作点 $\tau=0$ 下报告各场景、最差场景及场景等权汇总结果。主表同时包含 WR/LCB、Visible Recall、CNOR、Useful Cull 和 Bad Cull，分别对应视觉安全、数量覆盖、归一化剔除能力与实际工作量减少。目标场景校准的结果单独标注，校准数据与最终评价数据隔离。

图 4 展示同一观察区域下的参考、正确剔除、额外保留和可见遗漏；图像案例来自实际评价，不用示意图替代结果。表 3 承载定量主结果。

## 6.3 消融实验

| 正式对照 | 唯一研究因素 | 主要评价 |
|---|---|---|
| FULL / GEOMETRY_FIELD | 是否使用周围潜在遮挡关系；保留结构化场与同一训练目标 | 在相近安全性下比较 CNOR 和 Useful Cull，评价周围上下文的作用。 |
| FULL / GENERIC_RELATION_28 | 结构化解析生存场与同为 28 维运行表示的无结构关系潜变量 | 比较紧凑表示形式对安全性、剔除和排序的影响；不将该对照解释为单独隔离每一个数学先验。 |
| FULL / PBCE_OBJECTIVE | 完整表征不变，保守目标与逐 pose 平衡 BCE 对照 | 比较固定操作点的 WR/LCB 与剔除效率，并以 AP 辅助区分排序与决策边界变化。 |

消融范围保持上述三组。表 4 汇总重复实验；训练曲线、分数分布仅属于已有实验的补充分析。

## 6.4 跨场景迁移

采用 LOSO 和外部独立场景，冻结网络、检查点及评价协议后报告固定操作点结果。外部场景不参与架构、超参数或检查点选择；可见性标签用于最终评价。目标校准作为独立口径，不能以最终评价标签选阈值。表 5 区分固定阈值与目标校准，定性案例并入图 4。

## 6.5 运行与存储开销

表 6 记录实际导出资产、bytes/unit、共享权重、编译时间和客户端查询延迟。图 5 以候选规模为横轴报告中位数和尾部延迟，用于检验逐单元查询随工作负载增长的代价。WebGPU、WASM 采用同一 V5 资产版本；几何或代理基线同时计入前置表示及准备成本。

资产字节数反映获得可见性信号的网络启动成本，查询时间反映这一信号能否及时参与交互式内容选择。性能结果由 V5 实际部署测量，不以 V4 结果替代。

## 6.6 渐进式传输

相同候选单元、单位成本、缓存与带宽条件下比较原始顺序、几何启发式、已完成的代理／HZB 比较项、模型分数和 GT 参考顺序。设到达集合为 $\mathcal D(t)$，则：

$$
\operatorname{Coverage}(t)=
\frac{\sum_{u_i\in\mathcal D(t)\cap\mathcal G_p}w_{pi}}
{\sum_{u_i\in\mathcal G_p}w_{pi}}.
$$

Cost/Bytes@95/99/99.9 衡量恢复相同可见贡献所需的传输成本，而非只比较最终完整下载量；time-to-coverage 衡量内容到达速度；冗余量刻画达到该覆盖目标前用于不可见候选的开销。图 6 绘制覆盖曲线，表 7 汇总目标覆盖率对应的成本与时间。调度重放与完整浏览器端到端延迟分别报告。

## 实现依据

指标公式与边界条件核对于实现仓库 `4faca3ce83ecc269d83a94c215a353925aa9be7e`：

- [统一评价协议](https://github.com/luxingzhi27/web3d-pvs/blob/4faca3ce83ecc269d83a94c215a353925aa9be7e/docs/evaluation/evaluation_protocol.md)。
- [CNOR 实现](https://github.com/luxingzhi27/web3d-pvs/blob/4faca3ce83ecc269d83a94c215a353925aa9be7e/neural_instance_culling/model/common/culling_metrics.py)。

本文仅解释现有指标及其统计动机，不更改指标实现或实验划分。
