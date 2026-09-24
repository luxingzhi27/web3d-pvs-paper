# 11 — CGF / Graphics Writing Style Guide

## Lead with the graphics problem

Prefer:

> Progressive Web3D needs an occlusion signal before content residency.

Avoid:

> We propose a novel neural network with three modules.

## Explain design choices through constraints

For every component, answer:

- what graphics/system constraint requires it?
- what simpler control tests the hypothesis?

Examples:

- relation context exists because target-local shape does not encode surrounding occluders;
- the fixed directional field exists because the runtime asset must remain compact and analytically queryable;
- the shared boundary exists because target-scene calibration weakens label-free deployment.

## Separate fact, hypothesis, result

Fact:

> We use a fixed first-order directional basis.

Hypothesis:

> The structure may transfer more predictably than an unrestricted latent.

Result:

> Full outperforms Generic-28 under metric X.

Do not write a hypothesis as if already proven.

## Avoid novelty inflation

Do not claim:

- first use of visibility in streaming;
- first neural visibility;
- first online PVS;
- first client-side content selection.

Claim the actual intersection.

## Prefer hypothesis-driven ablations

Good:

> Full vs. Geometry Field tests whether surrounding occlusion context is necessary.

Weak:

> Removing the graph reduces metric X.

## Safety-first language

For conservative PVS:

- Weighted Recall / LCB first;
- culling efficiency second.

Do not headline Accuracy/F1.

## Be precise about “geometry”

Prefer:

- detailed streamed geometry;
- runtime scene representation;
- proxy geometry;
- compact metadata.

Avoid saying a method needs the “full detailed mesh” unless that is strictly true.

## Be precise about generalization

Distinguish:

- shared in-domain;
- LOSO;
- external blind holdout;
- target-calibrated diagnostics.

Do not call target-calibrated performance zero-shot.

## Keep V4 and V5 evidence separate

V4 can be described as:

- historical deployed system;
- legacy baseline;
- engineering reference.

V5 is the paper method.

Do not merge them into a single result row without an actual V5 deployment.

## Section transitions

Each section should motivate the next:

- Related Work → missing operating point.
- Problem Formulation → representation requirements.
- Representation → need for a stable deployment boundary.
- Boundary Learning → deployable score.
- Web Integration → experimental questions.
