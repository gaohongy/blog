---
title: Computer Architecture
subtitle:
description:
keywords:
summary:
license:
date: 2023-12-01T11:46:15+08:00
lastmod: 2023-12-03T22:57:19+08:00
tags:
categories:
  - Computer-Architecture
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

## 解决 Branch Hazards 的n种方法
### static branch prediction
1. freeze of flush the pipeline (predicted-not-taken)

2. treat every branch as not taken

3. treat every branch as taken
> This buys us a one-cycle improvement when the branch is actually taken, because we know the target address at the end of ID, one cycle before we know whether the branch condition is satisfied in the ALU stage.

三种方式下一个非常重要的问题就是**在哪个阶段确定的是分支指令? 在哪个阶段确定的分支指令的目标地址?**
这个问题有点影响因素：
1. 指令类型不同
2. 硬件的设计方式不同
3. 选择的预测机制不同

无条件分支指令在ID阶段就可以确定是分支指令以及目标地址, 这个不需要看硬件设计和预测机制

有条件分支指令在一般设计需要到EX阶段结束后能够确定目标地址，如果是高级设计则可以提前到ID阶段确定目标地址。注意说的是可以，在采用预测未选中机制时，还是采用EX阶段确定目标地址的方式，在采用预测选中机制时，需要在ID阶段完成

4. delayed branch
一个很重要的情况是: 这种技术手段主要用在早期没有分支预测的流水线 RISC 上，现代 RISC 实现早就可以在流水线的第 2 级利用分支预测确定跳转的目标，分支延迟槽也就失去了原来的价值, 所以说在考虑分支延迟的时候，应该是不用考虑分支预测的方式的

分支延迟槽的作用机理就是把无论分支最终是否选中后续都会执行的指令放到分支指令之后，使得当这部分指令执行结束后恰好就能确定分支指令的目标地址了，然后就可以顺次执行下去，使用这部分指令掩盖或者代替了确定分支指令目标地址之前停顿的时间

branch delay slot will contains the instructions which are executed whether or not the branch is taken.

architectures with delay branches often disallow putting a branch in the delay slot.

