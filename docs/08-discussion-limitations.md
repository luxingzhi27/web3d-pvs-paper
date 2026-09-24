# 08 — Discussion and Limitations

## Candidate-count dependence

V5 runtime query work scales with candidate units. This is a real trade-off relative to grid/froxel methods whose compute is tied more strongly to grid resolution.

The paper should show empirical scaling rather than hide it.

## Startup-asset scaling

Per-unit metadata means total visibility-asset size grows with scene unit count.

Trade-off:

- smaller streaming units → finer visibility/scheduling;
- more units → larger metadata + more runtime queries.

## Dynamic geometry

The compiled field reflects static geometry.

Geometry changes can invalidate:

- local descriptors;
- AABB relations;
- compiled occlusion fields.

Possible future directions:

- local recompilation;
- hybrid dynamic-occluder layer;
- runtime HZB residual.

Do not imply full dynamic-scene support unless implemented.

## Thin / porous / non-box-like occluders

The proxy relation graph is intentionally compact and AABB based. It may be less informative for:

- thin structures;
- porous geometry;
- foliage;
- highly non-box-like occluders.

This is a representation limitation, not merely an implementation bug.

## Low-order directional field

The fixed first-order basis is intentionally low capacity.

Benefits:

- compactness;
- smoothness;
- fixed runtime shape;
- analytic query.

Cost:

- limited angular frequency.

Generic-28 and possible higher-order controls help contextualize this design.

## Conservative safety-efficiency frontier

No method simultaneously maximizes:

- zero misses;
- aggressive culling.

Frame results as a safety-efficiency frontier.

## Generalization claim discipline

LOSO plus an external blind holdout can support meaningful cross-scene transfer under the tested distributions.

They do not justify claims of arbitrary open-world generalization.

## Survival-field semantics

The analytic field is an intermediate structured statistic.

Do not equate it with exact physical object-visibility probability.

## V4 / V5 system evidence

If V5 browser deployment is incomplete:

- keep V4 system results clearly labeled historical/legacy;
- do not merge V4 runtime and V5 model results into one “ours” row;
- close the V5 deployment gap before making end-to-end claims.
