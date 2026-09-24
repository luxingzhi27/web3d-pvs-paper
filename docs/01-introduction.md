# 01 — Introduction

## Section goal

The Introduction must make the reader believe that **pre-geometry visibility is a distinct systems/graphics problem**, not just a different neural-network implementation.

## Recommended logic

### Paragraph 1 — Progressive Web3D makes prioritization unavoidable

Start from the system constraint:

- large scenes exceed acceptable startup transfer;
- browsers must progressively request resources;
- the first-order question is **which resource should arrive next?**

Mention current practical signals:

- view frustum;
- distance;
- hierarchy;
- geometric error / SSE;
- cache state.

Do not start from “occlusion culling is important”.

### Paragraph 2 — Existing pre-download signals miss occlusion

Use a simple example:

```text
camera → building A → building B
```

Both A and B can be:

- inside the frustum;
- close to the camera;
- high SSE / high projected importance;

while B is fully occluded by A.

Thus “relevance before download” lacks an occlusion term.

### Paragraph 3 — The circular dependency

State the key contradiction:

> Streaming would benefit from visibility before transfer, while conventional visibility algorithms derive visibility from scene geometry that may not yet be resident.

Introduce **pre-geometry visibility**.

This paragraph should explicitly distinguish:

- from-point runtime culling;
- from-region PVS;
- pre-geometry from-region visibility.

### Paragraph 4 — Why existing PVS does not directly remove the bootstrap problem

Acknowledge strong prior work:

- precomputed PVS;
- online PVS;
- Camera Offset Space;
- Trim Regions;
- Disocclusion Buffer;
- NeuralPVS.

Do not say these methods are “too slow” in general.

Instead say they assume that the visibility processor has access to a sufficiently detailed representation of the scene.

### Paragraph 5 — Key insight

The server/content pipeline *does* have geometry offline.

Therefore the useful question is not:

> Can the browser reconstruct the full scene for visibility?

but:

> Can geometry-dependent occlusion knowledge be compiled into a much smaller transferable asset?

Introduce:

```text
scene geometry
   ↓ offline shared compiler
compact visibility asset
   ↓ transfer
browser region query
   ↓
visibility score
```

### Paragraph 6 — Why a shared boundary matters

If every new scene needs visibility labels to calibrate a threshold, the “deploy on unseen geometry” story is weakened.

Therefore the method trains a common `logit = 0` boundary with conservative safety semantics across heterogeneous domains.

This is important enough to appear in the Introduction, not only in Training Details.

### Paragraph 7 — Web integration

Explain that the score is not a separate download model.

Instance scores are aggregated to resources and injected into an otherwise standard progressive scheduler.

The novelty is the availability of an occlusion-aware score before detailed content residency.

### Paragraph 8 — Contributions

Use 3–4 contributions, not 7 architecture bullets.

Recommended contributions:

1. pre-geometry from-region visibility formulation;
2. geometry-compiled compact occlusion field;
3. shared conservative cross-domain boundary;
4. Web3D integration/evaluation.

## Teaser / Figure 1

Figure 1 should explain the problem, not the neural architecture.

Suggested layout:

```text
Server                           Browser

detailed GLBs ───── deferred ───────┐
                                    │
compact visibility asset ───────────┤
                                    ↓
                              current view-cell
                                    ↓
                             visibility scores
                                    ↓
                       resource download priority
                                    ↓
                        useful geometry arrives first
```

Include one occluded resource that frustum/SSE would otherwise prioritize.

## What not to put in the Introduction

Avoid:

- exact 256-point input;
- exact 4×7 field;
- 12 anchors / top-8;
- SmoothMax equations;
- all metric names.

Those belong later.
