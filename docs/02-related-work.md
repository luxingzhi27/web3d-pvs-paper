# 02 — Related Work

## Goal

The Related Work section should explain **when and where visibility becomes available** in prior systems, then locate our operating point. It should not be organized by implementation ingredients such as PointNet, GNN, attention, SH, or survival analysis.

Recommended structure:

1. Visibility Culling and From-Region PVS
2. Learned Visibility Representations
3. Visibility-Aware 3D / Web Streaming

---

## 2.1 Visibility Culling and From-Region PVS

### Runtime from-point culling

Keep HZB / hardware occlusion-query work brief. Their role is to establish the standard geometry-resident case:

> Conventional runtime occlusion culling rejects hidden objects efficiently once a suitable scene representation is resident at the renderer.

This motivates geometry/proxy-resident HZB baselines, but it is not the main research neighborhood.

### Classical precomputed PVS

Core references:

- Teller & Séquin, *Visibility Preprocessing for Interactive Walkthroughs*, SIGGRAPH 1991.
- Wonka et al., *Visibility Preprocessing with Occluder Fusion for Urban Walkthroughs*, 2000.
- Nirenstein & Blake, *Hardware Accelerated Visibility Preprocessing using Adaptive Sampling*, EGSR 2004.
- Bittner et al., *Adaptive Global Visibility Sampling*, SIGGRAPH 2009.

What this literature establishes:

- from-region PVS is a mature problem;
- expensive geometry reasoning can be moved offline;
- runtime visibility can therefore be cheap.

Classical pipeline:

```text
complete scene geometry
    ↓ offline
view-cell-specific PVS database
    ↓ runtime lookup
```

Important novelty discipline:

> Offline preprocessing itself is not our contribution.

Our distinction is that we do not store a discrete PVS for each view cell. A shared geometry-only compiler produces a compact per-unit representation that remains queryable for regions at runtime.

### Networked PVS and selective transmission

Important reference:

- Koltun, Chrysanthou, Cohen-Or, *Hardware-Accelerated From-Region Visibility Using a Dual Ray Space* (2001).

Why it matters:

This work already uses PVS for remote walkthrough / selective network transmission. Therefore we must not claim that “PVS-guided streaming” is new.

Correct comparison:

```text
server-side networked PVS:
full geometry at server
    ↓
server computes PVS
    ↓
client receives selected content

ours:
geometry compiled offline
    ↓
small visibility asset at client
    ↓
client queries visibility
    ↓
client prioritizes detailed content
```

The novelty is **where visibility is represented and queried**, not the general idea of using PVS for transmission.

### Online from-region PVS

Core modern references:

- Camera Offset Space (SIGGRAPH Asia 2019 / TOG).
- Guided Visibility Sampling++ (I3D 2021).
- Trim Regions (SIGGRAPH 2023 / TOG).
- Disocclusion Buffer (SIGGRAPH Asia 2025).

These papers show that online PVS is no longer synonymous with expensive offline preprocessing.

**Camera Offset Space** is particularly relevant because it targets streaming rendering.  
**Guided Visibility Sampling++** represents modern ray/sampling-based aggressive PVS.  
**Trim Regions** explicitly discusses dynamic applications including variable-bandwidth streaming.  
**Disocclusion Buffer** represents a recent highly parallel conventional PVS direction.

The correct synthesis is:

> Precomputed, sampling-based, image-space, and disocclusion-based methods have progressively reduced the cost of from-region visibility. Their common operating assumption is that the component performing visibility has access to a sufficiently detailed scene representation.

Do **not** say that all of them require the “full detailed mesh”; say they require a sufficiently informative runtime scene representation.

---

## 2.2 Learned Visibility Representations

### NeuralPVS — primary comparison

NeuralPVS is the most important learned baseline/conceptual comparison.

Its pipeline is approximately:

```text
scene geometry
    ↓ froxelization
runtime froxel geometry representation
    ↓ sparse neural network
from-region PVS
```

Acknowledge its strengths fairly:

- learned from-region PVS;
- strong runtime performance;
- synthetic data for cross-scene generalization;
- runtime cost is driven largely by froxel resolution rather than raw scene complexity.

Do not frame our method as “NeuralPVS but smaller”.

The key difference is where geometry enters:

```text
NeuralPVS:
runtime geometry representation → neural PVS

GCOF-PVS:
offline geometry → compact visibility representation
compact representation + region → runtime visibility
```

Recommended wording:

> NeuralPVS learns a fast mapping from a runtime geometric scene representation to PVS. GCOF-PVS moves scene-context reasoning to an offline geometry-compilation stage and transmits only a compact per-unit visibility asset for later client queries.

