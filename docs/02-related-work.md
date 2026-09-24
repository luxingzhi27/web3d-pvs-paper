# 2. 相关工作（Related Work）

## 2.1 遮挡剔除与区域潜在可见集

可见性算法可以按照观察域区分为单视点可见性（from-point visibility）和区域可见性（from-region visibility），也可以按照结果的保守性、预处理需求及场景表示进一步分类 [1](../references/core-literature.md#ref-01)。单视点方法针对当前相机剔除被遮挡的几何，而区域潜在可见集（PVS）需要保留观察区域内任一允许视点可能看到的内容。后者能够支持局部相机运动下的内容复用和预取，但必须处理更多视线之间的遮挡关系。

在几何已驻留的渲染流水线中，Hierarchical Z-buffer 利用层次深度表示拒绝被遮挡对象 [2](../references/core-literature.md#ref-02)；Coherent Hierarchical Culling 将空间层次、时间相干性与硬件遮挡查询结合，降低等待查询结果造成的流水线停顿 [3](../references/core-literature.md#ref-03)。CHC++ 进一步改进可见性预测和查询批处理 [4](../references/core-literature.md#ref-04)。Masked Software Occlusion Culling 则通过深度与覆盖信息的分离提高软件光栅化剔除效率 [5](../references/core-literature.md#ref-05)。这些方法主要减少已经可访问内容的渲染工作量；将其用于下载前决策时，还需提供用于生成遮挡信息的几何、代理或已有深度。

区域 PVS 的经典路线是预先划分观察空间并存储区域可见集合。Teller 与 Séquin 利用建筑模型中的空间单元和通道关系进行可见性预处理 [6](../references/core-literature.md#ref-06)。Wonka 等在城市场景中研究遮挡体融合，使多个遮挡体的共同作用参与区域剔除 [7](../references/core-literature.md#ref-07)。Nirenstein 等通过线空间构造精确的区域可见性计算 [8](../references/core-literature.md#ref-08)。与解析求解不同，硬件加速自适应采样 [9](../references/core-literature.md#ref-09)、Guided Visibility Sampling [10](../references/core-literature.md#ref-10) 和 Adaptive Global Visibility Sampling [11](../references/core-literature.md#ref-11) 通过采样、射线变异或跨观察区域的信息共享提高预处理效率。这些工作建立了预处理代价、可见集合紧致程度与存储规模之间的不同折中。预计算 PVS 本身即可支持不依赖详细几何的运行时查表，因此，本文的差异不在于首次将可见性计算前置，而在于使用紧凑、连续可查询的单元表示替代逐区域存储的集合结果。

在线区域 PVS 将计算移到实际查询时刻，从而适应变化的观察区域。Dual Ray Space 通过二维对偶射线表示与图形硬件计算区域可见性，并以大型环境的远程漫游为应用背景 [12](../references/core-literature.md#ref-12)。Camera Offset Space 将相机位移与逐片元信息结合，用于流式渲染中的实时 PVS 生成 [13](../references/core-literature.md#ref-13)。Guided Visibility Sampling++ 利用硬件光线追踪加速区域可见性的渐进采样 [14](../references/core-literature.md#ref-14)。Trim Regions 通过在图像空间侵蚀物体轮廓实现遮挡体收缩，并结合分层场景遍历计算一般三维场景的区域 PVS [15](../references/core-literature.md#ref-15)。Disocclusion Buffer 以量化深度的稀疏分层场景表示显式计算显露区域，将顺序相关的遮挡传播转化为可并行处理的显露计算 [16](../references/core-literature.md#ref-16)。

上述在线方法已直接面向流式或远程渲染，不能将“PVS 可用于传输”作为与它们的区别。本文关注可见性计算在客户端的数据前提：几何相关的上下文分析在内容准备阶段完成，客户端查询时仅使用已编译资产，而不再构建相应的场景光栅化或射线求交表示。

## 2.2 学习式可见性表示

逐元素可见性并不必然依赖显式表面重建。Katz 等提出的 Hidden Point Removal 通过点集变换与凸包计算估计给定视点的可见点，为直接在离散元素上处理可见性提供了基础 [17](../references/core-literature.md#ref-17)。逐元素表示为对象级或更细粒度的可见性预测提供了不同于图像深度查询的切入点。

NeuralPVS 是与本文最接近的学习式区域 PVS 方法 [18](../references/core-literature.md#ref-18)。该方法将场景几何转换为与扩展视锥对齐的 froxel 网格，通过稀疏卷积和三维交错表示预测区域可见网格，并使用合成场景进行训练。固定网格分辨率限制了神经推理的输入规模，但查询仍包含从可访问场景几何生成输入表示的步骤。与此不同，GCOF-PVS 将目标局部几何和周围遮挡关系离线编译为单元描述符及方向场；运行时输入为紧凑系数与当前观察区域。两者的区别在于场景上下文进入计算链的位置和查询表示，而非仅在于网络大小。

Neural Visibility of Point Sets 将点集可见性表述为学习问题，利用三维 U-Net 提取视点无关特征，再结合观察方向预测逐点可见性 [19](../references/core-literature.md#ref-19)。其计算对象是点集中的元素，与本文的独立剔除单元具有不同粒度。

NVGS 将已有三维 Gaussian 资产的视点相关可见性编码为轻量神经表示，在光栅化之前剔除不可见 Gaussian [20](../references/core-literature.md#ref-20)。这说明可见性知识可以通过离线处理换取低成本在线查询。NVGS 主要减少已驻留 Gaussian 的渲染开销，本文则估计局部观察区域中的单元可见性，以支持详细内容到达之前的选择与排序。二者的离线阶段均可利用场景信息，其任务差异不取决于是否生成目标场景的可见性标签。

在光照传输中，NeRV 以神经可见性场辅助反射与重光照计算，体现了连续位置—方向可见性函数的表示价值 [21](../references/core-literature.md#ref-21)。本文的方向生存场采用受约束的方向—距离函数，但其输出是用于 PVS 预测的中间遮挡特征，而非经过物理标定的光线透射概率。相关工作的共同基础是将几何相关的可见性查询转化为紧凑函数；本文进一步关注该函数的可传输性、区域查询代价和单元级剔除用途。

## 2.3 可见性感知的三维内容传输

渐进式几何表示为按需访问大型三维内容提供了基础。Progressive Meshes 使用基础网格与逐步细化记录表示几何 [22](../references/core-literature.md#ref-22)，视点相关细化进一步根据观察状态分配几何细节 [23](../references/core-literature.md#ref-23)。Streaming QSplat 基于多分辨率包围球层次，由客户端按当前视点从粗到细请求模型内容 [24](../references/core-literature.md#ref-24)。这些方法主要解决表示分辨率与内容到达顺序的协同问题。

可见性同样长期用于选择性传输。Dual Ray Space 已将在线区域可见性用于远程环境访问 [12](../references/core-literature.md#ref-12)。Smart Visible Sets 在区域 PVS 的基础上进一步按观察方向和距离组织内容，以适应客户端观察参数和重要性选择 [25](../references/core-literature.md#ref-25)。View-Dependent Streaming of Progressive Meshes 根据当前视点的视觉重要性调整细化数据的传输 [26](../references/core-literature.md#ref-26)；Streaming HLODs 将层次化细节表示与外存、网络访问结合，以已到达的粗层次表示支撑交互过程 [27](../references/core-literature.md#ref-27)。因此，将 PVS 或可见性分数用于传输优先级并非本文单独提出的机制；本文研究的是详细几何尚未到达时，客户端如何获得可连续查询的遮挡相关性。

面向 Web3D，Lightweight Progressive Meshes 联合构件相似性与渐进网格，以减少冗余几何并支持渐进访问 [28](../references/core-literature.md#ref-28)。CEBOW 将传输调度、缓存管理和初始加载组织到云—边缘—浏览器架构中 [29](../references/core-literature.md#ref-29)。Fine-Grained Web3D Culling-Transmitting-Rendering Pipeline 进一步协同服务器 PVS 剔除、网络传输与浏览器增量实例渲染 [30](../references/core-literature.md#ref-30)。这些工作表明，网络和渲染收益取决于整个流水线的协同；单独提高可见性分类指标，并不能替代对实际传输过程的评价。

3D Tiles 标准以空间层次、包围体、几何误差及请求相关元数据组织大规模场景，使客户端在尚未取得详细内容时即可进行访问与细化决策 [31](../references/core-literature.md#ref-31)。本文与这种元数据驱动的选择方式互补：在已有候选选择之上，加入可由紧凑资产查询的遮挡信息。方法的核心不是替换层次遍历或调度器，而是提供面向独立剔除单元的几何下载前区域可见性表示。

---

**参考文献：** 上述编号对应[参考文献目录](../references/core-literature.md)中的 31 项文献；BibTeX 见 [related-work.bib](../references/related-work.bib)。
