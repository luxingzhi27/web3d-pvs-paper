# web3d-pvs-paper

Paper-planning repository for the Web3D-PVS / GCOF-PVS project.

## Current paper identity

**Working problem:** pre-geometry from-region visibility for progressive Web3D.

**Central claim:** a Web client can obtain a conservative, occlusion-aware visibility signal *before* detailed streamed geometry is resident by querying a compact visibility asset compiled offline from geometry only.

## Current method identity (V5)

1. Scene-independent 32D local surface geometry encoder.
2. Geometry-only potential-occluder graph over 12 fixed directions, top-8 candidates per target/direction.
3. Single-layer relation compiler producing 12×7 anchor responses.
4. Fixed first-order directional projection to a structured 4×7 (28D) monotone survival field.
5. Nine-support analytic from-region query → center/max/mean/min survival statistics.
6. 32D geometry + 4D field statistics + 16D query geometry → 52→32→1 visibility logit.
7. Shared conservative zero-logit boundary trained with a domain-robust constrained objective.
8. Instance scores are aggregated to resource/GLB scores and used for progressive delivery.

## Source-of-truth implementation snapshot

Primary implementation repository:

- `luxingzhi27/web3d-pvs`
- planning snapshot: `4faca3ce83ecc269d83a94c215a353925aa9be7e`
- date: 2026-09-24

Important V5 sources:

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

## Documentation map

| File | Purpose |
|---|---|
| `docs/00-paper-positioning.md` | Thesis, novelty boundary, contribution hierarchy |
| `docs/01-introduction.md` | Introduction logic and paragraph-by-paragraph writing plan |
| `docs/02-related-work.md` | Related Work structure, literature roles, comparison logic |
| `docs/03-problem-formulation.md` | Formal task definition and notation |
| `docs/04-gcof-method.md` | Geometry-compiled occlusion field method |
| `docs/05-shared-boundary-learning.md` | Robust zero-boundary training objective |
| `docs/06-web-streaming-system.md` | Web runtime and scheduling integration |
| `docs/07-evaluation-plan.md` | Research questions, baselines, metrics, missing evidence |
| `docs/08-discussion-limitations.md` | Limitations and reviewer-facing discussion |
| `docs/09-figures-tables.md` | Planned figures/tables and the claim each supports |
| `docs/10-title-abstract-conclusion.md` | Title candidates and abstract/conclusion logic |
| `docs/11-cgf-writing-style.md` | Writing rules for a CGF/graphics paper |
| `docs/12-evidence-status.md` | Supported claims vs. experiments still needed |
| `references/core-literature.md` | Core literature and why each paper matters |
| `paper/README.md` | Manuscript drafting workflow |

## Working rule

This repository is a **paper reasoning source of truth**, not a dump of every experiment note.

Each section document should answer four questions:

1. What does this section need to prove?
2. What is the minimum technical content needed to prove it?
3. Which existing work must it compare against?
4. Which experimental evidence is required before the prose can be frozen?

The manuscript should only be frozen after the corresponding evidence is stable.
