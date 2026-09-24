# 07 — Evaluation Plan

## Goal

Experiments should answer research questions rather than simply report every metric available in the repository.

## RQ1 — Can one fixed boundary remain conservative across scenes?

Primary decision protocol:

\[
\tau=0.
\]

Report:

- per-scene weighted recall;
- one-sided 95% LCB;
- scene-equal mean;
- worst scene;
- CNOR;
- Useful Cull;
- Bad Cull.

Compare:

- Full;
- Geometry Field;
- Generic Relation 28;
- PBCE.

Calibrated thresholds are diagnostic and should not replace the fixed-zero main result.

Recommended figure:

- visible/invisible score distributions for each scene with a vertical `z=0` line.

## RQ2 — Is surrounding occlusion context necessary?

Compare:

- Full;
- Geometry Field.

Keep the structured field, query machinery and robust objective aligned.

Question:

> Can local target geometry alone provide useful conservative culling?

Interpretation to test:

- local geometry may support a safe prior;
- surrounding relation context should improve useful culling / CNOR.

## RQ3 — Does the structured field matter?

Compare:

- Full;
- Generic Relation 28.

Control:

- same relation evidence;
- same 28-value runtime context budget;
- same robust objective.

Question:

> Is the gain due only to relation information, or does the structured analytic field provide a better deployment representation?

## RQ4 — Does shared conservative boundary learning matter?

Compare:

- Full with robust-boundary objective;
- PBCE objective.

Representation remains identical.

Report:

- fixed-zero WR / LCB;
- calibrated diagnostics;
- PR-AUC;
- score distributions;
- robust-risk and shared-dual training dynamics where stable.

Question:

> Does conventional balanced classification learn a transferable operational boundary?

## RQ5 — Does the model generalize without target-scene labels?

### External blind holdout

Preferred headline generalization protocol.

Freeze:

- architecture;
- checkpoint;
- hyperparameters;
- `z=0`.

Then compile the held-out scene from geometry only and evaluate afterwards.

Do not use the blind scene for variant/checkpoint selection.

### LOSO

Use as systematic secondary evidence across registered real scenes.

Keep representation transfer distinct from target-threshold calibration.

## RQ6 — What is the asset and runtime cost?

Must be measured for V5 itself.

Report:

- total runtime-asset bytes;
- bytes/unit;
- shared-model bytes;
- compilation time;
- WebGPU latency;
- WASM latency;
- candidate-count scaling.

Where fair, compare with geometry-shell HZB/proxy alternatives.

Do not use V4 runtime results as if they were V5.

## RQ7 — Does V5 improve progressive delivery?

Use V5 score sidecars.

Threshold-free ranking baselines:

- original/baseline order;
- projected area per byte;
- AABB model;
- HZB visible-first;
- V5 neural score;
- V5 cost-aware neural score;
- GT utility/byte oracle.

Metrics:

- Bytes@95;
- Bytes@99;
- Bytes@99.9;
- waste-before-99;
- fixed-bandwidth time equivalents.

All methods must rank the same candidate resource set.

## RQ8 — Does the signal help the real scheduler?

Use:

- cold cache;
- same scheduler state machine;
- same bandwidth;
- same pose set;
- only the priority source changes.

Report:

- time-to-coverage;
- bytes;
- waste;
- p50/p95.

Be explicit about measurement scope:

> Node download/scheduler replay is not the same as end-to-end browser first-frame rendering.

## Metrics hierarchy

### Safety first

- weighted recall;
- 95% one-sided LCB;
- Bad Cull;
- image-visible misses if available.

### Efficiency second

- CNOR;
- Useful Cull;
- predicted count.

### Ranking

- pose PR-AUC;
- prevalence;
- lift.

### System utility

- startup bytes;
- runtime;
- Bytes@x;
- waste;
- scheduler replay.

Avoid letting ordinary Accuracy/F1 dominate the paper story.

## Current status at source snapshot `4faca3c`

Current V5 validation supports a promising story:

- Full maintains strong fixed-zero safety while culling substantially more than Geometry Field;
- Generic-28 is a meaningful structured-vs-unstructured representation control;
- PBCE does not naturally inherit a safe common zero boundary.

Do not freeze final numerical tables until the registered formal random-repeat matrix and frozen test/generalization evidence are complete.
