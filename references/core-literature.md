# Core Literature

This file tracks the papers that directly define the research neighborhood. Final BibTeX should be built from authoritative publisher/project pages.

## A. Classical / From-Region Visibility

### Teller & Séquin — Visibility Preprocessing for Interactive Walkthroughs (SIGGRAPH 1991)
Role:
- foundational cell/PVS work;
- establishes offline from-region visibility preprocessing.

### Wonka et al. — Visibility Preprocessing with Occluder Fusion for Urban Walkthroughs (2000)
Role:
- representative urban PVS preprocessing.

### Nirenstein & Blake — Hardware Accelerated Visibility Preprocessing using Adaptive Sampling (EGSR 2004)
Role:
- sampling-based aggressive regional visibility.

### Bittner et al. — Adaptive Global Visibility Sampling (SIGGRAPH 2009)
Role:
- mature global view-cell preprocessing;
- exploits spatial coherence.

## B. Online From-Region PVS

### Koltun, Chrysanthou, Cohen-Or — Hardware-Accelerated From-Region Visibility Using a Dual Ray Space (2001)
Role:
- directly relevant to networked walkthroughs;
- PVS-guided selective transmission precedent.

Important lesson:
> Do not claim that visibility-guided streaming is new.

### Camera Offset Space — Real-time PVS for Streaming Rendering (SIGGRAPH Asia 2019 / TOG)
Role:
- modern GPU online from-region PVS;
- directly related to streaming rendering.

### Guided Visibility Sampling++ (I3D 2021)
Role:
- ray/sampling-based online PVS.

### Trim Regions for Online Computation of From-Region PVS (SIGGRAPH 2023 / TOG)
Role:
- primary modern conventional comparison;
- explicitly discusses dynamic/streaming scenarios.

Comparison axis:
- has access to a runtime scene representation;
- our target is client-side pre-content visibility.

### Disocclusion Buffer (SIGGRAPH Asia 2025)
Role:
- recent highly parallel conventional PVS direction;
- prevents outdated “online PVS is too slow” framing.

## C. Learned Visibility

### Wang et al. — NeuralPVS: Learned Estimation of Potentially Visible Sets (SIGGRAPH Asia 2025)
Primary learned-PVS comparison.

Key points:
- froxelized runtime geometry input;
- sparse neural network;
- from-region output;
- synthetic data for generalization;
- fixed-volume runtime operating point.

Main distinction:
> runtime geometry-to-PVS inference vs. offline geometry-to-compact-visibility compilation.

### Neural Visibility of Point Sets (SIGGRAPH Asia 2025)
Role:
- per-element learned visibility precedent.

### NVGS: Neural Visibility for Occlusion Culling in 3D Gaussian Splatting (CVPR 2026)
Role:
- lightweight neural visibility queried before rasterization.

Main distinction:
> visibility-distilled asset for render-time culling vs. geometry-only compiled asset for pre-download selection.

### NeRV: Neural Reflectance and Visibility Fields for Relighting and View Synthesis (CVPR 2021)
Role:
- precedent for continuous directional visibility functions.

Keep concise because the task differs substantially.

## D. Streaming / Web3D

### Smart Visible Sets for Networked Virtual Environments (SIBGRAPI 2002)
Role:
- PVS + visual importance + network transmission order.

Critical novelty discipline:
> visibility-guided scheduling is not new.

### Streaming QSplat (I3D 2001)
Role:
- view-dependent progressive transmission of large models.

### View-Dependent Streaming of Progressive Meshes (2004)
Role:
- visual importance drives refinement transmission.

### Streaming HLODs (2004)
Role:
- hierarchical LOD + priority streaming.

### Fine-Grained Web3D Culling-Transmitting-Rendering Pipeline (CGI 2023)
Role:
- direct Web3D culling/transmission/rendering integration.

### OGC / Cesium 3D Tiles
Role:
- practical metadata-driven pre-content request decisions;
- bounding volumes + geometric error/SSE + hierarchy.

Our positioning:
> add pre-content occlusion relevance; do not replace hierarchy/SSE.

## Literature matrix fields

For every paper, record:

- task;
- from-point / from-region;
- required runtime scene representation;
- preprocessing requirement;
- computation location;
- output representation;
- learned or analytic;
- scene-specific or shared;
- use in streaming;
- exact difference to V5.
