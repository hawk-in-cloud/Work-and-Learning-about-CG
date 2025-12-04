# Mip-NeRF: A Multiscale Representation for Anti-Aliasing Neural Radiance Fields

**Paper:** Mip-NeRF: A Multiscale Representation for Anti-Aliasing Neural Radiance Fields  
**Authors:** Jonathan T. Barron, Ben Mildenhall, Matthew Tancik, Peter Hedman, Ricardo Martin-Brualla, Pratul P. Srinivasan  
**Conference / Year:** ICCV 2021  
**Paper Link:** https://arxiv.org/abs/2103.13415  
**Code Link:** https://github.com/google/mipnerf  

---

## 一句话总结（One-Sentence Summary）

Mip-NeRF 通过引入 **圆锥体采样（conical frustum）** 和 **积分位置编码（Integrated Positional Encoding, IPE）**，解决了 NeRF 中的 aliasing(走样) / 多尺度不一致问题，新模型视觉效果更好。

---

## 1. 背景与问题（What & Why）

### 论文解决的问题
NeRF 使用点采样，会产生多种 aliasing（锯齿、闪烁、模糊）问题：
- 渲染距离远的物体不稳定  
- 缩放图像时产生明显 artifacts  
- 多分辨率训练 / 渲染不一致  

NeRF 不能像传统图形学那样使用 mipmap，因为它是隐式函数，没有真实纹理。

### 为什么重要
Mip-NeRF 提出第一个真正有效的 NeRF anti-aliasing 方案，使 NeRF 能在不同分辨率、不同摄像机距离下表现稳定，是后续 NeRF 系列的基础。

---

## 2. 方法（How）

Mip-NeRF 的核心方法：

---

### (1) 圆锥体采样（Conical Frustum Sampling）

普通 NeRF 采样“点”，即：
ray(t) = o + td
但像素对应的是一个 **面积（footprint）**。    
![圆锥采样](./Pics/圆锥采样.png)     
Mip-NeRF 使用：

> **圆锥截体（frustum）代替点采样**

- 离相机近 → 小截体  
- 离相机远 → 大截体  

使采样更加符合成像原理，避免 aliasing。

---

### (2) Integrated Positional Encoding（IPE）

NeRF 的位置编码是对点做 Fourier Features。  
Mip-NeRF 改为对 **区域（圆锥截体）** 求积分期望：
$$
\gamma(x) = \int \gamma(x) p(x)\, dx
$$

其中各个变量含义：
* x：区域内的所有真实 3D 点  
* p(x)：该区域的概率密度(一般是一个高斯分布（还是在 3D 上），在一个截面上)   
* γ(x)：点的高频位置编码（同nerf）   

---

作用：更平滑、理论更正确，避免高频 aliasing。

---

### (3) Multi-scale 表示

IPE 对不同截体大小自然形成分层（mip-level）：

- 大 footprint（距离） → 低频（远景）  
- 小 footprint → 高频（近景）

相当于 implicit mipmapping（隐式纹理贴图）。

---

### (4) 保持 NeRF 原结构不变

仍保留：

- 体渲染公式  
- MLP 网络结构  
- 方向编码  

只改变输入编码与采样方式。    
从本质上来看，Mip-Nerf的创新实质上只有一项（即对采样方法由点采样转换为区域采样）     
也许，3DGS也是如此，从点采样变成高斯球采样？

---

## 3. 创新点（Key Contributions）

1. **提出圆锥体采样**，让采样反映像素面积（footprint）。  
2. **提出 IPE（积分位置编码）**，解决 aliasing 的核心技术。  
3. **实现 implicit mipmapping**，多分辨率一致性显著提升。  
4. **无需改变 MLP 结构即可大幅提升渲染质量**。

---

## 4. 实验（Experiments）

### 数据集
- Synthetic NeRF dataset  
- Real Forward-Facing dataset  
- LLFF dataset  
- 多尺度缩放数据集  

### 对比实验
Mip-NeRF 在以下方面明显优于 NeRF：

- 渲染远景更清晰  
- 缩放视图稳定性提高  
- 几乎无 aliasing  
- 多尺度一致性更好  

### 消融实验
- 去掉 IPE → aliasing 严重  
- 去掉 frustum → 高低分辨率性能下降  

---

## 5. 局限性（Limitations）

- 训练速度比 NeRF 稍慢  
- 渲染速度也略慢  
- 仍然无法达到实时  
- 依然不具备场景泛化能力  

---

## 6. 个人理解与思考（My Notes）
- 因为本质是滤波处理，所以画面在高频情况不一定效果比Nerf好。
- Mip-NeRF 是从**点采样 → 面积采样**的改进。  
- IPE 位置编码认为 NeRF 不应编码“点”，而应编码“区域的期望值”。  
