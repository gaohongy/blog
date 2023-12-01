---
title: Computer Architecture
subtitle:
description:
keywords:
summary:
license:
date: 2023-12-01T11:46:15+08:00
lastmod: 2023-12-01T12:21:21+08:00
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

## 分支预测
1. 正常执行的效果
2. 预测未选中
3. 预测选中

三种方式下一个非常重要的问题就是**在哪个阶段确定的是分支指令? 在哪个阶段确定的分支指令的目标地址?**
这个问题有点影响因素：
1. 指令类型不同
2. 硬件的设计方式不同
3. 选择的预测机制不同

无条件分支指令在ID阶段就可以确定是分支指令以及目标地址, 这个不需要看硬件设计和预测机制
有条件分支指令在一般设计需要到EX阶段结束后能够确定目标地址，如果是高级设计则可以提前到ID阶段确定目标地址。注意说的是可以，在采用预测未选中机制时，还是采用EX阶段确定目标地址的方式，在采用预测选中机制时，需要在ID阶段完成
