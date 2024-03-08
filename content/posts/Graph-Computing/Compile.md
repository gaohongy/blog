---
title: Compile
subtitle:
description:
keywords:
summary:
license:
date: 2024-03-07T21:26:29+08:00
lastmod: 2024-03-08T16:35:12+08:00
tags:
categories:
  - Graph-Computing
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

目前涉及到两个关键的概念：MLIR 和 LLVM[^LLVM]。LLVM 通过引入 IR 的概念，减轻了传统编译器前后端之间的强耦合关系。与此同时也凸显出了模块化的概念，通过 IR 可以自由实现前后端的组合。而MLIR和IR是同一类别的事物，差别在于其更加通用可扩展，可以用于描述和表示程序的结构和语义信息。

如果想要利用MLIR实现图计算的相关编译工作，可行的思路就是将MLIR转换为LLVM IR（LLVM的中间表示形式），然后利用LLVM提供的优化器和代码生成器将LLVM IR编译成目标平台的机器代码，简单来说就是利用LLVM的后端，这样，MLIR可以利用LLVM的强大优化和代码生成能力，为不同的编程模型和应用场景提供高效的编译器支持。

[^LLVM]: [LLVM](https://gaohongy.github.io/blog/posts/compile-link/c-c++-compile-link/#llvm)

Just a guess:

gc: Graph Convolution（图卷积）
gnn: Graph Neural Networks（图神经网络）
gpm: 
