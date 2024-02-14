---
title: Parallel Computing
subtitle:
description:
keywords:
summary:
license:
date: 2024-01-29T23:41:59+08:00
lastmod: 2024-02-14T21:06:37+08:00
tags:
categories:
  - HPC
weight: 0

password:
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

## Motivation
Moores's Law states that, when the price remains constant, the number of components per chip that can be accommodated approximately doubles every 18-24 months, leading to a corresponding doubling of performance.

The phenomenon of failure of Moores's Law means that now we use more transistors to increse core number instead of frequency per chip(performance per chip).

So it is the developing tendency that how to use multi-cores to complete the computation tasks now, which is called parallel-computing.

## Classification
### Shared Memory Parallel Programming
1. Manual: multi-threading
2. Automatic: OpenMP

### Distributed Memory Parallel Programming
1. Message passing: e.g. MPI

并行编程模型和并行计算机的底层硬件结构是相互对应的，是有一种硬件结构从而对应到一种并行编程模型，对应关系如下：

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202402051845492.png)


如何理解编程模型所处的层次结构
以下图多线程编程模型为例，编程模型层或者说抽象层给使用者提供的接口就是“线程”，向底层则对应到pthread等多种类型的多线程库实现，再向下则是OS系统调用API，然后到达硬件层

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202402051822525.png)

## Programming model Definition
It is made of languages and libraries that create abstract view of the machine.

The key concepts for creating a parallel programming model:

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202402051844780.png)

## Performance Evaluation
Two important factor:

1. what needs to be done
2. what it costs to do it

## Data Parallel
一般认为，向量化 和 并行 还是有一点差别的，并行更多强调的是线程级或者说核一级的并行，向量化指代的更多是在一个核心上通过向量化指令和底层支持向量化的硬件结构实现的SIMD

## Example 
[High-Performance Matrix Multiplication](https://gist.github.com/nadavrot/5b35d44e8ba3dd718e595e40184d03f://gist.github.com/nadavrot/5b35d44e8ba3dd718e595e40184d03f0)
