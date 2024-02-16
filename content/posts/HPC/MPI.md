---
title: MPI
subtitle:
description:
keywords:
summary:
license:
date: 2023-12-12T21:49:28+08:00
lastmod: 2024-02-16T23:19:27+08:00
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

mpicc, mpic++, mpicxx
mpiexec, mpirun
mpichversion

mpicc 仅可编译 .c 文件，编译 .cpp文件会报错
mpic++ 编译 .cpp 文件

MPI标准的不同实现：MPICH、MVAPICH、MVAPICH2、Open MPI

MPI编译器mpicc只是对于普通c/c++编译器的封装，在普通编译器的基础上添加了MPI相关文件的路径便于进行头文件引入和链接

mpiexec is a direct link to mpiexec.hydra

mpirun is a wrapper script that determines your batch system and simplifies the launch for the user, but eventually also calls mpiexec.hydra

通信子 <=> 进程集合

MPI_Init 会初始化一个通信子communicator MPI_COMM_WORLD

## Hello World

```c
MPI_Init(NULL, NULL);

// Get the number of processes
int world_size;
MPI_Comm_size(MPI_COMM_WORLD, &world_size);

// Get the rank of the process
int world_rank;
MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);

MPI_Finalize();
```

## Send and Receive
MPI 提供了一些数据结构，对应C/C++中的基础数据结构，用于在更高层次指定消息结构

Receive 如果收不到消息会一直阻塞等待

```c
int world_rank;
MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);
int world_size;
MPI_Comm_size(MPI_COMM_WORLD, &world_size);

int number;
if (world_rank == 0) {
    number = -1;
    MPI_Send(&number, 1, MPI_INT, 1, 0, MPI_COMM_WORLD);
} else if (world_rank == 1) {
    MPI_Recv(&number, 1, MPI_INT, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
    printf("Process 1 received number %d from process 0\n", number);
}
```
## Dynamic Receiving
利用MPI_Recv的最后一个 MPI_Status 参数来获取接受到的消息的信息，以便应对动态长度的传输数据

## CUDA-Aware MPI
[An Introduction to CUDA-Aware MPI](https://developer.nvidia.com/blog/introduction-cuda-aware-mpi/)

## Reference
> - [1] [MPI Tutorial](https://mpitutorial.com/tutorials/)
