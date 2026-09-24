# 03 — Problem Formulation

## Section goal

Formalize the task so the method is clearly different from:

- ordinary from-point occlusion culling;
- discrete precomputed PVS;
- runtime geometry-to-PVS inference.

## Entities

Renderable units:

\[
\mathcal U = \{u_i\}_{i=1}^N.
\]

Streaming resources / GLBs:

\[
\mathcal R = \{r_j\}_{j=1}^M.
\]

Mapping:

\[
g(u_i) = r_j.
\]

A resource can contain multiple units/instances.

## View region

Define a camera region / view-cell:

\[
\mathcal B.
\]

The exact region can be a disk or an oriented box under the current dataset contract.

## Ground-truth from-region visibility

A unit is potentially visible if at least one view/sample in the region can observe it:

\[
y_i(\mathcal B)=
\mathbf 1\left[
\exists c\in\mathcal B:
u_i\text{ contributes to the rendered image}
\right].
\]

The paper must use the exact dataset semantics, not an informal “object visible” definition.

## Pre-geometry constraint

At decision time, the client does not own the detailed content \(G_i\).

It only owns compact metadata / compiled asset \(a_i\).

The task is:

\[
f_\theta(a_i,\mathcal B)\rightarrow z_i,
\]

where \(z_i\) is a visibility logit or score.

The design target is:

\[
|a_i|\ll|G_i|.
\]

## Conservative asymmetry

False negative:

- visible geometry delayed or omitted;
- holes / missing content / pop-in / delayed complete frame.

False positive:

- additional transfer or rendering work.

Therefore:

\[
C_{\mathrm{FN}}\gg C_{\mathrm{FP}}.
\]

This asymmetry justifies constrained training and safety-first evaluation.

## Runtime decision

For fixed-boundary filtering:

\[
\hat y_i=\mathbf 1[z_i\ge0].
\]

For resource scheduling:

\[
p_g = \max_{i:g(i)=g} s_i
\]

or the exact score transform used by the implementation.

The filtering and ranking tasks must remain conceptually separate in the paper.
