# 03A — 统一 View-Cell、GT 与在线查询协议

本文档是论文中关于 **view-cell、离线可见性 GT、候选集合与在线查询** 的统一 source of truth。

后续所有章节如果涉及：

- view-cell 定义；
- camera sampling；
- PVS ground truth；
- candidate generation；
- dataset split；
- runtime query；

都必须服从本文档。


---

# 1. 可见性单位

论文统一以 **renderable unit** 为可见性预测与剔除单位。

## BIM 场景

unit = BIM 构件。

## 标准图形学场景

使用冻结的 deterministic partition rule，把场景切成可独立剔除的 renderable units。

每个 unit 至少具有：

- unique unit ID；
- world-space AABB；
- geometry；
- 渲染所需的基础元数据。

论文核心方法只关心：

> unit 是否属于当前 view-cell 的 potentially visible set。

---

# 2. 合法区域中心

每个场景先按照登记规则选择合法 region centers。

中心采样应覆盖：

- 可通行空间；
- 建筑附近；
- 开阔区域；
- 不同高度层。

并检查相机中心与场景 geometry 的安全距离。

允许：

> 不同场景使用不同的合法中心生成规则。

但必须：

> 对每个场景登记并冻结中心生成规则。

这样可以适配：

- BIM campus；
- indoor graphics scenes；
- village / city；
- multi-level environments；

而不强迫所有场景共享不合理的同一空间采样器。

---

# 3. 离线观察方向

标准协议对每个 region center 使用：

## 水平角

$$
0^circ, 90^circ, 180^circ, 270^circ
$$

## 俯仰角

$$
-15^circ, 0^circ, 15^circ
$$

因此每个物理中心展开：

$$
4	imes3=12
$$

个固定观察方向。

定义：

> **一个 region center + 一个固定方向 = 一个 view-cell。**

这 12 个方向只用于：

- 离线数据覆盖；
- dataset split 中的同中心分组；
- 训练 / evaluation workload。

它们**不是前端相机必须吸附的离散方向**。

在线查询可以使用当前真实 camera direction。

---

# 4. View-Cell 的空间区域：水平圆盘

论文主协议只使用：

> **固定方向 + 世界 XZ 平面中的水平圆盘。**

对于每个 scene，登记圆盘半径：

$$
r_s.
$$

view-cell 中：

- camera orientation 固定；
- camera height 固定；
- camera position 只在水平圆盘内变化。

高空 / 不同楼层视角通过：

> 额外设置更高或更低的 region center

表示。

不在单个 view-cell 内改变高度。



---

# 5. 离线区域位置采样：32 个 Camera Positions

每个 view-cell 的区域 GT 使用：

$$
32
$$

个 camera positions。

包括：

- 圆盘中心；
- 其余位置在圆盘面积内做 deterministic / frozen area-uniform sampling。

所有 32 个位置共享：

- 相同 camera direction；
- 相同 pitch / yaw；
- 相同投影参数；
- 相同 camera height。

注意：

> 这 32 个相机只用于离线构建 region GT。

前端 runtime **不会渲染这 32 个相机**。

---

# 6. 区域 Ground Truth

在每个 view-cell 的 32 个位置上，以：

$$
mathrm{FOV}_y = 66^circ
$$

执行硬件 Color-ID rendering。

对 unit (u_i)：

## Region Visibility Label

如果它在至少一个采样位置中可见：

$$
y_i(mathcal B)=1.
$$

因此：

$$
mathrm{PVS}(mathcal B)
=
igcup_{m=1}^{32}
V(c_m).
$$

这里表示：

> **区域内可能可见（potentially visible in the sampled region）**

而不是：

> center camera 的瞬时 visible set。

## Visual Weight

对于 unit (u_i)，其区域 visual weight 使用：

$$
w_i(mathcal B)
=
max_{m=1,ldots,32}
w_i(c_m),
$$

即 32 个 Color-ID samples 中最大屏幕覆盖贡献。

论文必须明确：

> 这是 sampled regional ground truth，不是对连续圆盘 visibility 的数学穷举证明。

---

# 7. Candidate Set 独立生成

候选集合不能通过：

> “GT visible IDs + 一些额外对象”

构造。

它必须独立由 camera/view geometry 得到。

对于圆盘半径 (r_s)，从 region center 沿当前固定观察方向向后移动：

$$
Delta =
rac{r_s}{	an 30^circ}.
$$

在该后退位置构造：

$$
66^circ
$$

FOV 的 candidate frustum，并对所有 unit AABB 做 frustum test。

得到：

$$
mathcal C(mathcal B).
$$

必须检查：

$$
mathrm{GT}_{visible}
subseteq
mathcal C(mathcal B).
$$

如果出现：

$$
u_iin mathrm{GT}_{visible}
quad	ext{但}quad
u_i
otin mathcal C,
$$

该样本 / candidate protocol 应视为失败。

禁止：

> 为了通过检查而把漏掉的 GT ID 人工补进 candidate。

同时必须在论文中准确说明：

