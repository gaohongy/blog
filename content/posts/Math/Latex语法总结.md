---
title: Latex语法总结
subtitle:
description:
keywords:
summary:
license:
date: 2021-09-23T20:50:00+08:00
lastmod: 2023-12-22T23:06:28+08:00
tags:
categories:
  - Math
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

## 点
1. 单个点 ($\cdot$) ： `$\cdot$`
2. 横向多个点 ($\cdots$) : `$\cdots$`
3. 竖向多个点 ($\vdots$) : `$\vdots$`
4. 斜向多个点 ($\ddots$) : `$\ddots$`

## 数学字体
> math**bb**：blackboard bold（黑板粗体），用于期望
> math**bf**：math boldface，用于矩阵、向量
> math**rm**：math roman，罗马体（整体），用于微分符号d、二项式系数C、等号上的def、自然常数e、虚数单位i
> math**cal**：math calligraphy（美术字），用于集合，分布

| `A` |      `$A$` |    `$\mathbb{A}$` |  `$\mathbf{A}$` |  `$\mathrm{A}$` |  `$\mathcal{A}$` |
| ---- | ----      | ----    | ---- | ---- | ---- |
|   A   |   $A$   |   $\mathbb{A}$   |  $\mathbf{A}$ |  $\mathrm{A}$ | $\mathcal{A}$ | 

> 参考
> - [Latex数学字体](https://www.jianshu.com/p/6de552393933)
> - [【latex】\mathbf{} \matrm{}](https://blog.csdn.net/dujuancao11/article/details/126089781)

## 公式标号
在**块级公式(`$$$$`)中**使用`\tag{自定义编号}`
引用使用`$(自定义编号)$`

## 表达式
> 将下标放置于文本正上/下方

`$\sum_{i=1}^n$`
$\sum_{i=1}^n$

`$\sum \limits_{i=1} \limits^n$`
$\sum \limits_{i=1} \limits^n$

需要注意的是其仅仅支持应用于数学的操作符,若需要在非数学操作符上下方添加文字,需要通过`\mathop`将表达式转化为数学操作符,示例如下

`$\mathop{minimize} \limits_{w,b} J(w, b)$`
$\mathop{minimize} \limits_{w,b} J(w, b)$

> 参考
> - [1] [LaTex中把下标置于文本正下方的方法](https://blog.csdn.net/da_kao_la/article/details/84836098)

## 数学模式重音符号1
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/20210923204247.png)

## 希腊字母
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/20210923204409.png)

## 二元关系
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/20210923204442.png)

## 二元运算
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/20210923204458.png)

## “大”运算符
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/20210923204514.png)

## 箭头
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/20210923204532.png)

## 定界符
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/20210923204544.png)

## 大定界符
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/20210923204559.png)

## 其它符号
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/20210923204614.png)

## 非数学符号
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/20210923204637.png)

## AMS定界符
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/20210923204655.png)

## AMS希腊和希伯来字母
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/20210923204712.png)

## AMS二元关系
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/20210923204759.png)

## AMS箭头
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/20210923204821.png)

## AMS二元否定关系符和箭头
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/20210923204837.png)

## AMS二元运算符
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/20210923204859.png)

## AMS其它符号
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/20210923204916.png)

## 数字字母
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/20210923204932.png)

## 矩阵
### 无括号
$$
\begin{matrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9
\end{matrix}
$$

```
$$
\begin{matrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9
\end{matrix}
$$
```

### 圆括号
> **pmatrix** (parentheses brackets matrix)

$$
\begin{pmatrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9
\end{pmatrix}
$$

```
$$
\begin{pmatrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9
\end{pmatrix}
$$
```

### 方括号
> **bmatrix** (brackets matrix)

$$
\begin{bmatrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9
\end{bmatrix}
$$

```
$$
\begin{bmatrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9
\end{bmatrix}
$$
```
### 花括号

$$
\begin{Bmatrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9
\end{Bmatrix}
$$

```
$$
\begin{Bmatrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9
\end{Bmatrix}
$$
```

### 单竖线括号
> **vmatrix** (vertical bar brackets matrix)

$$
\begin{vmatrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9
\end{vmatrix}
$$

```
$$
\begin{vmatrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9
\end{vmatrix}
$$
```
### 双竖线括号

$$
\begin{Vmatrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9
\end{Vmatrix}
$$

```
$$
\begin{Vmatrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9
\end{Vmatrix}
$$
```

> 参考
> - [1] [各种数学符号](https://blog.csdn.net/u012684062/article/details/78398191)
> - [2] [转义表示](https://www.math-linux.com/latex-26/faq/latex-faq/)

## 极限符号

1. $\lim_{n \to \infty}$

> `$\lim_{n \to \infty}$`

2. $\lim \limits_{n \to \infty}$

> `$\lim \limits_{n \to \infty}$`

3. $\displaystyle \lim_{n \to \infty}$

> `$\displaystyle \lim_{n \to \infty}$`

4. $\varlimsup \limits_{n \to \infty}$

> `$\varlimsup \limits_{n \to \infty}$`

5. $\varliminf \limits_{n \to \infty}$

> `$\varliminf \limits_{n \to \infty}$`

## 存在任意

1. $\forall$

> `$\forall$`

2. $\exists$

> `$\exists$`