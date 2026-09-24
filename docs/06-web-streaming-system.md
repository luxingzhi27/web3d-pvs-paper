# 06 — Progressive Web3D Integration

## Section goal

Show why the representation solves a Web-system problem. The scheduler itself should not be oversold as a new scheduling algorithm.

## 6.1 Offline scene compilation

For a new scene:

```text
detailed geometry
   ├─ surface sampling → z_i
   └─ AABBs → proxy relation graph
                      ↓
               frozen shared compiler
                      ↓
                  field C_i
                      ↓
               runtime asset export
```

Central deployment property:

> No target-scene visibility labels or target-scene fine-tuning are required for geometry compilation.

## 6.2 Browser runtime

The browser receives compact per-unit metadata and shared query weights.

The point encoder and relation compiler are offline.

Runtime:

```text
view-cell
   ↓
candidate units
   ↓
nine-support analytic field query
   ↓
visibility logits
```

The paper must report actual exported bytes, not only a theoretical FP16 count.

## 6.3 Instance-to-resource aggregation

When multiple instances map to one GLB/resource, use the exact implementation rule. Current conceptual form:

\[
p_g=\max_{i:g(i)=g}p_i.
\]

Rationale:

If any instance of a shared resource is strongly relevant, fetching the resource can be useful.

## 6.4 Ranking and filtering are different tasks

### Threshold filtering

Uses the conservative decision boundary and decides which resources can be deferred.

Evaluate:

- retained GLBs;
- coverage ceiling;
- missed visible utility;
- retained bytes.

### Threshold-free ranking

Uses continuous scores over the same candidate resource set.

Evaluate:

- Bytes@95;
- Bytes@99;
- Bytes@99.9;
- waste-before-99;
- bandwidth-converted time.

Do not mix a threshold-filtered subset into the ranking curves.

## 6.5 Cost-aware priority

Current system also considers:

\[
p_g/B_g^\alpha.
\]

The paper should present both:

- pure visibility score;
- cost-aware visibility score.

This separates gains from occlusion prediction from gains due merely to preferring small resources.

Select \(\alpha\) on validation, never test.

## 6.6 Relation to modern Web selection

Position V5 as complementary to:

- frustum selection;
- hierarchical traversal;
- SSE / geometric error;
- cache policy;
- 3D Tiles-style request logic.

It adds:

> occlusion-aware relevance before detailed content residency.

It does not replace the rest of a production streaming stack.
