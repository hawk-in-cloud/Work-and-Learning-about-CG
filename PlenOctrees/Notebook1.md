# PlenOctrees: Novel View Synthesis with Radiance-Field Octrees

**Paper:** PlenOctrees for Real-Time Rendering of Neural Radiance Fields  
**Authors:** Alex Yu, Vickie Ye, Matthew Tancik, Angjoo Kanazawa  
**Conference / Year:** ICCV 2021  
**Paper Link:** https://arxiv.org/abs/2103.14024  
**Code Link:** https://github.com/sxyu/PlenOctrees  
**Project Page:** https://alexyu.net/plenoctrees/

---

## 一句话总结（One-Sentence Summary）

PlenOctrees 改变了Nerf的8层MLP结构，改成了八叉树体素结构（Octree）+球谐系数（SH）辐射场，提高了运算速度，实现实时的nerf。

---

## 1. 背景与问题（What & Why）

### 论文要解决的问题：

NeRF 能渲染高质量的新视角画面，但存在重大工程缺陷：

1. **渲染速度极慢（每帧几秒 → 无法实时）**  
   因为每条光线需要对 MLP 进行数百次推理。

2. **MLP 计算冗余，难以部署**  
   全场景隐式编码 → 毫无空间索引结构 → 光线行进效率低。

3. **NeRF 训练好后只能 inference，仍然很慢**  
   缺乏显式空间加速结构（如 BVH / Octree）。

### 为什么重要？

很多场景应用都必须达到**实时渲染性能**。

PlenOctrees 是首个让 NeRF 实现**实时 FPS**的工作。

---

## 2. 方法（How）

PlenOctrees 的核心方法是：

> **将 NeRF 的辐射场（颜色 + 密度）从隐式 MLP 转换为显式八叉树结构（octree），并用球谐函数（SH）存储方向颜色。**

方法步骤如下。

---

### (1) 使用 NeRF 训练得到一个已收敛的辐射场

首先训练标准 NeRF（或 FastNeRF），得到：

- 密度 σ(x)  
- 颜色 c(x, d)

但不用于实时渲染，只用于后续 baking。

---

### (2) 对空间构建八叉树（Octree）

将场景空间切分为体素（voxel）：

- 稀疏空间 → 不存储  
- 高密度区域 → 分裂成更细体素  
- 节点深度自适应场景复杂度

等价于：

> NeRF 的连续场 → 分块为稀疏体素网格

---

### (3) 用球谐系数（Spherical Harmonics, SH）替代 MLP

NeRF 输出颜色依赖方向：

$$
c(x, d)
$$

PlenOctrees 用 SH 逼近方向相关颜色：

$$
c(x, d) \approx \sum_{i=1}^{m} SH_i(d)\, w_i(x)
$$

- 每个 voxel 节点存储 SH 系数 w  
- 不再需要方向 MLP → 实现快速 shading

这是非常关键的降维思想：

> **用 SH 取代 MLP，可大幅提升渲染速度。**

---

### (4) 密度直接存储在 voxel 节点中

每个 voxel 存储：

- σ（密度）  
- SH 颜色系数  

给光线行进提供快速查询。

---

### (5) 实时渲染时的流程

Ray → Octree traversal → accumulate σ & SH color → pixel

无需 MLP，无需复杂网络推理。

速度可达 **150 FPS**（Desktop GPU）。

---

## 3. 创新点（Key Contributions）

1. **提出 NeRF → PlenOctree 的 baking 技术**  
   让隐式场转换为显式八叉树数据结构。

2. **用球谐函数（SH）替代 MLP 输出方向颜色**  
   大幅减少计算量，方向依赖建模仍然准确。

3. **首次实现 NeRF 级画质的实时渲染**  
   性能提升两个数量级（100×）。

4. **提出空间稀疏化策略**  
   将密度低的 voxel 剪枝，显著压缩存储。

---

## 4. 实验（Experiments）

### 数据集
- Synthetic NeRF dataset  
- Real LLFF dataset  
- Tanks and Temples  

### 对比实验
与：

- NeRF  
- FastNeRF  
- Sparse voxel rendering  

相比：

| 方法 | FPS | 画质 |
|------|------|-------|
| NeRF | 1 FPS | 好 |
| FastNeRF | ~10 FPS | 略差 |
| PlenOctrees | 150 FPS | 略降但非常接近 |

PlenOctrees 画质略低于 NeRF（尤其边界细节），但速度提升巨大。

### Ablation
验证：

- 不使用 SH → 方向依赖缺失  
- 不使用 sparse octree → 存储膨胀  
- 过度压缩 voxel → 画质下降  

---

## 5. 局限性（Limitations）

1. **无法自适应更新（不可微）**  
   八叉树是 baked 结构，不像 NeRF 可以继续训练。

2. **显式结构导致内存占用较大**  
   比 MLP 需要更多空间。

3. **细节损失**  
   由于 voxel 的有限分辨率和 SH 的低频特性，极高频纹理略模糊。

4. **无法处理动态场景**  
   结构是静态构建的。

5. **训练仍需 NeRF**  
   baking 前仍要训练一个 NeRF（慢）。

---

## 6. 个人理解与思考（My Notes）

- PlenOctrees 的核心思想是：  
  **把 NeRF 的输出场外显化 → 用空间结构 + 低维 SH 代替 MLP**  
  与 3DGS 的思想非常接近。

- 它启发了后续大量显式神经场方法：  
  - Plenoxels  
  - Instant-NGP  
  - 3D Gaussian Splatting  
  - TensorRF

- 它证明了一个关键事实：  
  **NeRF 的 MLP 并不是必须的。只要区域内函数足够光滑，用显式结构也可以渲染。**

- 这也进一步表明：  
  **显式点/体/高斯结构才是实时渲染的未来，而不是隐式的 MLP。**

- 对你的 3DGS 毕设非常有参考价值：  
  - SH 替代方向网络  
  - 显式结构取代 MLP  
  - 稀疏空间结构  
  - 大规模场景渲染的可行路径  