> 该 inclusion check 只验证被冻结的 32 个 sampled positions，并不构成对连续圆盘所有位置的形式化 visibility coverage proof。

---

# 8. View-Cell 样本打包

一个 view-cell 对应一条训练 / evaluation sample。

其中保存：

- region center；
- frozen direction；
- disk radius；
- camera projection parameters；
- candidate unit IDs；
- visible unit IDs；
- visual weights。

使用 CSR / columnar representation 只是实现形式，不属于论文算法贡献。

---

# 9. Train / Calibration / Validation / Test 划分

split 必须按：

> **physical region center**

分组。

同一个物理中心展开出的：

- 12 个方向；
- 每个方向内部的 32 个 GT camera positions；

必须全部属于同一个 split。

禁止：

> 同一 physical center 的不同方向跨 train / validation / test。

这样可以避免空间上高度相关的相机区域跨 split 泄漏。

---

# 10. 在线前端的一次 View-Cell Query

runtime 输入：

- current camera position；
- current camera orientation；
- registered disk radius；
- projection parameters。

在线流程：

```text
当前 camera position + orientation
        ↓
建立当前 view-cell anchor
        ↓
沿当前朝向后退 r / tan(30°)
        ↓
66° AABB frustum
        ↓
candidate units
        ↓
一次 batch visibility query
        ↓
per-unit visibility scores
        ↓
冻结阈值得到 regional PVS
        ↓
当前真实 60° frustum
        ↓
当前真正需要显示 / 处理的 units
```

关键点：

> 前端 camera direction 不需要吸附到离线数据的 12 个方向。

12 个方向只是：

- training / evaluation coverage protocol。

runtime 使用：

> 当前真实 camera orientation。

---

# 11. View-Cell Query 的复用条件

一次 regional visibility prediction 可以在以下条件满足时复用：

1. camera 仍然位于该 anchor 的水平圆盘内；
2. camera orientation / projection parameters 仍满足当前 view-cell 的冻结条件。

当：

- camera 走出圆盘；
- direction 改变到需要重新建立 region anchor；
- projection contract 改变；

则重新执行一次 batch query。

论文中不要把这个机制写成：

> 前端每帧都重新跑 32 个 sampled views。

实际在线 query 是：

> **一次区域级批量预测。**

---

# 12. V5 内部 9 点与离线 32 个 Camera Samples 的区别

这是最容易混淆、必须明确写清楚的地方。

## 32 个位置

作用：

> **离线生成区域 GT。**

它们需要真正进行 Color-ID rendering。

## 9 个 support points

作用：

> **V5 单次 query 内部，从已经编译好的 structured field 提取 region features。**

它们是：

- disk center；
- 8 个圆周 support points。

V5 在这些 support 上只做：

> analytic field evaluation

并得到：

$$
S_{center}, S_{max}, S_{mean}, S_{min}.
$$

它们不是：

- 9 个额外 rendered cameras；
- 9 次完整 neural inference；
- 9 个独立 view-cells。

因此可以记成：

```text
32 = offline GT camera samples
 9 = one V5 query's internal analytic supports
 1 = one runtime batch model query per view-cell
```

---

# 13. 60° 与 66° 的统一语义

## 66°

用于：

- offline GT Color-ID sampling；
- region candidate generation；
- PVS model query contract。

目的：

> 对真实显示视锥提供保守余量。

## 60°

用于：

> 当前真实 rendering / display frustum。

因此 runtime 的语义是：

```text
66° regional PVS
    ↓
current 60° display frustum
    ↓
instantaneous visible/render set
```

不能把二者混为同一个 FOV。

---

# 14. 当前实现与论文协议的关系

当前已部署 V4：

- 已实现一次 batch query 路径；
- region gate 当前按 HKUST 的 2 m 水平圆盘实现。

V5：

- 目前 Python evaluation 中实现了：
  - center + 8 circumference supports；
  - analytic field statistics；
- 尚不能写成“V5 browser deployment 已完成”。

因此最终论文中：

- **协议定义**可以统一为本文档；
- **system implementation status**仍必须区分 V4 / V5。

---

# 15. Source Code References

当前协议对应的主要代码位置：

- 标准场景方向计划：  
  `neural_instance_culling/tools/graphics_scene_importer/generate_pose_plan.mjs`
- GT / candidate 构建：  
  `neural_instance_culling/dataset/build_rvc_viewcell_pose_csr.py`
- 当前 V4 前端 region gate：  
  `slm2viewer/src/CameraPredictionGate.js`
- 当前 V4 Worker batch query：  
  `slm2viewer/src/LightweightPVSWorker.js`

论文写作时引用这些实现语义即可，不必在正文写具体源码行号。

---

# 16. 论文统一用语

统一使用：

- **renderable unit**
- **region center**
- **view-cell**
- **horizontal disk**
- **offline GT camera positions**
- **candidate units**
- **regional PVS**
- **runtime batch query**
- **V5 analytic support points**

避免使用：

- “32 runtime views”
- “9 camera queries”
