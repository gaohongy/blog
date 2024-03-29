---
title: Greedy Algorithm
subtitle:
description:
keywords:
summary:
license:
date: 2023-12-24T18:28:42+08:00
lastmod: 2023-12-24T23:33:25+08:00
tags:
categories:
  - Algorithm
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

## 贪心策略正确性证明

### 归纳法

> 归纳法证明的核心逻辑：证明采用贪心策略执行到任意步数时获得的解都是最优解的一部分 
> 
> 数学表达: 算法执行到第k步时$(k \in N^+)$，一定存在一个最优解包含这k步按照贪心策略选择的解

1. 证明 k = 1时，一定存在一个最优解包含算法第一步的选择

假设存在一个最优解，其中第一步的选择与贪心算法的选择不同，然后根据题目条件证明将第一步的选择替换为算法的选择依然是一个最优解，这样就证明一定存在一个最优解包含了算法第一步的选择

2. k = m 时，假设命题成立，要证明 k = m + 1 时，命题成立

k = m 时，命题成立，那么可以得到一个包含贪心算法前 k 步选择的一个最优解，并且通过反证法可以证明当前解中剩余步骤得到的结果也是最优的

根据归纳假设，对于剩余的问题，一定存在一个最优解会包含贪心算法在第 m + 1 步的选择

对于剩余问题来说，上述得到的两个解都是最优解，因为可以用第2个包含贪心算法第m+1步选择的最优解去替换第一个最优解，这样就可以得到一个包含贪心算法在前m+1步选择的一个最优解，从而命题得证

形式化表达就是
$$
T = \{贪心算法前m步的选择\} \cup T' \\
T^* = \{贪心算法第m+1步的选择\} \cup T''
$$

T'是剩余问题的一个最优解，$T^*$也是剩余问题的一个最优解，不过其包含了贪心算法在第m+1步的选择

既然都是最优解，那么就可以用$T^*$替换T‘，得到$\{贪心算法前m步的选择\} \cup \{贪心算法第m+1步的选择\} \cup T'$这样一个包含贪心算法前m+1步选择的一个最优解，从而命题得证

### 交换论证法

> 证明的逻辑在于：要证明贪心策略是正确的，首先给出不符合贪心策略的解，通过交换某些元素使得解变得更优，说明重复操作下去最终得到的形式和贪心策略相符就证明了贪心策略是正确的

1. 首先给出解的形式，以及最优化的度量值公式（例如要求最短时间，那么时间就是度量值）
2. 给出一个非最优解的形式，计算非最优解的度量
3. 交换非最优解中的某些元素，得到一个新的解，再次计算度量值
4. 计算新的解和非最优解之间度量值的差值，说明新的解是更优的
5. 说明如果可以进行这种交换，那么只要重复交换下去最终得到的最优解的形式和贪心策略相同