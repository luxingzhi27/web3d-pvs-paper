# 09 — Planned Figures and Tables

## Figure 1 — Problem teaser

Claim:

> Visibility is needed before detailed geometry residency.

Show:

- server-side detailed resources;
- compact visibility asset;
- browser view-cell;
- an occluded but frustum-visible resource;
- visibility-aware download priority.

Do not show the full neural architecture here.

## Figure 2 — Operating-point comparison

Three conceptual pipelines:

```text
Conventional online PVS
resident scene representation → visibility → render

NeuralPVS
runtime geometry representation → neural PVS → render

GCOF-PVS
compact compiled asset → visibility → download → geometry → render
```

Purpose:

make the content-residency difference immediately visible.

## Figure 3 — GCOF architecture

Show:

- local surface encoder;
- 12-direction top-8 relation graph;
- attention relation compiler;
- 12×7 anchor responses;
- fixed projection;
- 4×7 field;
- OFFLINE / WEB RUNTIME boundary;
- nine-support query;
- tiny head.

## Figure 4 — Structured field intuition

For one target:

- several directions;
- survival vs. normalized distance;
- monotonicity;
- `S(0)=1`.

Optional Generic-28 comparison.

## Figure 5 — Shared zero boundary

Per-scene visible/invisible score distributions with vertical `z=0`.

Compare:

- Full;
- PBCE.

This can become one of the strongest paper figures.

## Figure 6 — Safety-efficiency frontier

Possible axes:

- x = Useful Cull / CNOR;
- y = weighted recall or miss risk.

Show main variants and baselines.

## Figure 7 — Asset/runtime scaling

Two panels:

1. visibility-asset bytes vs. unit count;
2. query latency vs. candidate count.

Include WebGPU and WASM once V5 runtime exists.

## Figure 8 — Streaming utility

x-axis:

- downloaded MiB.

y-axis:

- visible-weight coverage.

Methods:

- V5 score;
- V5 score/byte;
- AABB;
- HZB;
- projected-area/byte;
- oracle.

---

## Table 1 — Related-work operating point

Potential columns:

- method;
- from-region;
- runtime scene representation required;
- learned;
- pre-content client query;
- streaming use.

Keep the wording neutral and factual.

## Table 2 — Scene summary

- scene;
- domain type;
- units;
- GLBs/resources;
- view-cell type;
- split counts;
- role in shared / LOSO / blind evaluation.

## Table 3 — Fixed-zero main result

Methods:

- Full;
- Geometry Field;
- Generic-28;
- PBCE.

Report:

- worst-scene WR/LCB;
- scene-equal WR;
- CNOR;
- Useful Cull;
- Bad Cull;
- PR-AUC lift.

## Table 4 — Generalization

Separate:

- shared-known scenes;
- LOSO;
- external blind holdout.

Explicitly mark whether target labels/calibration are used.

## Table 5 — Runtime asset

- method;
- startup bytes;
- bytes/unit;
- WebGPU p50/p95;
- WASM p50/p95;
- candidate count.

## Table 6 — Streaming

- Bytes@95;
- Bytes@99;
- waste;
- bandwidth-converted time.

## Table 7 — Scheduler replay

- p50/p95;
- downloaded bytes;
- waste;
- startup-asset overhead.

## Rule

Every figure/table must answer a reviewer question. If it only documents implementation detail without supporting a claim, move it to supplementary material.
