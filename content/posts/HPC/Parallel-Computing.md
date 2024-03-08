---
title: Parallel Computing
subtitle:
description:
keywords:
summary:
license:
date: 2024-01-29T23:41:59+08:00
lastmod: 2024-03-08T16:35:12+08:00
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

## Law
并行计算领域的两个关键定律就是 Amdahl 和 Gustafson，从不同角度诠释了 **加速比** 与 **系统串行化程度**、**CPU核心数** 之间的关系

### The difference between Amdahl's Law and Gustafson's Law
The reason for the discrepancy between the speed up predictions by the two laws is that[^AMDAHL-VS-GUSTAFSON]:

[^AMDAHL-VS-GUSTAFSON]: [AMDAHL VS. GUSTAFSON](https://infogram.com/amdahl-vs-gustafson-1g6qo2qq66nk278?ref=hackernoon.com#:~:text=The%20reason%20for%20the%20discrepancy%20between%20the%20speed,static%20no%20matter%20how%20much%20parallelization%20is%20available.)

1. Gustafson's Law assumes that the amount of work to be done increases as the number of processor increases!
2. Amdahl's Law, on the other hand, assumes the amount of work to be done is static no matter how much parallelization is available.

简单来说，就是 Amdahl 认为工作负载是不变的，但是 Gustafson 认为工作负载是变化的。

由于在 Amdahl 的假设中，工作负载不变，那么在计算加速比时就可以以加速前的时间处于加速后的时间

由于在 Gustafson 的观点中，工作负载是变化的，所以计算加速比更好的方法是通过比较相同时间处理的数据量

假设基准工作量是 1，并行比例是 f，串行比例是 1-f，则加速前并行部分工作量为 f，串行部分工作量为 1-f，加速后并行部分工作量为 f * p（p是处理器数量，因为是并行，所以不同处理器可以同时完成工作），串行部分工作量不变仍为 1-f（由于是串行，即使处理器增多也需要逐一执行，所以完成的工作量不变）

因此 Gustafson 定律得出的加速比为 $speedup = \frac{(1-f) + f \times p}{(1 -f) + f}$

### Contradiction

根据 Amdahl，即使处理器数量无限增大，加速比是存在一个上限的。但是 Gustafson 得出的结论却是 随着处理器数量增大，加速比可以无限增大。

两个定律的结论不同，这是不是说两个定律中有一个是错误的呢，其实不然，两者的差异其实是因为这两个定律对一个客观事实从不同角度去审视的后果，他们的侧重点不同。[^Contradiction]

Amdahl强调，当串行化比例一定时，加速比是有上限的，不管你堆叠多少个CPU参与计算，都不能突破上限。

而Gustafson 定律的出发点不同，对Gustafson定律来说，如果可被并行化的代码所占比例足够大，那么加速比就能随着CPU的数量线性增长。

所以这两者并不矛盾，从极端的角度来说，如果系统中没有可以被并行化的代码，那么对于两个定律，其 加速比是1，反之，如果系统中可被并行化的代码比例100%，那么两个定律得到的加速比都是处理器个数。

[^Contradiction]: [Whether the Amdahl's law and Gustafon's law contradict each other?](https://developer.aliyun.com/article/1093916#:~:text=%E5%BF%AB%E7%9A%84%E9%80%9F%E5%BA%A6%E3%80%82-,%E4%BA%8C%E3%80%81%E6%98%AF%E5%90%A6%E7%9F%9B%E7%9B%BE,-%E4%B8%A4%E4%B8%AA%E5%AE%9A%E5%BE%8B)

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
