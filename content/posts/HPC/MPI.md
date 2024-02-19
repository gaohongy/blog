---
title: MPI
subtitle:
description:
keywords:
summary:
license:
date: 2023-12-12T21:49:28+08:00
lastmod: 2024-02-20T00:19:38+08:00
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

## MPI Install

```bash
./configure -prefix=/home/hongyu_gao2001/mpi/mpi-install --disable-fortran
make
make install
```

If we execute the command `man mpicc` or `man mpicxx`, we can get two command line arguments `-compile_info` and `-link_info`, which are used to show the steps for compiling a program(what options and include paths are used) and show the steps for linking a program(what options and libraries are used).

If we use the `mpicc -compile_info` or `mpicc -link_info`, we can get the following compile options:

```bash
-I/home/hongyu_gao2001/mpi/mpi-install/include 
-L/home/hongyu_gao2001/mpi/mpi-install/lib 
-Wl,-rpath 
-Wl,/home/hongyu_gao2001/mpi/mpi-install/lib 
-Wl,--enable-new-dtags 
-lmpi
```

> For a detailed explanation of the above options, please refer to the [C C++ Compile Link](https://gaohongy.github.io/blog/posts/compile_link/c-c++-compile-link/#%E7%BC%96%E8%AF%91%E6%B5%81%E7%A8%8B).

Acording to the result of the option `mpicc -compile_info` or the `mpicc -link_info`, we can learn about that the mpicc and mpicxx is just a wrapper of gcc and g++, them add extra options for gcc and g++.

And `-prefix=` option and `make install` make the mpicc and mpicxx carry the compile option according to the content of the `-prefix=`, which is equal to the gcc or g++ which carry these optinos manually.

According to the [rpath | wikipedia](https://en.wikipedia.org/wiki/Rpath#cite_note-1:~:text=readelf%20%2Dd%20%3Cbinary_name%3E%20%7C%20grep%20%27R.*PATH%27%20displays%20the%20RPATH%20or%20RUNPATH%20of%20a%20binary%20file.%20In%20gcc%2C%20for%20instance%2C%20one%20could%20specify%20RPATH%20by%20%2DWl%2C%2Drpath%2C/custom/rpath/.), we can use the `readelf -d <binary_name> | grep 'R.*PATH'` to verify the role of the compile option `-Wl`.

```bash
$ gcc mpi_hello_world.c -I/home/hongyu_gao2001/mpi/mpi-install/include -L/home/hongyu_gao2001/mpi/mpi-install/lib -lmpi -o mpi_hello_world_norpath

$ ./mpi_hello_world_norpath 
./mpi_hello_world_norpath: error while loading shared libraries: libmpi.so.12: cannot open shared object file: No such file or directory

$ readelf -d mpi_hello_world_norpath  | grep 'R.*PATH
```

```bash
$ mpicc mpi_hello_world.c -o mpi_hello_world

$ ./mpi_hello_world 
./mpi_hello_world: error while loading shared libraries: libmpi.so.12: cannot open shared object file: No such file or directory

$ readelf -d mpi_hello_world  | grep 'R.*PATH
0x000000000000001d (RUNPATH)            Library runpath: [/home/hongyu_gao2001/mpi/mpi-install/lib]
```

The GNU Linker (GNU ld) implements a feature which it calls "new-dtags", which can be used to insert an rpath that has lower precedence than the LD_LIBRARY_PATH environment variable.[^new-dtags]

[^new-dtags]: [The role of GNU ld](https://en.wikipedia.org/wiki/Rpath#cite_note-1:~:text=The%20GNU%20Linker%20(GNU%20ld)%20implements%20a%20feature%20which%20it%20calls%20%22new%2Ddtags%22%2C%20which%20can%20be%20used%20to%20insert%20an%20rpath%20that%20has%20lower%20precedence%20than%20the%20LD_LIBRARY_PATH%20environment%20variable.)

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
MPI_Recv的问题在于只能提前开辟尽可能大的store buffer，在接受完消息后再通过status得知消息长度。替代方案是首先使用 MPI_Probe 获取消息长度，然后根据消息长度开辟存储空间，再使用 MPI_Recv 接收消息，避免空间浪费（不过这只是逻辑说法，实际连接起这些概念的都是status，MPI_Probe也是将即将接收到的消息状态存储到status数据结构中）

## CUDA-Aware MPI
[An Introduction to CUDA-Aware MPI](https://developer.nvidia.com/blog/introduction-cuda-aware-mpi/)

## Reference
> - [1] [MPI Tutorial](https://mpitutorial.com/tutorials/)
