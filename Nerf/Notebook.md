## 关于Nerf的学习笔记

### Nerf是什么？
Nerf-Neural Radiance Field（神经辐射场）算法

论文官网：https://www.matthewtancik.com/nerf

关于Nerf的一篇写的不错的文章：https://blog.csdn.net/qq_45752541/article/details/130072505

* Nerf的定义：用一个MLP神经网络模型，去隐式地学习一个3D场景，最后输出能渲染出复杂环境的不同角度的视角图片。

MLP神经网络：多层感知机（MLP），也叫人工神经网络（ANN，Article Neural Network），除了输入输出层，中间可存在多层隐式连接层，最简单的ANN/MLP至少拥有三层网络结构。

MLP网络结构是全连接的，也就是相邻层的每个节点都相互连接。

结构图：

![MLP原理图](./Pics/MLP原理.png)



