# 06 — Progressive Web3D Integration（Web 渐进式传输集成）

## 本节目标

这一节要证明：

> GCOF-PVS 的 representation 不是为了单纯做一个离线分类 benchmark，而是为“客户端在 detailed geometry residency 之前做 occlusion-aware content decision”设计的。

同时要避免把 scheduler 本身包装成主要算法创新。

---

# 6.1 Offline Scene Compilation

对于一个新场景：

```text
detailed geometry
   ├─ surface sampling
   │      ↓
   │     z_i
   │
   └─ AABBs
          ↓
   proxy relation graph
          ↓
   frozen shared compiler
          ↓
      field C_i
          ↓
  runtime visibility asset
```

核心 deployment property：

> 新场景的 visibility asset 由 geometry-only preprocessing + frozen shared model 生成，不需要 target-scene visibility labels，也不需要 target-specific fine-tuning。

这一点是 V5 与普通 scene-specific PVS database / visibility distillation 的重要区别。

---

# 6.2 Browser Runtime

浏览器最终只需要：

- compact per-unit geometry descriptor；
- 4×7 structured field；
- AABB / unit metadata；
- unit → resource mapping；
- shared lightweight query-head weights。

浏览器不运行：

- 256-point geometry encoder；
- proxy relation graph builder；
- relation compiler。

运行时链路：

```text
current view-cell
    ↓
candidate units
    ↓
9 support analytic field query
    ↓
visibility logits
```

论文最终必须报告：

> 实际导出 runtime asset 的真实字节数。

不能只拿 FP16 理论估算冒充最终资产大小。

---

# 6.3 Instance-to-Resource Aggregation

模型预测单位是 instance / renderable unit，但网络下载单位往往是 GLB/resource。

如果多个实例映射到同一 GLB，可以采用：

[
p_g
=
\max_{i:g(i)=g}
p_i.
]

直觉：

> 只要同一资源中的任意一个实例当前高度可见，下载这个资源就可能有收益。

最终论文必须以实际 implementation 的 aggregation rule 为准。

---

# 6.4 Filtering 与 Ranking 必须分开

这是系统评价里非常重要的概念边界。

## Threshold Filtering

使用：

- fixed zero；
- 或冻结的 safe threshold；

输出：

> 哪些 candidate resource 可以 defer / filter。

评价：

- retained GLB count；
- retained bytes；
- coverage ceiling；
- visible utility miss。

## Threshold-Free Ranking

完全不先过滤 candidate set。

而是：

> 在同一个 candidate GLB 集合上改变下载顺序。

评价：

- Bytes@95；
- Bytes@99；
- Bytes@99.9；
- waste-before-99；
- bandwidth-converted time。

不能把一个已经 threshold-filtered 的小集合拿去和完整集合的 ranking baseline 比较。

---

# 6.5 Cost-Aware Priority

系统还可以使用：

[
\frac{p_g}{B_g^\alpha}.
]

其中：

- (p_g)：visibility relevance；
- (B_g)：resource bytes；
- (alpha)：cost sensitivity。

论文必须同时报告：

### Pure visibility

[
p_g.
]

### Cost-aware visibility

[
p_g/B_g^\alpha.
]

这样才能区分：

> 收益来自 visibility 预测，

还是：

> 只是因为偏爱小文件。

(alpha) 必须在 validation 上选定，不能看 test。

---

# 6.6 与 3D Tiles / Web Selection 的关系

GCOF-PVS 不应该被写成替代：

- frustum；
- hierarchy；
- SSE / geometric error；
- cache policy；
- 3D Tiles traversal。

而是补充一个现有 pre-content metadata 通常缺少的信号：

> **occlusion-aware relevance before detailed content residency**

可以把现代 Web selection 理解为：

```text
camera
+ bounding volumes
+ hierarchy
+ SSE
+ cache
    ↓
candidate resources
```

GCOF-PVS 在 candidate selection / scheduling 中增加：

```text
compact occlusion metadata
+ view region
    ↓
visibility relevance
```

因此它更适合作为：

> 可插拔的 visibility signal，

而不是重写整个 Web streaming stack。
