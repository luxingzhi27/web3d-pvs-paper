# 08 — Discussion and Limitations（讨论与局限）

## 1. Candidate-Count Dependence

V5 的 runtime query cost 随 candidate unit 数增长。

这是一个真实 trade-off。

与固定 voxel/froxel grid 方法相比：

- NeuralPVS 类方法的成本更依赖 grid resolution；
- V5 更接近 per-candidate (O(N))。

因此论文必须：

- 实测 candidate-count scaling；
- 不要隐藏这一点；
- 说明 Web candidate filtering / hierarchy 可以先缩小 query set。

---

## 2. Startup Visibility Asset 也随 Unit 数增长

因为是 per-unit representation：

[
\text{asset bytes}
\propto
N_{unit}.
]

这和 streaming granularity 形成 trade-off：

### unit 更小

优点：

- visibility 更细；
- scheduling 更精确。

缺点：

- unit 数更多；
- visibility metadata 更大；
- runtime query 更多。

这应该成为 Discussion 的正式内容，而不是只在实验表格里出现。

---

## 3. Dynamic Geometry

当前 compiled field 主要描述静态场景遮挡结构。

动态变化可能使：

- local descriptor；
- AABB relation；
- compiled field；

失效。

未来方向可以包括：

- local recompilation；
- dynamic occluder residual；
- 与 runtime HZB 混合。

在没有实现前，不要声称完整支持 dynamic scenes。

---

## 4. AABB Proxy 对复杂遮挡体的表达能力有限

relation graph 有意采用轻量 AABB proxy。

因此可能更难处理：

- 很薄的几何；
- porous structure；
- foliage；
- 栅栏；
- 高度非盒状 occluder；
- projection overlap 很大但实际遮挡很少的形状。

这不是单纯 implementation bug，而是 representation trade-off。

---

## 5. Low-Order Directional Field 的容量限制

Full 使用固定一阶方向 basis：

[
[1,d_x,d_y,d_z].
]

优点：

- compact；
- smooth；
- fixed runtime shape；
- analytic query；
- 方向结构可解释。

代价：

- angular frequency 有限；
- 复杂方向遮挡可能表达不足。

Generic-28、未来 higher-order field 都可以帮助解释这一 trade-off。

---

## 6. Safety–Efficiency Frontier

任何 conservative PVS 都存在：

[
\text{Recall}\uparrow
\Rightarrow
\text{Cull Efficiency}\downarrow
]

的基本 trade-off。

论文不应该追求“一个单独指标全胜”。

更合理的表达：

> 在满足安全约束的前提下最大化 useful culling / streaming utility。

---

## 7. Generalization Claim 的边界

即使完成：

- LOSO；
- Bistro external blind holdout；

也只能说明：

> 在测试过的多种场景分布上具有有意义的 cross-scene transfer。

不能声称：

- arbitrary scene；
- open-world；
- 任意 geometry distribution；
- 动态世界。

---

## 8. Survival Field 的语义

analytic survival field 是：

> structured intermediate statistic。

它不是：

> 精确物理 object visibility probability。

最终 visibility 仍由：

- field statistics；
- geometry descriptor；
- query geometry；

共同预测。

---

## 9. V4 / V5 System Evidence 必须严格分开

当前 V5 的 model evidence 比 system deployment 更完整。

如果 V5 browser runtime 尚未完全闭环：

- V4 runtime 只能标为 legacy / historical system evidence；
- 不能和 V5 model result 拼成一个“ours”；
- 最终 CGF 稿最好在投稿前完成 V5 end-to-end runtime 与 streaming evaluation。
