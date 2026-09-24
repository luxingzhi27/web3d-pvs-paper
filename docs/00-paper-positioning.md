# 00 — Paper Positioning

## One-sentence research question

> How can a Web client estimate conservative from-region visibility before the detailed scene geometry required by conventional visibility algorithms has been downloaded?

A more precise version for the paper:

> Can a Web client predict occlusion-aware from-region visibility using only compact pre-transmitted scene descriptors, before the detailed geometry required by conventional visibility algorithms is resident?

## Core problem

Large Web3D scenes cannot be transferred in full before interaction begins. Existing streaming systems therefore need to decide **what to download next**.

Frustum, distance, hierarchy and screen-space-error metadata can be evaluated before tile content arrives, but they do not provide occlusion-aware relevance.

This creates a circular dependency:

```text
visibility is useful for deciding what geometry to download
                    ↑
                    │
conventional visibility usually requires scene geometry
```

The paper should name this **pre-geometry visibility**.

## What the paper is NOT

Do not position the paper as:

- “a neural occlusion-culling network”;
- “a faster NeuralPVS”;
- “a new download scheduler”;
- “a PointNet/GNN architecture”;
- “a WebGPU optimization paper”.

Those are components or consequences.

## Strongest paper identity

**A compact geometry-compiled visibility representation with a scene-independent conservative decision boundary for pre-geometry Web streaming.**

The two core technical pieces are:

1. **GCOF representation** — geometry-only scene context is compiled offline into a compact structured field that can be queried without the detailed target geometry.
2. **Shared conservative boundary learning** — the same zero-logit boundary is trained to have a safety-oriented meaning across heterogeneous scenes/domains.

The Web streaming pipeline demonstrates why these two properties matter.

## Novelty boundary

Do not claim:

- visibility-guided streaming is new;
- neural visibility is new;
- from-region PVS is new;
- metadata-driven pre-content selection is new.

Existing work already covers each of these individually.

The claim is the intersection:

```text
learned visibility
+ from-region query
+ geometry-only scene compilation
+ client-side query before detailed geometry residency
+ shared conservative boundary
+ progressive Web delivery
```

## Contribution hierarchy

### Contribution A — Problem formulation

Pre-geometry from-region visibility for progressive Web3D.

### Contribution B — Representation

Geometry-only local descriptors + potential-occluder relations are compiled into a compact 4×7 structured directional survival field.

### Contribution C — Learning objective

A fixed zero boundary is made conservative across heterogeneous domains using an efficiency objective under a domain-robust safety constraint.

### Contribution D — System evidence

The signal is integrated with resource-level streaming and evaluated in terms of safety, culling efficiency, representation cost, runtime cost and progressive-delivery utility.

## Key reviewer question the whole paper must answer

> Why should the client download this visibility asset instead of simply downloading coarse geometry and running a conventional visibility method?

The paper must therefore compare:

- startup bytes;
- runtime cost;
- candidate-count scaling;
- safety;
- useful culling;
- streaming benefit;

against geometry-resident/proxy alternatives such as HZB where possible.
