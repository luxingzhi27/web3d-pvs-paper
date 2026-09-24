# 00 — 论文定位

## 研究问题

本文研究 **pre-geometry from-region visibility**：

> 在详细场景几何尚未到达 Web 客户端之前，如何基于紧凑的预传输表示估计一个局部观察区域内的 potentially visible units？

该问题面向 progressive Web3D。客户端需要在内容尚未 resident 时决定哪些 units 更值得保留、处理或优先传输，而传统遮挡计算通常依赖一份已经可访问的场景表示。

## 核心思想

场景发布前，内容构建端已经拥有完整几何。本文将 geometry-dependent occlusion context 离线编译为紧凑的 per-unit visibility representation：

```text
scene geometry
    ↓ offline compilation
compact visibility representation
    ↓ client query
from-region visibility score
    ↓
unit-level culling / progressive ordering
```

运行时客户端不需要重新执行完整场景几何分析，只需针对当前 view-cell 查询已编译表示。

## 方法定位

方法由三部分组成：

1. **Geometry-only scene compilation**  
   使用局部表面几何和 potential-occluder relations 编译遮挡上下文。

2. **Compact structured visibility representation**  
   将方向相关遮挡信息压缩为可解析查询的 compact field，并输出 unit-level visibility score。

3. **Conservative PVS learning**  
   针对 visible miss 与 extra retention 的非对称代价训练模型，使预测适合安全优先的 PVS 使用场景。

## 论文贡献

1. 提出面向 progressive Web3D 的 pre-geometry from-region visibility 方法，使客户端可以在 detailed geometry residency 之前获得遮挡感知的 unit relevance。
2. 提出 geometry-compiled occlusion field，将 geometry-only local/context information 编译为紧凑、连续可查询的 per-unit visibility representation。
3. 在统一的 view-cell 协议下，从 PVS quality、cross-scene transfer、runtime/storage cost 和 progressive delivery 四个方面进行系统评价。

## 论文边界

本文不把以下内容作为独立创新点：

- visibility-guided streaming 本身；
- neural visibility 本身；
- from-region PVS 本身；
- Web 层次结构或 SSE-based selection。

本文关注的是这些方向在 **detailed geometry 尚未 resident** 的条件下如何结合。
