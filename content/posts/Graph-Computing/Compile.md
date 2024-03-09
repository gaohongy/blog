---
title: Compile
subtitle:
description:
keywords:
summary:
license:
date: 2024-03-07T21:26:29+08:00
lastmod: 2024-03-09T21:55:18+08:00
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

目前涉及到两个关键的概念：MLIR 和 LLVM[^LLVM]。LLVM 通过引入 IR 的概念，减轻了传统编译器前后端之间的强耦合关系。与此同时也凸显出了模块化的概念，通过 IR 可以自由实现前后端的组合。而MLIR和IR是同一类别的事物，差别在于其更加通用可扩展，可以用于描述和表示程序的结构和语义信息。

如果想要利用MLIR实现图计算的相关编译工作，可行的思路就是将MLIR转换为LLVM IR（LLVM的中间表示形式），然后利用LLVM提供的优化器和代码生成器将LLVM IR编译成目标平台的机器代码，简单来说就是利用LLVM的后端，这样，MLIR可以利用LLVM的强大优化和代码生成能力，为不同的编程模型和应用场景提供高效的编译器支持。

[^LLVM]: [LLVM](https://gaohongy.github.io/blog/posts/compile-link/c-c++-compile-link/#llvm)

Just a guess:

gc: Graph Convolution（图卷积）
gnn: Graph Neural Networks（图神经网络）
gpm: 


在了解 MLIR 过程中发现使用到了一个名为 TableGen 的工具，有一种说法是它是 一种声明式编程语言。似乎通过相关工具，可以将.td文件生成C++代码。粗浅理解似乎是为了能够运用某些公共组件

## 对于项目的理解
从关键技术那张图上，能够了解到CGA_Framework就是那个统一编程框架

王老师说我们做图学习还有编译的部分

我目前对于统一编程框架的理解是：目前项目中的include下面的各种.h文件提供的就是C++接口的功能，而apps则是使用这些接口的示例程序，apps就应当是等价于实际使用这个统一编程框架的用户所编写的代码

统一编程框架位于框架层，要注意到其上还存在一个应用层，应用层和框架层的接口应当是 GraphScope 和 DGL。也就是说用户仍然是利用GraphScope 和 DGL，来完成图计算、图挖掘和图学习的任务，但是开发者需要提供从GraphScope和DGL到统一编程框架的转换机制（**对应着任务中的“用户透明的代码自动转换机制"**)，目前猜测自动代码生成技术就是来负责实现这种转换的

这种转换应该是在存在着统一编程框架和Graph
现在涉及到几个关键词，自动代码生成，领域特定语言DSL，中间表达IR

从领域特定语言的作用机制来说，库从某种意义上来说本身就是一种DSL，如何从DSL转到另一种语言，这里如果借助IR，似乎再通过llvm IR以及llvm backend，那就直接变成了可执行程序，所以没有想到如何实现这种语言的转换，所以猜测的可能是通过automatic code generation，但是还并不了解这个自动代码生成的机制，不过用户透明的代码自动转换机制肯定是通过某种机制来实现的

编译层的作用可能是实现高层次代码转换为中间表达IR，逐层下降逐层优化

## 如何理解 MLIR
MLIR 用于解决编程语言、编译器和硬件之间的交互问题，它的出现是为了应对日益复杂的编程语言和硬件架构

MLIR提供了一个统一的中间表示（IR），可以作为不同编程语言编译器和LLVM后端之间的桥梁。

通过自定义 dialect，可以实现语言扩充 和 实现特定领域的编译优化