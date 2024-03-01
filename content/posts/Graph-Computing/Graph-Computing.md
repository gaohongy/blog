---
title: Graph Computing
subtitle:
description:
keywords:
summary:
license:
date: 2024-02-29T15:59:15+08:00
lastmod: 2024-03-01T21:53:47+08:00
tags:
categories:
  - Graph-Computing
weight: 0

password: woshinidaye
message:

draft: false
repost:
  enable: false
  url:

hiddenFromHomePage: false
hiddenFromSearch: false

comment: false
lightgallery: force
---

## Task
1. 领域专用模型 -> 图计算通用模型 （中间代码实现转换，类似 llvm 提供的中间层）
2. dynamic graph data control
3. 编译优化提升性能

## 传统图计算解决方案不足
缺少一个可扩展的通用系统来解决大型图的计算问题

传统算法的典型问题：

1. 比较差的内存访问局部性
2. 针对单个顶点的处理工作过少
3. 计算过程中伴随着并行度的改变

## 通用图计算软件
- 基于遍历算法的、实时的图数据库 : Neo4j、OrientDB、DEX 和 InfiniteGraph
- 以图顶点为中心的、基于消息传递批处理的并行引擎 : Hama、Golden Orb、Giraph 和 Pregel

## BSP 模型

简单理解就是分布式内存模型

全称：整体同步并行计算模型（Bulk Synchronous Parallel Computing Model）

又名：大同步模型、桥模型

1次BSP计算 <=> 1系列全局超步（一次迭代）：
1. 局部计算
2. 通信
3. 栅栏同步

## Graph neural network

### Category
- recurrent graph neural network
- convolutional graph neural network
- graph autoencoder
- spatial-temporal graph neural network


## Graph Convolution Network

GNN 和 CNN 等传统神经网络相比，关键在于它的用武之地在于不规则格式的输入数据，传统神经网络处理的都是规则格式的输入数据

图神经网络的关键在于对点或者边所表示的特征进行重构，挖掘出其中包含的信息。一个常见的词汇是 embedding，表示将节点或者边的特征数据表示为，或者说映射为向量格式，以便进行运算

GNN 的存储结构是 无向图的邻接矩阵，是一个稀疏矩阵

Euclidean data 和 non-Euclidean data[^non-Euclidean-data]。需要理解什么是欧几里得空间，大致可以理解为坐标空间

[^non-Euclidean-data]: [Geometric deep learning: going beyond Euclidean data](https://arxiv.org/abs/1611.08097)

多层 GNN 的作用在于扩大节点的感受野，把远距离节点的信息传播到当前节点，使得所有节点接收到的信息都是全局的信息

图任务的特点是：并非所有节点或边都存在标签，即图任务并非完全的有监督学习，它是掺杂了部分无监督学习的

### GCN 计算思想
1. 计算每个节点的特征信息，将一个节点本身的特征信息和邻居节点的特征信息进行聚合，通过全连接层获取该节点的特征向量（并不是仅仅把节点自身的特征信息作为特征向量，因为还要考虑邻居节点的情况）
2. 涉及到的矩阵：adjacency matrix-A, degree matrix-D, feature vector-F

3. 邻接矩阵A 和 特征矩阵F 相乘 表示得到一个节点的相邻节点的信息

![](https://img2024.cnblogs.com/blog/1898659/202403/1898659-20240301205406282-415755068.png)

4. 由于一个节点还需要考虑自身的信息，所以会将 A 加上一个单位矩阵，形成 A'，A‘ 和 F 的乘积才能表示一个节点聚合到的包含自身在内的所有信息

![](https://img2024.cnblogs.com/blog/1898659/202403/1898659-20240301205318372-659134034.png)

5. 由于聚合后数据量较大，所以采用度矩阵的逆矩阵（里面的数据变为了倒数），起到一个求平均的作用，$\widetilde{D}^{-1}(\widetilde{A}X) = (\widetilde{D}^{-1} \widetilde{A})X$

![](https://img2024.cnblogs.com/blog/1898659/202403/1898659-20240301205520301-57883426.png)

6. 但是仅仅左乘这个度逆矩阵，仅仅对行起到作用，并没有对列起到作用，所以再右乘一次, $\widetilde{D}^{-1} \widetilde{A} \widetilde{D}^{-1} X$

![](https://img2024.cnblogs.com/blog/1898659/202403/1898659-20240301210935832-168640913.png)

7. 左右各乘一次度数逆矩阵，这样的问题是这个求平均或者说归一化的过程进行了2次，所以改进为 $\widetilde{D}^{-\frac{1}{2}} \widetilde{A} \widetilde{D}^{-\frac{1}{2}} X$

这样做两次的意义，或者说这个归一化的过程的实际意义：如果只进行邻接矩阵和特征矩阵的乘法，很容易造成的一个问题是一个节点聚合到的信息有误。假设节点A与节点B相邻，节点A的度数为1，即节点A仅和节点B相邻，但是节点B的度数为1000，显然我们不能将B的特征等价于A的特征，但是如果仅仅采用邻接矩阵和特征向量，很容易得出一个结论就是节点A和节点B的特征相似。但是假设一个现实场景，节点B代表一个富二代，节点A代表一个穷人但是和节点B是同学，不应该得到的结论是节点A和节点B相似。

做两次归一化操作可以避免这个问题，因为 $\widetilde{D}^{-\frac{1}{2}} \widetilde{A} \widetilde{D}^{-\frac{1}{2}} = \frac{1}{\sqrt{degree(v_i)} \sqrt{degree(v_j)}} A$，这个等式不太好理解，实际的含义是A中的$a_{ij}$这个元素会变为$a_{ij} \frac{1}{\sqrt{degree(v_i)} \sqrt{degree(v_j)}}$，观察上述图片，左乘表示对一行的数据进行操作，右乘表示对一列的数据进行操作，实际上对于A中的$a_{ij}$，与其发生运算的数据就是第i个节点的度数和第j个节点的度数的倒数平方根，即公式中的那一部分

在节点A和B的示例中，当我们左右乘上了$\widetilde{D}^{\frac{1}{2}}$，由于节点B的度数较大，那么同A中元素相乘的数据就比较小，这恰恰符合节点A和节点B的特征信息相似度低的情况

## Legion

### Hierarchical graph partitioning mechanism
Improves the multi-GPU cache performance

### Unified multi-GPU cache
Minimize the PCIe traffic incurred by caching both **graph topology** and **features** with the highest hotness

### Automatic cache management mechanism
Adapts the multi-GPU cache plan 

according to the hardware specifications and various graphs 

to maximize the overall training throughput.