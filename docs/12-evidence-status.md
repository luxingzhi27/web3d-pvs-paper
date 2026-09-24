# 12 — Evidence Status（证据状态）

## Source Snapshot

主实现仓库：

- `luxingzhi27/web3d-pvs`
- 当前论文规划基准：`4faca3ce83ecc269d83a94c215a353925aa9be7e`
- 日期：2026-09-24

本文档用于防止：

> “代码里已经计划/实现”被误写成“论文已经实验证明”。

---

# 零、已冻结的统一方法协议

论文方法层统一采用：

- renderable unit 作为唯一 visibility / culling 单位；
- 每个 physical center 展开 12 个离线 yaw/pitch directions；
- view-cell 为固定方向下的 horizontal disk；
- 每个 view-cell 用 32 个圆盘内 Color-ID camera positions 生成 sampled regional GT；
- 66° FOV 用于 GT、candidate 与 PVS contract；
- 当前真实显示使用 60° frustum；
- candidate 由后退视锥独立生成，禁止补 GT；
- split 按 physical center 分组；
- runtime 每个 view-cell 只执行一次 batched unit query；
- V5 内部 9 点只用于 analytic field statistics。

统一写作时避免把离线 GT sampling 与在线 query 混淆，不使用“32 个 runtime views”或“9 次 camera query”这类表述。

详细 source of truth：

- `docs/03a-unified-viewcell-protocol.md`

---

# 一、已经实现且可以作为 Method 事实描述的内容

## V5 Architecture

已经实现：

- 32D local geometry encoder；
- geometry-only 12-direction top-8 relation graph；
- relation attention compiler；
- fixed 12×7 → 4×7 directional projection；
- analytic monotone survival transform；
- 9-support region query；
- Full 52→32→1 visibility head；
- Geometry Field control；
- Generic Relation 28 control。

这些可以在 Method 中按代码事实描述。

---

## Shared-Boundary Objective

已经实现：

- fixed zero boundary；
- extra-retention objective；
- blended count/visual safety risk；
- 10 risk domains；
- SmoothMax-style robust aggregation；
- shared dual；
- PBCE objective control。

这些也可以作为正式方法描述。

---

## Evaluation Infrastructure

已经实现：

- train/calibration/validation/test 权限隔离；
- V5 columnar score bundle；
- fixed-zero evaluation；
- calibrated diagnostic evaluation；
- bootstrap LCB；
- shared / LOSO contracts；
- streaming score export infrastructure。

注意：

> infrastructure implemented ≠ formal result completed。

---

# 二、当前已经有的 Model Evidence

目前已经存在：

- Full 的长期 validation trajectory；
- Geometry Field 的长期 validation trajectory；
- Generic-28 的长期 validation trajectory；
- PBCE 的长期 validation trajectory；
- 多个随机重复，其中部分 seed3 control 在当前快照时尚未全部闭环。

当前可以作为内部论文故事依据的趋势：

- Full 在 fixed-zero 下具有强 safety；
- Geometry Field 可以很 conservative，但 culling efficiency 较低；
- Generic-28 提供了有意义的 structured-vs-unstructured 对照；
- PBCE 并不会自然获得所需的 shared zero boundary。

但是：

> 最终正文数字必须等 formal matrix 完整后再冻结。

---

# 三、最终论文最重要但尚需闭环的证据

## 1. 完整 Formal Random-Repeat Matrix

用途：

- main model table；
- mean/std；
- seed stability。

未完成前，不冻结 Full / ablation 最终数字。

---

## 2. External Blind Holdout

这是最强 generalization claim 的关键。

必须保证：

- blind scene 不参与 architecture design；
- 不参与 checkpoint selection；
- 不参与 hyperparameter tuning；
- 不参与 threshold tuning。

用途：

> 证明 frozen shared model 可以对未见 scene 仅凭 geometry compilation 获得可用 visibility。

---

## 3. V5 Browser Deployment

当前源仓库 README 明确：

> V5 尚未正式接入 browser runtime。

需要完成：

- V5 export format；
- WebGPU implementation；
- WASM implementation；
- PyTorch / Web parity。

否则最终论文无法完整证明：

> Web client runtime feasibility。

---

## 4. V5 Runtime Benchmark

需要：

- actual exported asset bytes；
- bytes / instance；
- candidate-count scaling；
- WebGPU p50 / p95；
- WASM p50 / p95；
- desktop / mobile。

用途：

> 回答“为什么不直接传 proxy geometry 跑 conventional PVS？”。

---

## 5. V5 Image / Color-ID Validation

当前正式 image-level evidence 主要来自 V4。

V5 如果最终要声称：

- visual correctness；
- image-safe filtering；

必须有自己的 frozen image evaluation。

---

## 6. V5 Streaming Re-Evaluation

现有 streaming framework 已经较完整，但最终 paper 必须重新用 V5 score 生成：

- pure neural ranking；
- cost-aware ranking；
- Bytes@95/99/99.9；
- threshold filtering；
- scheduler replay。

不能直接沿用 V4 system result 冒充 V5 end-to-end evidence。

---

## 7. IFCBench / HZB / Scheduler 尚未闭环的部分

凡源仓库当前标记：

- unavailable；
- pending；
- formal result not ready；

的项目，在正式产物生成前继续保持 unavailable。

不能为了表格完整人工补值。

---

# 四、论文写作的证据门

任何正文句子出现以下词：

- demonstrates；
- outperforms；
- generalizes；
- real-time；
- reduces transfer by；
- improves latency by；

都必须能指向一个已经完成、冻结、协议正确的 formal artifact。

否则只能写成：

- design goal；
- hypothesis；
- preliminary validation；
- planned evaluation。