### Neural Visibility of Point Sets

Role:

- establishes that per-element viewpoint-dependent visibility can be learned;
- prevents any claim that neural visibility classification itself is novel.

Keep this concise.

### NVGS (CVPR 2026)

Very relevant conceptually.

Pattern:

```text
asset visibility samples
    ↓ distillation
compact neural visibility
    ↓ runtime query
avoid rasterizing occluded primitives
```

Similarity:

- lightweight learned visibility queried before expensive graphics work.

Difference:

- NVGS distills visibility for an existing 3DGS asset and uses it for render-time culling;
- V5 aims to compile a new scene from **geometry only**, with no target-scene visibility labels or fine-tuning, then query visibility before detailed content residency.

Useful contrast:

> visibility-distilled asset vs. geometry-compiled visibility asset.

### Neural visibility fields / NeRV-type work

Use only as a short representation precedent for continuous directional visibility functions. Do not let neural rendering / relighting literature dominate this subsection.

### Subsection synthesis

The gap is not:

> Can visibility be learned?

That is already established.

The gap we care about is:

> Can geometry-only scene context be compiled by a shared model into a compact transferable visibility representation whose predictions remain conservatively interpretable at a fixed decision boundary on unseen geometry?

---

## 2.3 Visibility-Aware 3D and Web Streaming

### Visibility-guided transmission is not new

Core reference:

- *Smart Visible Sets for Networked Virtual Environments* (2002).

It already connects:

```text
PVS
  ↓ direction / distance / importance
network transmission priority
```

Therefore the scheduler must not be presented as the headline novelty.

Correct claim:

> The scheduler is an application of the proposed signal; the research contribution is making an occlusion-aware signal available before detailed geometry residency.

### View-dependent progressive geometry streaming

Representative references:

- Streaming QSplat.
- View-Dependent Streaming of Progressive Meshes.
- Streaming HLODs.

What they establish:

- view-dependent transmission ordering;
- coarse-to-fine refinement;
- importance-driven progressive geometry delivery.

Common operating point:

- the client typically owns a hierarchy, proxy, coarse representation, or refinement structure.

These papers show that “what should arrive next?” is a long-standing graphics problem.

### Web3D pipelines

Important reference:

- *Fine-Grained Web3D Culling-Transmitting-Rendering Pipeline* (CGI 2023).

It already unifies culling, network transmission, and browser rendering. Therefore we must not imply that combining PVS and Web streaming is unprecedented.

The distinction is the **source and residency requirement of the visibility signal**.

### 3D Tiles as practical background

3D Tiles demonstrates that pre-content request decisions from compact metadata are standard practice:

- bounding volumes;
- hierarchy;
- geometric error / SSE;
- viewer request volume;
- content URI.

Useful framing:

```text
metadata + camera → request decision
```

Our intended addition is:

```text
compact occlusion metadata + camera region → occlusion-aware relevance
```

Position the method as complementary to frustum/SSE/hierarchy selection rather than replacing it.

---

## Final synthesis paragraph for Related Work

The section should end by making three facts explicit:

1. PVS can guide streaming — long established.
2. Visibility can be computed online or learned — established.
3. Web clients can make pre-content decisions from lightweight metadata — established.

The missing operating point addressed here is their intersection:

> **client-queryable learned from-region visibility from compact geometry-compiled metadata before detailed content residency.**

---

## Do not create standalone Related Work subsections for

- PointNet / point encoders;
- graph neural networks;
- attention;
- spherical harmonics;
- survival analysis;
- generic “deep learning in graphics”.

These are implementation ingredients, not the research neighborhood.

---

## Literature-comparison template

For every paper, record:

| Dimension | Question |
|---|---|
| Visibility type | from-point or from-region? |
| Scene availability | what representation is required at query time? |
| Computation location | server, preprocessing pipeline, renderer, client? |
| Representation | full geometry, proxy, voxel/froxel, PVS table, latent? |
| Query output | binary PVS, primitive visibility, continuous score? |
| Generalization | scene-specific preprocessing, retraining, shared model? |
| Streaming use | none, filtering, selective transmission, ordering? |
| Main cost | storage, preprocessing, GPU time, candidate count, startup bytes? |

---

## Claim discipline

Avoid:

- “all previous PVS methods need the full detailed mesh”;
- “no previous work applies visibility to streaming”;
- “NeuralPVS cannot be used for Web streaming”;
- “ours is the first learned visibility model”.

Prefer:

- “requires a sufficiently detailed runtime scene representation”;
- “does not directly address the client bootstrap condition considered here”;
- “operates at a different point in the content-residency pipeline”;
- “we focus on the intersection of learned visibility and pre-content delivery”.
