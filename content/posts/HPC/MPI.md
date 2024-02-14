---
title: MPI
subtitle:
description:
keywords:
summary:
license:
date: 2023-12-12T21:49:28+08:00
lastmod: 2024-02-14T21:06:37+08:00
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

mpicc 仅可编译 .c 文件，编译 .cpp文件会报错

MPI标准的不同实现：MPICH、MVAPICH、MVAPICH2、Open MPI

MPI编译器mpicc只是对于普通c/c++编译器的封装，在普通编译器的基础上添加了MPI相关文件的路径便于进行头文件引入和链接

mpiexec is a direct link to mpiexec.hydra

mpirun is a wrapper script that determines your batch system and simplifies the launch for the user, but eventually also calls mpiexec.hydra

## CUDA-Aware MPI
[An Introduction to CUDA-Aware MPI](https://developer.nvidia.com/blog/introduction-cuda-aware-mpi/)

## Reference
> - [1] [MPI Tutorial](https://mpitutorial.com/tutorials/)
