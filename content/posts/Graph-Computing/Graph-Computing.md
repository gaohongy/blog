---
title: Graph Computing
subtitle:
description:
keywords:
summary:
license:
date: 2024-02-29T15:59:15+08:00
lastmod: 2024-02-29T21:55:29+08:00
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