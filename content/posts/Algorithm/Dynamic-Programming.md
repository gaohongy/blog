---
title: Dynamic Programming
subtitle:
description:
keywords:
summary:
license:
date: 2023-12-26T12:27:18+08:00
lastmod: 2023-12-26T23:31:50+08:00
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

定义状态转移方程的关键在于对状态做出完整表示，举个例子，最长上升子序列问题中，不同的状态是一些子序列，以子序列的最后一项作为划分依据就可以完整表示不同的状态

当然二维结构的理解起来不会这么直观，例如0/1背包，f[i][c]前i个物品且背包容量为c时的物品选择，不如一维结构的定义明了

## 最优子结构证明
1. 考虑解的形式
2. 给出一个原问题的最优解，给出一个比原问题小一级问题的子问题，以及这个子问题的最优解
3. 反证法，假设上面提到的这个子问题的解并非最优，存在一个更优的解
4. 说明使用这个更优的子问题的解可以构造出一个更优的原问题的解，和最初的假设原问题的最优解矛盾,从而说明上面给出的子问题的解是最优的