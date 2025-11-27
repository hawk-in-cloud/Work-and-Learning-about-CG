## I. 隐式场基础（NeRF 系列）——理解 3DGS 的起点
| 时间            | 论文                                                      | 贡献               | 为什么必须读                                    |
| ------------- | ------------------------------------------------------- | ---------------- | ----------------------------------------- |
| **2020**      | **NeRF: Representing Scenes as Neural Radiance Fields** | 隐式辐射场，体渲染公式，位置编码 | 3DGS 的数学基础来自 NeRF 的体渲染与 view-dependent 表达 |
| **2020–2021** | NeRF++ / Mip-NeRF                                       | 多分辨率、抗锯齿         | 理解空间、方向编码为何重要                             |

你会从这一步理解：   
3DGS 本质是 NeRF 的“显式体积 + 点化表示 + 高速 rasterization 替代 ray marching”。

### II. NeRF → 显式场（Explicit Field）转变的关键前置知识
| 时间       | 论文              | 关键思想                   | 为什么重要                                  |
| -------- | --------------- | ---------------------- | -------------------------------------- |
| **2021** | **PlenOctrees** | NeRF 转八叉树体素 + SH       | 这是“NeRF 显式化”的开端，直接影响 3DGS 的 SH 表示与加速思路 |
| **2022** | **Plenoxels**   | 直接优化显式体素（density + SH） | 和 3DGS 类似：不训练 MLP，直接优化显式参数             |
| **2022** | **TensoRF**（可选） | 分解高维场，加速 NeRF          | 理解“表示优化”如何影响渲染速度                       |

显式化路线直接连接到 3DGS 的设计理念：  

不用 MLP，把场景变为可优化的显式结构 → 才能实时化。

### III. 点云方向（Point-based Neural Field）——3DGS 的最直接前驱
| 时间       | 论文                                        | 思想                    | 与 3DGS 的关系                            |
| -------- | ----------------------------------------- | --------------------- | ------------------------------------- |
| **2022** | **Point-NeRF**                            | 用点云作为显式 anchor + 神经特征 | 3DGS 是“点 + 球谐（SH）”的更高效版本              |
| **2023** | **Neural Point Light Fields (NPLF)**      | 点光场，splatting + 路径可微  | 3DGS 的可微 splatting 与 NPLF 架构极其相似      |
| **2023** | **Differentiable Splatting / Surfels 系列** | 显式点/surfel splat 渲染   | 3DGS 本质是“高斯 surfel 的可微 rasterization” |

这些论文构成 3DGS 的“点场表示思想”来源。

### IV. 3D 高斯点云（核心阶段）
| 时间                 | 论文                                                               | 内容                                                        | 必读原因             |
| ------------------ | ---------------------------------------------------------------- | --------------------------------------------------------- | ---------------- |
| **2023（SIGGRAPH）** | **3D Gaussian Splatting for Real-Time Radiance Field Rendering** | 高斯点云、可微 SRP（Splat Rasterization Pipeline）、显式 SH 颜色模型、极快训练 | 你毕设的核心。全部概念来自这里。 |

看完 3DGS 后，你会理解：

* Covariance → ellipse 投影
- SH → 方向相关颜色
- Tile-based → 实时
- α-blending → 近似体积渲染

### V. 3DGS 改进 / 扩展方向（供你毕设做创新）
| 时间   | 论文                                       | 创新点                         | 你可能做的毕设方向        |
| ---- | ---------------------------------------- | --------------------------- | ---------------- |
| 2023 | **Gaussian Surfels**                     | 用 surfel 替代 Gaussian，提高几何精度 | 改进几何的精度/法线一致性    |
| 2024 | **Compact 3DGS / GS-Lite**               | 更轻量的 GS（减少点数）               | 做“优化点云数量/内存”的论文  |
| 2024 | **4D Gaussian Splatting / Dynamic 3DGS** | 支持 dynamic scene            | 做“动态高斯”或“骨骼驱动高斯” |
| 2024 | **Triangular Gaussian Splatting**        | 用三角高斯减少排序开销                 | 做“透明排序加速”        |
| 2024 | **Gaussian Opacity Fields**              | 更好的透明模型                     | 做“玻璃、水、材质优化”     |


你的毕设创新点可以直接从这些论文中选方向。

### VI. 工程实现（引擎/渲染层要补的知识）  
| 主题                       | 关键知识                                 | 为什么要看                    |
| ------------------------ | ------------------------------------ | ------------------------ |
| **可微渲染**                 | weighted OIT, depth blending         | 3DGS 的 blending 本质是透明度渲染 |
| **GPU-driven rendering** | Compute culling / tile binning       | 3DGS 的实时性核心来自 GPU 驱动     |
| **透明渲染 OIT**             | weighted blended OIT / depth peeling | 3DGS 是“透明物体 + 多层累积”      |
| **Vulkan / DX12**        | tile-based pipeline + bindless       | 你要做一个引擎化 Demo，需要底层 API   |

### VII. 完整路线图（图示版）
```
───────────────────────────────────────────────
               3DGS 必读论文路线图
───────────────────────────────────────────────

(1) 隐式场 NeRF 基础
     ├─ NeRF (2020)
     ├─ NeRF++ / Mip-NeRF
     └─ Instant-NGP (2022) ← NeRF 加速关键

(2) 显式体积方向（NeRF → Explicit）
     ├─ PlenOctrees (2021)
     ├─ Plenoxels (2022)
     └─ TensoRF (可选)

(3) 点云显式方向（Point-based）
     ├─ Point-NeRF (2022)
     ├─ Neural Point Light Fields (2023)
     └─ Differentiable Surfels / Splatting 系列

(4) 3DGS 主体
     └─ 3D Gaussian Splatting (SIGGRAPH 2023)

(5) 3DGS 改进方向
     ├─ Gaussian Surfels
     ├─ GS-Lite / CompactGS
     ├─ 4D Gaussian Splatting
     ├─ Triangular GS
     └─ Gaussian Opacity Field

(6) 工程基础（非论文但必须补）
     ├─ OIT / Transparency
     ├─ GPU-driven rendering
     └─ Vulkan / DX12 / Compute Shader
───────────────────────────────────────────────
```