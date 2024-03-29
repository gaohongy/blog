---
title: 算法复杂性分析
subtitle:
description:
keywords:
summary:
license:
date: 2023-12-22T15:26:55+08:00
lastmod: 2023-12-23T18:25:30+08:00
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

## 渐进符号及相应定理
1. $O$

$f(n)=O(g(n))$ 称 g(n)是 f(n)的一个渐近上界

当且仅当存在正的常数 c 和 $n_0$，使得对于所有的 $n \ge n_0$， 有 $f(n) \le cg(n)$。

> $O$比率定理：对于函数 $f(n)$ 和 $g(n)$，如果极限 $\lim \limits_{n \to \infty}\frac{f(n)}{g(n)}$存在，则$f(n) = O(g(n))$ 当且仅当存在正的常数 c，使得 $\lim \limits_{n \to \infty}\frac{f(n)}{g(n)} \le c$

2. $\Omega$

$f(n)=\Omega(g(n))$ 称 g(n)是 f(n)的一个渐进下界

当且仅当存在正的常数 c 和 $n_0$，使得对于所有的 $n \ge n_0$， 有 $f(n) \ge cg(n)$。

> $\Omega$比率定理：对于函数 $f(n)$ 和 $g(n)$，如果极限 $\lim \limits_{n \to \infty}\frac{f(n)}{g(n)}$存在，则$f(n) = \Omega(g(n))$ 当且仅当存在正的常数 c，使得 $\lim \limits_{n \to \infty}\frac{f(n)}{g(n)} \ge c$

3. $\Theta$

$f(n)=\Theta(g(n))$ 称 g(n) 和 f(n) 同阶

当且仅当存在正的常数 $c_1$, $c_2$ 和 $n_0$，使得对于所有的 $n \ge n_0$， 有 $c_1g(n) \le f(n) \le c_2g(n)$。

> $\Theta$比率定理：对于函数 $f(n)$ 和 $g(n)$，如果极限 $\lim \limits_{n \to \infty}\frac{f(n)}{g(n)}$ 和 $\lim \limits_{n \to \infty}\frac{g(n)}{f(n)}$ 均存在，则$f(n) = \Theta(g(n))$ 当且仅当存在正的常数 $c_1$, $c_2$，使得 $\lim \limits_{n \to \infty}\frac{f(n)}{g(n)} \le c_1$, $\lim \limits_{n \to \infty}\frac{g(n)}{f(n)} \le c_2$

4. $o$

$f(n) = o(g(n)) 当且仅当 $f(n) = O(g(n))$ 和 $g(n) \ne O(f(n))$, 即$g(n)$是$f(n)$的上界，但反之不成立

一般情况下，$f(n) = o(g(n))$ 的充要条件为 $\lim \limits_{n \to \infty}\frac{f(n)}{g(n)} = 0$

## 常用渐进函数阶的比较
对于$\forall$ 给定正实数 c, p, 和 $\varepsilon$，满足以下不等式

1. $\exists \ n_0$, s.t. $\forall \ n > n_0$, $(c \cdot logn)^p < (logn)^{p + \varepsilon}$

> 幂函数底数扩大系数使函数增大的速度不如扩大指数，不管系数扩大多少倍

2. $\exists \ n_0$, s.t. $\forall \ n > n_0$, $(c + n)^p < n^{p + \varepsilon}$

> 和1类似

3. $\forall$ 实数 $y$, $\exists \ n_0$, s.t. $\forall \ n > n_0$, $n^p(logn)^y < n^{p + \varepsilon}$

> 引申一下就是 $\forall$ 实数 $y$, $\exists \ n_0$, s.t. $\forall \ n > n_0$, $(logn)^y < n^{\varepsilon}$

4. $\exists \ n_0$, s.t. $\forall \ n > n_0$, $n^p < 2^n$ 

> 幂函数不论指数多大，增长速度都不及指数函数
