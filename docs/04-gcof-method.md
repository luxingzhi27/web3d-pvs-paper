# 04 — Geometry-Compiled Occlusion Fields

## Section goal

Explain why V5 is not merely an instance classifier. The core chain is:

```text
local target geometry
+ geometry-only occlusion context
→ compact structured directional field
→ analytic region query
→ tiny visibility head
```

## 4.1 Local surface geometry encoder

Current implementation per renderable unit:

- 256 fixed surface samples;
- per-point channels: normalized local xyz + normal = 6D;
- 3 unit-level size ratios.

Network:

```text
point MLP: 6 → 32 → 64 → 64, SiLU
pool: max(64) || mean(64) || size ratios(3)
unit MLP: 131 → 64 → 32, SiLU then Tanh
```

Output:

\[
z_i\in\mathbb R^{32}.
\]

Emphasize:

- shared across scenes;
- no scene ID;
- no unit embedding;
- no per-instance learned residual;
- no scene-normalized world position;
- can compile a new scene without fitting target-specific parameters.

The PointNet-like architecture itself is not the novelty.

## 4.2 Geometry-only potential occlusion relations

Current graph:

- 12 fixed icosahedral directions;
- top-8 potential occluders per target/direction;
- AABB orthographic overlap and depth ordering;
- no visibility labels.

The relation artifact must declare `usesVisibilityLabels=false`.

Eight-dimensional edge feature:

1–3. relative direction;  
4. \(\log(1+d_{ij}/r_i)\);  
5. \(\log(r_j/r_i)\);  
6. target projected-overlap ratio;  
7. source projected-overlap ratio;  
8. \(\log(1+\mathrm{gap}/r_i)\).

The relation graph is deterministic geometry preprocessing, not learned scene memory.

## 4.3 Single-layer relation field compiler

For each edge:

\[
[z_i,z_j,e_{ij}]\in\mathbb R^{72}
\]

through:

\[
72\rightarrow64\rightarrow32.
\]

Within each target-anchor group:

\[
h_{ik}=\sum_j\alpha_{ijk}m_{ijk}.
\]

The compiler also keeps:

- \(\log(1+n_{ik})\);
- overlap sum.

Rationale:

Softmax attention normalizes neighborhood mass, so count and overlap statistics restore information about how much occluding evidence exists.

## 4.4 Anchor responses and fixed directional projection

Base branch:

\[
[z_i,a_k]:35\rightarrow32\rightarrow7.
\]

Relation delta:

\[
[h_{ik},\log(1+n_{ik}),o_{ik},a_k]
:37\rightarrow32\rightarrow7.
\]

Response:

\[
q_{ik}=q^{base}_{ik}+\mathbf1[n_{ik}>0]\Delta q_{ik}.
\]

Twelve anchor responses:

\[
Q_i\in\mathbb R^{12\times7}.
\]

They are projected with the fixed first-order basis:

\[
[1,d_x,d_y,d_z]
\]

using a Moore–Penrose pseudoinverse:

\[
Q_i\rightarrow C_i\in\mathbb R^{4\times7}.
\]

The projection is fixed, not learned. Runtime field size is 28 values.

Key hypothesis:

> A structured low-order directional field provides a better deployment representation than an equally compact unrestricted relation latent.

This is tested by `GENERIC_RELATION_28`.

## 4.5 Analytic monotone survival field

The 7 directional parameters represent:

- no-hit mass;
- two mixture logits;
- two positive locations;
- two positive scales.

Normalized distance:

\[
t=\log(1+d/r_i).
\]

The analytic transform is constructed so:

\[
S(0)=1
\]

and survival is non-increasing with distance along a fixed direction.

Writing rule:

> Do not call \(S\) the exact physical visibility probability. It is a structured intermediate occlusion/survival statistic.

## 4.6 Nine-support from-region query

Disk view-cell:

- center + 8 ring points.

Oriented-box view-cell:

- center + 8 corners.

For each support evaluate \(S_{ik}\), then compress to:

\[
[S_{center},S_{max},S_{mean},S_{min}].
\]

Important distinction:

> one compiled field + nine analytic evaluations + one small MLP, rather than nine complete neural visibility inferences.

## 4.7 Query geometry and visibility head

Current query geometry is 16D and contains:

- target→region world direction;
- region→target direction in camera right/up/forward;
- normalized distance/radius terms;
- normalized region extents;
- FOV;
- region type;
- near/far normalized terms.

Full head input:

\[
32+4+16=52.
\]

Head:

\[
52\rightarrow32\rightarrow1.
\]

This should be the endpoint of the inference method section.

## Hypothesis-driven representation controls

### Geometry Field

Removes surrounding relation context but retains the structured field and same query.

Question:

> Is target-local geometry alone sufficient for useful conservative visibility?

### Generic Relation 28

Keeps relation evidence and the same 28-value runtime budget, but removes the analytic survival-field structure.

Question:

> Does structure help beyond an unconstrained compact relation latent?
