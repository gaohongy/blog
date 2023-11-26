---
title: NP完全性分析
subtitle:
description:
keywords:
summary:
license:
date: 2023-11-26T12:58:07+08:00
lastmod: 2023-11-26T14:07:12+0800
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

resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
---

## 问题的分类
1. 最优化问题(optimization problem)
2. 判定问题(decision problem)
两类问题的关联在于：通常，通过对待优化的值强加一个界，就可以将一个给定的最优化问题转化为一个相关的判定问题了。同时从直观上来看，如果最优化问题容易，则对应的判定问题也是容易的。换言之，如果我们能够证明判定问题是困难的，则相当于也说明了对应的最优化问题是困难的.

**为什么如何重视判定问题 ？**
简单来说，判定问题的定义是明确且易表达易理解的，同时最优化问题是容易转换为判定问题的。

不过有一个疑问是为何只考虑最优化问题，在课程中提及的问题都是最优化问题，而是书籍中给出的示例一般也是最优化问题，但是一半问题中并非仅仅是最优化问题，不过似乎都是易转换为判定问题的

## 计算复杂度
两个重要工作：
1. 确定问题计算复杂度的一个上界
2. 确定问题计算复杂度的一个下界

## 证明
所有多项式时间内可验证的问题组成了NP类问题，其中能够在多项式时间内求解的为P类问题，能够由其他NPC问题归约得到的是NPC问题

P : 多项式时间内可求解，存在确定型多项式时间算法
NP：多项式时间内可验证，存在非确定型多项式时间算法（随便猜一种情况然后验证，把所有情况猜完一定可以找到解）


## NPC问题证明
### 证明的基本逻辑
当证明一个问题为 NP 完全问题时，我们是在陈述它是一个困难的问题。 我们并不是要证明存在某个有效的算法，而是要证明不太可能存在有效的算法。
