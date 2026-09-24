# 10 — Title, Abstract and Conclusion

## Working title candidates

### Preferred problem-first title

**Pre-Geometry Visibility for Progressive Web3D via Geometry-Compiled Occlusion Fields**

Strength:

- immediately communicates the problem;
- does not overcommit to “generalization” in the title.

### Method-name title

**GCOF-PVS: Geometry-Compiled Occlusion Fields for Pre-Geometry Visibility in Progressive Web3D**

Strength:

- establishes a reusable method name.

### More general graphics title

**Geometry-Compiled From-Region Visibility Before Content Residency**

Potential weakness:

- Web3D motivation becomes less explicit.

## Abstract logic

Use six moves.

### 1. Context

Large Web3D scenes require progressive delivery.

### 2. Problem

Existing visibility algorithms assume access to a scene representation, while a streaming client needs visibility to decide which geometry to request before that geometry is resident.

### 3. Formulation

Introduce pre-geometry from-region visibility.

### 4. Method

Shared geometry encoder + geometry-only potential-occluder relations → compact structured directional field → lightweight region query.

### 5. Training

Shared conservative zero boundary trained across heterogeneous domains.

### 6. Evidence

Report only completed evidence:

- fixed-boundary safety;
- useful culling;
- unseen-scene transfer;
- asset/runtime cost;
- streaming benefit.

Do not list every architecture dimension in the abstract.

## Conclusion logic

Return to the systems paradox:

> Conventional visibility becomes available after scene geometry is present; progressive delivery benefits from visibility before that point.

Summarize the conceptual answer:

```text
geometry-only scene context
→ compact compiled visibility asset
→ conservative shared-boundary query
→ progressive delivery
```

Do not end by repeating network layer dimensions.
