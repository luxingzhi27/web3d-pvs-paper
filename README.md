# web3d-pvs-paper

这是 Web3D-PVS / GCOF-PVS 项目的**论文规划与论证仓库**。

当前阶段以**中文**为主，用于逐步收敛论文定位、相关工作、方法逻辑、实验问题与证据边界；等各部分稳定后，再在 `paper/` 下单独建立英文初稿。

## 当前论文定位

**研究问题：** 面向渐进式 Web3D 的“几何下载前（pre-geometry）from-region 可见性预测”。

**核心主张：** 浏览器在详细场景几何尚未到达之前，可以先接收一个由几何离线编译得到的紧凑可见性资产，并在客户端对当前观察区域执行轻量查询，从而获得保守、遮挡感知的可见性信号，用于内容过滤与下载排序。

## 当前 V5 方法主线

1. 场景无关的 32D 局部表面几何编码器；
2. 12 个固定方向 × 每方向 top-8 potential occluder 的纯几何关系图；
3. 单层 relation compiler，生成 12×7 anchor responses；
4. 固定一阶方向基投影为结构化 4×7（28D）单调 survival field；
5. 对水平圆盘 view-cell 的中心与 8 个圆周 support 做解析查询，得到 center/max/mean/min 四个 survival statistics；
6. 32D geometry + 4D field statistics + 16D query geometry → 52→32→1 visibility logit；
7. 用 domain-robust constrained objective 学习跨场景共享的保守零阈值决策边界；
8. 以 renderable unit 为统一预测与剔除单位，将 unit-level visibility signal 用于渐进式 Web 内容选择与排序。

## 源代码基准快照

主实现仓库：

- `luxingzhi27/web3d-pvs`
- 当前论文规划基准 commit：`4faca3ce83ecc269d83a94c215a353925aa9be7e`
- 日期：2026-09-24

V5 主要实现位置：

- `neural_instance_culling/model/v5/geometry_encoder.py`
- `neural_instance_culling/model/v5/core.py`
- `neural_instance_culling/model/v5/survival.py`
- `neural_instance_culling/model/v5/losses.py`
- `neural_instance_culling/model/v5/train.py`
- `neural_instance_culling/model/v5/runner.py`
- `neural_instance_culling/dataset/v5/proxy_relation_graph.py`
- `neural_instance_culling/benchmark/v5/*`
- `docs/experiments/pvs_v5_global_robust_boundary_2026-09-21.md`
- `docs/evaluation/pvs_v5_metrics_ledger.md`

## 文档索引

| 文件 | 作用 |
|---|---|
| `docs/00-paper-positioning.md` | 论文主问题、novelty 边界、贡献层级 |
| `docs/01-introduction.md` | Introduction 的段落逻辑与写法 |
| `docs/02-related-work.md` | Related Work 结构、文献角色、对比逻辑 |
| `docs/03-problem-formulation.md` | 正式问题定义与符号 |
| `docs/03a-unified-viewcell-protocol.md` | 统一水平圆盘 view-cell、GT、candidate 与在线查询协议 |
| `docs/04-gcof-method.md` | Geometry-Compiled Occlusion Field 方法 |
| `docs/05-shared-boundary-learning.md` | 共享保守零边界训练目标 |
| `docs/06-web-streaming-system.md` | Web runtime 与调度集成 |
| `docs/07-evaluation-plan.md` | 研究问题、baseline、指标与待补证据 |
| `docs/08-discussion-limitations.md` | 局限性与 reviewer-facing discussion |
| `docs/09-figures-tables.md` | 计划图表与每张图要证明的结论 |
| `docs/10-title-abstract-conclusion.md` | 标题、摘要、结论的写作逻辑 |
| `docs/11-cgf-writing-style.md` | 面向 CGF/图形学论文的写作规则 |
| `docs/12-evidence-status.md` | 已有证据与尚未完成的实验边界 |
| `references/core-literature.md` | 核心文献与每篇论文的作用 |
| `paper/README.md` | 英文初稿的后续写作流程 |

## 当前工作原则

这个仓库当前不是“最终论文正文”，而是**论文论证的 source of truth**。

每个章节文档都应回答四个问题：

1. 这一节需要证明什么？
2. 为证明这一点，最少需要写哪些技术内容？
3. 必须与哪些已有工作比较？
4. 哪些实验/证据完成后，这一节才能冻结？

等这些问题稳定以后，再开始英文初稿。
