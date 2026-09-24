# 12 — Evidence Status

## Source snapshot

Primary implementation repository:

- `luxingzhi27/web3d-pvs`
- planning snapshot: `4faca3ce83ecc269d83a94c215a353925aa9be7e`
- date: 2026-09-24

## Strong / already implemented

### V5 architecture

Implemented:

- 32D local geometry encoder;
- geometry-only 12-direction top-8 relation graph;
- relation attention compiler;
- fixed 12×7 → 4×7 directional projection;
- analytic monotone survival transform;
- nine-support region query;
- Full 52→32→1 visibility head;
- Geometry Field control;
- Generic Relation 28 control.

### Shared-boundary objective

Implemented:

- fixed zero boundary;
- extra-retention objective;
- blended count/visual safety risk;
- ten risk domains;
- SmoothMax-style robust aggregation;
- shared dual;
- PBCE objective control.

### Evaluation infrastructure

Implemented:

- strict train/calibration/validation/test permissions;
- V5 score bundles;
- fixed-zero evaluation;
- calibrated diagnostic evaluation;
- bootstrap LCB;
- shared / LOSO contracts;
- streaming-score export infrastructure.

## Current model evidence

Available:

- long validation trajectories for Full, Geometry Field, Generic-28 and PBCE;
- multiple random repeats, though the formal matrix was not fully closed at this snapshot.

Current qualitative story:

- Full maintains strong fixed-zero safety;
- Geometry Field is safe but less effective at culling;
- Generic-28 provides a strong structured-vs-unstructured representation control;
- PBCE does not naturally create the desired common zero boundary.

Do not freeze final manuscript numbers until the formal result matrix is complete.

## Evidence still needed for the strongest paper

### Complete formal random-repeat matrix

Needed before the main model table is frozen.

### External blind holdout

Needed for the strongest label-free unseen-scene claim.

### V5 browser deployment

Current source README states V5 is not yet integrated into the browser runtime.

Needed:

- V5 export format;
- V5 WebGPU path;
- V5 WASM path;
- PyTorch/Web parity checks.

### V5 runtime benchmark

Needed:

- actual asset bytes;
- bytes/unit;
- candidate-count scaling;
- desktop/mobile latency.

### V5 Color-ID / image validation

Current formal image evidence is V4-era. V5 needs its own image-level validation if the paper makes image correctness claims.

### V5 streaming re-evaluation

Final paper should regenerate with V5 scores:

- threshold-free ranking;
- cost-aware ranking;
- Bytes@95/99/99.9;
- threshold filtering;
- scheduler replay.

### HZB / IFCBench closure

Any result currently marked unavailable/pending in source documents should remain unavailable until formally produced.

## Writing rule

Any manuscript sentence using words such as:

- “demonstrates”;
- “outperforms”;
- “generalizes”;
- “real-time”;
- “reduces transfer by”;

must map to completed evidence listed here.
