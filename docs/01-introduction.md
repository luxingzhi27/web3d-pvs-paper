# 01 — Introduction 写作方案

## 第 1 段：背景与应用

说明 visibility determination 和 from-region PVS 在交互式图形学中的作用。与单视点可见性相比，from-region PVS 可以覆盖局部相机运动范围，因此适用于 prefetching、remote rendering 和 progressive rendering。

引入 Web3D 场景：大型场景无法在启动阶段全部传输，客户端需要持续决定哪些 units 应优先获得和处理。

## 第 2 段：现有方法的能力与前提

概述 classical precomputed PVS、modern online PVS 和 NeuralPVS。现有方法已经显著降低了 from-region visibility 的计算成本，但通常假设执行 visibility computation 的一方能够访问一份足够表达遮挡关系的 scene representation。

避免把现有方法简单描述为“速度慢”。

## 第 3 段：Pre-Geometry Visibility Gap

在 progressive Web delivery 中，详细场景几何本身可能正是尚未传输的内容，因此存在循环依赖：

```text
visibility helps decide what to deliver
                ↑
scene representation is still unavailable
```

由此定义本文的问题：

> **pre-geometry from-region visibility**：在 detailed geometry residency 之前估计当前区域的 potentially visible units。

## 第 4 段：核心思想

内容构建端在离线阶段拥有完整 geometry，因此可以提前提取并压缩与遮挡相关的场景上下文。

本文将其编译为 compact visibility representation：

```text
geometry
    ↓ offline compilation
compact visibility asset
    ↓
client-side region query
    ↓
unit visibility
```

该表示应明显小于详细几何，同时保留足够的方向与遮挡信息。

## 第 5 段：方法概览

介绍 GCOF-PVS：

- 用共享 local geometry encoder 表示 unit 自身几何；
- 用 geometry-only potential-occluder relations 描述周围遮挡上下文；
- 将其编译为 compact directional occlusion field；
- 对当前 horizontal-disk view-cell 做轻量解析查询；
- 输出 per-unit visibility score。

由于 PVS 中漏掉可见单位的代价高于额外保留不可见单位，训练采用 safety-oriented conservative objective。

## 第 6 段：系统使用方式

客户端通过当前 camera 建立 view-cell，先由 AABB frustum 得到 candidate units，再执行一次 batched regional visibility query。预测结果用于 unit-level PVS filtering，并可作为 progressive content ordering 的遮挡相关性信号。

## 第 7 段：贡献与结果概览

建议贡献压缩为三点：

1. pre-geometry from-region visibility formulation and framework；
2. geometry-compiled compact occlusion representation and query；
3. unit-level Web3D evaluation covering visibility quality, generalization, runtime/storage cost and progressive delivery。

最后用一小段概括主要定量结果；数字只在正式实验冻结后填写。
