---
categories:
  - 并行计算
comment: false
date: '2023-05-31T11:17:35+08:00'
lastmod: 2023-12-18T18:10:06+08:00
description: null
draft: false
fontawesome: true
fraction: true
hiddenFromHomePage: false
hiddenFromSearch: false
keywords: null
license: null
lightgallery: force
math: true
message: null
password: null
repost:
  enable: false
  url: null
ruby: true
subtitle: null
summary: null
tags: []
title: GPU Structure and Programing(CUDA)
toc: true
weight: 0
---

## Todo
- [ ] L2 cache gpgpu-sim 源码分析
- [ ] Bank conflict 的题目分析
- [ ] warp occupancy 概念和计算 
- [ ] 由broadcast式访问global memory引申的对于constant memory的理解和使用
- [ ] 并行化+访存优化，并行化中有一个branch divergence的问题
- [ ] 查找 DRAM burst突发传送官方文档说明
- [ ] 发现矩阵乘法是一个结合各种并行算法以及CUDA硬件架构知识的好的入手点，create一门课程 “从矩阵乘法入门并行计算-CUDA版”
- [] 需要验证如果shared memory中的元素大小和bank大小不一致时，访问其中更小的数据是否会造成bank conflict。需要借助nvprof，但是nvprof在选择检测bank事件时无法正常工作

> - CUDA C只是对标准C进行了语言级的扩展，通过增加一些修饰符使编译器可以确定哪些代码在主机上运行，哪些代码在设备上运行
> - GPU计算的应用前景很大程度上取决于能否从问题中发掘出大规模并行性

## 宏观视角
高性能计算的第一性原理：访存优化。所有的努力（优化硬件设计，优化算法）都是在试图解决内存墙。

访存优化3大关键：
- 一是减少数据搬运
- 二是减少数据访存延时
- 三是保证负载均衡

GPU中的并行算法设计：设计block和thread的workload，搞清楚一个block负责哪部分的计算，一个thread要负责哪部分的计算。而设计的原则就是尽可能地减少访存，提高数据的复用概率，然后让所有的处理器都满负荷地进行工作，不能浪费。

## SIMD & SIMT

涉及到AVX指令，正在尝试
说明两种方式的区别

The two most important things about SIMD and SIMT are:
1. How is the SIMT to implement ?
2. How is the SIMD to calculate ?

GPU从整体上来说是SIMT，但是到Warp层次后实际就是SIMD了

## Kernel hardware mapping

kernel function -> GPU
block -> SM（one block can only be executed by one SM, but one SM can execute multiple blocks)
> 这里有一个疑问，一个SM可以执行多个block，多个block的执行是并发的还是可以并行的，换个角度来说就是正在执行的warp是同属于一个block的，还是可以隶属于多个block

关于此问题，还尚未找到官方资料中给出的证据，暂且认为SM上执行的warp可能属于不同block，也就是可以理解为开始执行kernel函数后，grid中的多个block被分配到某一个SM上，然后在SM的视角下就不再有block的概念，它所能看到的就是一些warp，通过warp scheduler来对warp进行调度，并且认为block一开始被分配到某个SM后不会在执行过程中去更换SM，直到执行完毕。

这种理解方式有一定的合理性，原因在于grid，block，warp本身就是为programmer所提供的逻辑概念，对于硬件来说，它能看到的不过是一些thread，它调度的也是thread。但是对于人来说，thread这一调度单位太细，很难实现对具体问题的抽象，所以才提供了更高一级的概念，即grid，block和warp。从这一角度来看，一个SM在同一时间能调度多个block就是合理的。

thread -> SP

main time consuming:

1. kernel function startup
2. thread block switch

## Hardware structure
Grid、Block are login concepts, they are created by CUDA for programmers.
According to the real physical level, every SM in GPU will excute multiple blocks, and it will divides block into multiple warps. The basic execution unit of SM is warp.

In fact, the amazing computing capility of GPU comes from multiple thread. But we couldn't just use thread this only one concept to program since that it is difficult to describe the job or we can say organize them.

So, CUDA introduce the concept of Grid and Block which are logic concepts, they are created by CUDA for programmers. In fact, we we can see them as a organization structure.

首先给出一个gpu的简易逻辑图，理解gpu，sm，sp，warp scheduler的关系

一个kernel 函数是一个grid，对应整个gpu
然后grid中包含很多block，这个和gpu中的sm对应
block包含很多thread，这个和sm中的sp对应

sm的组成：
1. register
2. shared memory
3. the cache of constant memory
4. the cache of texture and surface memory
5. L1 cache
6. warp scheduler
7. SP（截止到2023/12/09，对sp的理解是它只是一个泛称，不是特指某种具体的运算单元，SP可能对应于不同的硬件组件，包括浮点运算单元（FP）、整数运算单元（INT）、特殊功能单元（SFU））

所以调度问题分为两个层面：
1. 对于block的调度: 这个和sm有关系，多个block和多个sm，sm可以任意选择block执行，而且这种选择并不是一选定终生，中间过程还可以调整，但是block并不能拆分，只能整个放到sm上
2. 对于thread的调度: 把一个block放到一个sm上，这时候我们的视角就要缩小到这个sm中了，这时候我们不需要考虑block层面了，只是要考虑block中这一大堆thread如何调度。这时候warp的概念就出现了，把一大堆thread范围为一些warp。然后由warp scheduler调度这一个warp，更准确的说法是由warp scheduler调度warp中的thread，所以warp只是一个中间概念，最终调度的仍然是thread(明确这一点对于理解后文的[bank conflict](https://gaohongy.github.io/blog/posts/%E5%B9%B6%E8%A1%8C%E8%AE%A1%E7%AE%97/gpu-structure-and-programing/#bank-conflict)至关重要)


According to the real physical level, a SM(Streaming Multiprocessor) has many SP(Streaming Processor). Considure how can a 
 every SM in GPU will excute multiple blocks, and it will divides block into multiple warps. The basic execution unit of SM is warp. 

> Some official concepts about warp:

1. A block assigned to an SM is further divided into 32 thread units called warps.
2. The warp is the unit of thread scheduling in SMs.
3. Each warp consists of 32 threads of consecutive threadIdx values: thread 0 through 31 form the first warp, 32 through 63 the second warp, and so on.
4. An SM is designed to execute all threads in a warp following the Single Instruction, Multiple Data (SIMD) model

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202309241749537.png)

## Memory structure

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202311151322227.png)
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202311081852318.png)

**How to detect the using situation of the different types of memory?**

1. Use nvcc compilation option `--ptxas-option=-v`
   `--ptxas-option` is used to specify options directly to ptxas(the PTX optimizing assembler, its location in the whole compilation process can be seen at [CUDA Compilation](https://www.cnblogs.com/hongyugao/p/17445574.html#CUDA%20Compilation:~:text=occupancy(%E5%8D%A0%E7%94%A8%E7%8E%87)%E3%80%82-,CUDA%20Compilation,-%E6%B6%89%E5%8F%8A%E5%88%B0%E4%B8%A4))
2. Use nvcc compilation option `-keep`
3. Use `nvprof` command `nvprof --print-gpu-trace <program path>`

### Cache
There are 5 types of cache in cuda:
1. shared memory(equivalent to a user-managed chache)
2. L1 cache
3. L3 cache
4. constant cache
5. texture cache

We will talk more details about shared memory later, so we only focus on the remaining four types of real cache.

#### L2 cache
Just like L1 cache and shared memory, the L2 cache and global memory is related.

L2 cache is used to provide higher bandwidth and lower latency accesses to global memory.

We can refer to [Query L2 cache Properties](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#query-l2-cache-properties) to get the properties of L2 cache.

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202311242150971.png)


### Register & Local Memory

### Global Memory
The path of accessing global memory: L1 cache -> L2 cache -> global memory

#### coalesced & uncoalesced
coalesced memory access <=> a global memory access request from **a warp** will cause to 100% degree of coalescing.
> The above conclusion has two important things we need to pay attention to:
> 1. The object we talk about is a warp
> 2. The definition of degree of coalescing is described as the following  formula

$Degree \ of \ coalescing \ (合并度) = \frac{ request \ bytes \ number \ (warp实际请求数据量) }{ bytes \ number \ that \ participate \ in \ the \ data \ transforming \ (实际输出的数据量)}$

看到一句话，提到了DRAM burst，暂时还没有找到官方的解释
> CUDA Coalesced access uses the DRAM’s burst mode

因为coalesced access是基于DRAM的burst mode来实现的，所以本质上会涉及到DRAM burst发生的性质和要求：
- 对齐
- 访存大小

疑惑点其实是在于发生coalesced是否和warp相关，是不是必须是同一个warp内的线程的访问才肯跟造成coalesced access，还是说同一个block不同warp，还是说不同block都可以？如果仅从DRAM burst发生的角度考虑，burst发生的条件应该是和CUDA的一些概念无关的，所以视角似乎可以直接放到不同线程上，并不需要考虑是否是同一warp或者是否是同一block

想要理解memory coalesced和uncoalesced，思维必须从串行思维转换到并行思维，

> Coalesce happens amongst threads, not amongst different iterations of the loop within each thread’s execution.

关注的重点不应放在一个单独的thread上，我觉得一个比较合适的视角是放在一个warp上（关于这一点，有一个很明显的错误示范，就是矩阵乘法P=MxN，如果从单一thread的角度来看，对M的访问应当是满足coalesced地，但是如果考虑属于一个warp的不同thread，就会发现实际上对N的访问才是coalesced。考虑某一时刻属于同一个warp的thread的访存方式。关于这一示例的详细分析，可见[The CUDA Parallel Programming Model - 5. Memory Coalescing](https://nichijou.co/cuda5-coalesce/)

#### Common memory access types
Please note that the third and the last code can't get the right answer. The following code is just to used to describe types of memory access type.

1. Sequential **coalesced** access(顺序的合并访问)
```cuda
__global__ void add_sequential_coalesced(float *x, float *y, float *z) {
    int threadId = threadIdx.x + blockDim.x * blockIdx.x;

    z[threadId] = x[threadId] + y[threadId];
}
```

2. Out-of-order **coalesced** access(乱序的合并访问)
```cuda
__global__ void add_out_of_order_coalesced(float *x, float *y, float *z) {
    int threadId = threadIdx.x ^ 0x1;
    threadId = threadIdx.x + blockDim.x * blockIdx.x;
    
    z[threadId] = x[threadId] + y[threadId];
}
```

3. Misaligned **uncoalesced** access(不对齐的非合并访问)
```cuda
__global__ void add_misaligned_uncoalesced(float *x, float *y, float *z) {
    int threadId = threadIdx.x + blockDim.x * blockIdx.x + 1;

    z[threadId] = x[threadId] + y[threadId];
}
```

4. Strided **uncoalesced** access(跨越式的非合并访问)
```cuda
__global__ void add_stripped_uncoalesced(float *x, float *y, float *z) {
    int threadId = blockIdx.x + threadIdx.x * blockDim.x;

    z[threadId] = x[threadId] + y[threadId];
}
```

> Please Note that this is different from [grid stride loop](https://gaohongy.github.io/blog/posts/%E5%B9%B6%E8%A1%8C%E8%AE%A1%E7%AE%97/gpu-structure-and-programing/#grid-stride-loop), which emphasizes that how to solve the big problem which the scale is bigger than the amount of threads. But there we want to emphasize a type of memory access.

5. Broadcast **uncoalesced** access(广播式的非合并访问)
```cuda
__global__ void add_broadcast_uncoalesced(float *x, float *y, float *z) {
    int threadId = threadIdx.x + blockDim.x * blockIdx.x;

    z[threadId] = x[0] + y[threadId];
}
```

broatcast这种方式还涉及到constant memory的使用

其实global memory就类似dram，l2 cache也就是个cache，所以thread访问global memory的过程和体系结构里面对于cache的分析过程是完全一样的，thread请求一个字节的数据，发现cache中不存在，即发生cache miss，然后就去访存，并且把数据缓存到cache line中，访问同一cache line对应数据的thread的再访存就是cache hit了，所说的这个合并访问似乎不过是访问这个cache line的过程，只要是一次访存对应几次都是cache hit就算是合并访存了，似乎完全可以这样理解.
其实这块判断是否会发生合并的一个前提就是确定从global memory一次到底取多少数据，现有认知是按照字节编制，但是按照字进行读取，但是书上却说一次读取32Bytes，一个字总不能有32Bytes吧。

### Shared Memory
共享内存中的内存块通常被直接称为 memory tile 或简称为 tile。（可能这就是Tiled Matrix Multiplication的由来）

#### Create Shared Memory

1. 静态shared memory，使用`__shared__`限定符创建时就指定存储空间的大小

```
__shared__ float array[1024];
```

2. 动态shared memory，不确定空间大小，需要动态申请时

```
extern __shared__ float array[1024];
```

需要在kernel函数调用时，指定申请的shared memory的大小

```
kernel<<<gridSize, blockSize, sizeof(float) * 1024>>>( … );
```

在C/C++中，存在一个变长数组（Variable Length Arrays，VLA）的概念，允许使用变量来指定数组的大小。
但是实际测试，变量指定数组大小应用于kernel函数时，会报错"error: expression must have a constant value"



#### Bank Conflict
To understand this problem well, we should revisiv the [hardware structure of gpu](https://gaohongy.github.io/blog/posts/%E5%B9%B6%E8%A1%8C%E8%AE%A1%E7%AE%97/gpu-structure-and-programing/#hardware-structure).

在此基础上，我们将gpu的建议结构图进行扩充，装入shared memory和bank的结构
还需要一张shared memory中是如何划分的bank

bank的划分单位和最大bandwith都是32bits=4bytes=1word
但是寻址单位还是1byte

哪些情况下会产生bank conflict, 首先看一下都有哪些可能的bank访问情况
1. 同一warp：
	1.1 两个thread访问同一个bank中相同的字中的地址 (broadcast, conflict-free)
	1.2 两个thread访问同一个bank中不同的字中的地址 (conflict)
	1.3 两个thread访问不同bank (conflict-free)
2. 不同warp：
	2.1 两个thread访问同一个bank相同的字中的地址 (conflict)
	2.2 两个thread访问同一个bank不同的字中的地址 (conflict)
	1.3 两个thread访问不同bank (conflict-free)

要理解bank conflict，需要首先了解bank是怎么回事，
> To achieve high bandwidth, shared memory is divided into equally-sized memory modules, called banks, which can be **accessed simultaneously**.

尤其是后面这句话比较重要，不同bank可以同时响应数据请求（实现这一点应该是需要硬件支持的，每一个bank是一个独立的存储单元）

所以就可以理解为什么不同thread访问同一个bank的时会降低效率，因为本来可以同时读，现在只能串行读(关于这一点还有以下疑点：bank coflict 只发生在不同warp中的thread在访问同一个bank的不同byte时，同一个warp内的thread无论如何访问都不会产生bank conflict)

这样来看，bank本身和ram的性质类似，但是整个shared_memory可以看为是多个ram拼接而成

According to the [real hardware architecture of SM](https://gaohongy.github.io/blog/posts/%E5%B9%B6%E8%A1%8C%E8%AE%A1%E7%AE%97/gpu-structure-and-programing/#hardware-structure), SM has multiple **warp schedulers**.

A block will be distributed to a SM, but the unit of execution of SM is warp which has 32 threads. 

> It is easy to understand the principle of this setting, as we all know a block has many threads, if SM dispatch all of them at the same time, it will casuce difficulties. So the designer divide the block into warp.

All warps in the same block will share the same shared memory. Shared memory is also divided into many subdivisions. The number of subdivisions equals to the number of warp.
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202311222214856.png)
Warp access shared memory use the bank as the unit.

The most optimal situation is every warp correspondens to a bank. At this situation, the time of accessing whole 32 banks is just 1 memory cycle.
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202311222218255.png)
> To be precise, it should contains 32 threads and banks in figure. It is just a schematic drawing.

But if many bank access the same bank, it will cause the following situation. At this situation, the time of accessing whole 32 banks is 32 memory cycles.
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202311222219570.png)
> To be precise, it should contains 32 threads and banks in figure. It is just a schematic drawing.


**An correlative calculation of this problem**
The hardware splits a memory request with bank conflicts into as many separate conflict-free requests as necessary, decreasing throughput by a factor equal to the number of separate memory requests. 

If the number of separate memory requests is n, the initial memory request is said to cause n-way bank conflicts.

To get maximum performance, it is therefore important to understand how memory addresses map to memory banks in order to schedule the memory requests so as to minimize bank conflicts.

**How can we solve this problem ?**

We can pad and adjust the memory structure as the following picture shows.
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202311222225417.png)

> Reference
> - [1] [hardware-effects-gpu-Bank conflicts](https://github.com/Kobzol/hardware-effects-gpu/blob/master/bank-conflicts/README.md)
> - [2] [Nsight Compute CLI - Metric Comparison](https://docs.nvidia.com/nsight-compute/NsightComputeCli/index.html#nvprof-metric-comparison)

### Constant Memory
A simple use of constant memory comes from [convolution](https://gaohongy.github.io/blog/posts/%E5%B9%B6%E8%A1%8C%E8%AE%A1%E7%AE%97/gpu-structure-and-programing/#convolution).

In convolution, because there are four aspects which leads to that we can use constant memory.

1. The ratio of floating-point arithmetic calculation to global memory accesses is so low.(计算访存比较低，简单理解就是读了很多数据但是计算的比较少，事倍功半)
2. The size of mask is small. (The constant memory size is small)
3. The constants of mask are not changed throughout the execution of the kernel. (The constant memory is prohibited modification)
4. All threads need to access the mask elements. (store memory into cache is effective)

According to the [picture](https://gaohongy.github.io/blog/posts/%E5%B9%B6%E8%A1%8C%E8%AE%A1%E7%AE%97/gpu-structure-and-programing/#memory-structure) at the beginning of the Memory structure. We can learn about the constant memory is in DRAM.

But because the variable or space in constant is prohibited modification, so cuda runtime can put its content to cache trustfully, at the same time, no modification means that there is no cache coherence issue.

There are three important aspects of using constant memory:
1. `__constant__ float M[];`, use the `__constant__` specifier and M should be a global variable
2. `cudaMemcpyToSymbol(M, M_h, Mask_Width*sizeof(float));`


### Host Side Memory

#### pageable memory
> 可分页内存

- 使用`malloc()/new()`和`free()/delete()`函数分配和释放
- 此类型内存是可以从内存被换出到磁盘的

#### pinned memory 
> [pinned memory](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#page-locked-host-memory), aka non-pageable memory(不可分页内存) / page-locked(页锁定内存)

- 使用[`cudaHostAlloc()`](https://docs.nvidia.com/cuda/cuda-runtime-api/group__CUDART__MEMORY.html#group__CUDART__MEMORY_1gb65da58f444e7230d3322b6126bb4902) / [`cudaMallocHost()`](https://docs.nvidia.com/cuda/cuda-runtime-api/group__CUDART__MEMORY.html#group__CUDART__MEMORY_1gab84100ae1fa1b12eaca660207ef585b)和`cudaFreeHost()`函数分配和释放
> `cudaHostAlloc()`和`cudaMallocHost()`的关系是 `cudaHostAlloc(xxx, yyy, cudaHostAllocDefault)` 等价于 `cudaMallocHost(xxx, yyy)`

- 此类型内存一直停留在内存，不会被换出到磁盘
- 此类型内存支持DMA访问，支持与GPU之间进行异步通信（asynchronous data transfer）


**Some background on the memory management in operating systems**

- `cudaMemcpy()` uses the hardware **direct memoryory access (DMA) device**.
- The operating system give a translated physical address to DMA, i.e. the DMA hardware operates on physical addresses.
- Uses the DMA to implement the `cudaMemcpy()` faces a chance that the data in the pageable memroy can be overwritten by the paging activity before the DMA transmission.

The solution is to perform the copy operation in two steps:

1. For a host-to-device copy, the CUDA runtime first copies the source host memory data into a **pinned memory buffer**, sometimes also referred to as **page locked memory buffer**.
2. It then uses the DMA device to copy the data from the pinned memory buffer to the device memory.

The problems of this solution:

1. Extra copy adds delay to the cudaMemcpy() operation.
2. Extra complexity involved leads to a synchronous implementation of the cudaMemcpy() function.
> About the synchronous and asynchronous, please see the [API synchronization behavior](https://docs.nvidia.com/cuda/cuda-runtime-api/api-sync-behavior.html#api-sync-behavior__memcpy-sync)

To solve this problem, we can use the `cudaHostAlloc()` to open up pinned memory buffer, and use the `cudaMemcpyAsync()` to copy a data asynchronously.

#### Zero-Copy Memory
首先，零拷贝内存并不是像unified memory这样的逻辑存在，其是一种物理存在。其特别之处在于实际的物理存储空间实际是 Host Memory，但是 device 却可以通过某种方式直接访问，无需人工进行拷贝操作。

**The way of opening up zero-copy memory**

The zero-copy memory is a special host memory, it is a pinned memory.

1. When we use `cudaHostAlloc()` to open up a pinned memory, we need to transmit the third parameter `flag`. We need to transmit `cudaHostAllocMapped` as a flag to cudaHostAlloc().
2. **Host code** use [`cudaHostGetDevicePointer()`](https://docs.nvidia.com/cuda/cuda-runtime-api/group__CUDART__MEMORY.html#group__CUDART__MEMORY_1gc00502b44e5f1bdc0b424487ebb08db0) to get a pointer which points to the pinned memroy.
> Please note that we should get the zero-copy memory pointer in host code rather than device cost, although it is more reasonable to get this pointer in device code.

3. At this time, the pinned memory is called zero-copy memory.

```c++
#include <iostream>
#include <cuda_runtime.h>

__host__ __device__ void output(int *a) {
    for (int i = 0; i < 10; i++)
        printf("%d", a[i]);
    printf("\n");
}

__global__ void kernel(int *d_a) {
    output(d_a);
    d_a[2] = 5;
}

int main() {
    int *h_a, *d_a;

    cudaHostAlloc((void **)&h_a, 10 * sizeof(int), cudaHostAllocMapped);
    cudaHostGetDevicePointer((void **)&d_a, h_a, 0);

    for (int i = 0; i < 10; i++) h_a[i] = 1;

    kernel<<<1, 1>>>(d_a);
    cudaDeviceSynchronize();

    output(h_a);

    return 0;
}

// output:
// 1111111111
// 1151111111
```

### Unified Memory
Unified Memory是一种逻辑上的存在，它提供了一种抽象层，让程序员可以将主机（CPU）和设备（GPU）上的内存视为一个统一的内存空间。

使用Unified Memory的情况下，程序员无需显式地管理数据的迁移，系统会根据需要自动处理。

Unified Memory通过使用页表和硬件支持，实现了逻辑上的一致性。

Unified Memory并不是物理上的一块内存，而是一个逻辑概念，通过系统的管理和硬件支持，实现了对主机和设备上内存的透明管理。这有助于简化GPU编程中的内存管理任务。

#### Optimization Techniques
[Unified Memory for CUDA Beginners](https://developer.nvidia.com/blog/unified-memory-cuda-beginners/)

## Memory API

CUDA C提供了与C语言在语言级别上的集成，主机代码和设备代码由不同的编译器负责编译，设备函数调用样式上接近主机函数调用

`cudaMemcpy()` will synchronize automatically, so if the last line code is `cudaMemcpy()`, we needn't to use the `cudaDeviceSynchronize()`

**Different devices corresponding to different memory functions**


| Location       | memory allocate   | memory release |
| -------------- | ----------------- | -------------- |
| Host           | malloc/new        | free/delete    |
| Device         | cudaMalloc        | cudaFree       |
| Unified Memory | cudaMallocManaged | cudaFree       |

**Which memory types do we have ?**
Host and device has different authorities to use the memory. The following table describes their authorities.


| Memory type     | Host | Device |
| --------------- | ---- | ------ |
| Global memory   | W/R  | W/R    |
| Constant memory | W/R  | R      |
|                 |      |        |

**Why we need unified memory ?**

1. Additional transfers between host and device memory increase the latency and reduce the throughput.
2. Device memory is small compared with the host memory. Allocating the large data from host memory to device memory is difficult.

> Annotate: W means Write and R means Read

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202309271517094.png)

### 2D Array
Refering to the methos of opening up a 2D space in host code which see the first dimension of array is some pointers and the second dimension of array is a 1D array. When we want to allocate a 2D array in device memory, the above method is difficult, because we need to access a space of device memory in host code.

So, CUDA provide **pitched memory** to implement it. We can use `cudaMallocPitch()` to create a 2D space in device memory, `cudaMemset2D()` to copy data between host and device and `cudaFree()` to release space.

## Software structure

> All CUDA threads in a grid execute the same kernel function;

It is easy to explain it. When we want to call a kernel function, we will specify the grid and block structure using the `dim3` data type. It means that we want to use all these threads where locate in the grid to execute this kernel function.

In general, a grid is a three-dimensional array of blocks1, and each block is a three dimensional array of threads.
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202310122206388.png)

From a code implementation perspective, these two three-dimensional arrays are both a `dim3` type parameter, which is a C struct with three unsigned integer fields: x, y, and z.
The first execution configuration parameter specifies the dimensions of the grid in the number of blocks. And the second specifies the dimensions of each block in the number of threads.
For example, as the following code shows, there is a grid and a block. The grid consists of 32 blocks, and it is a linear structure. The block consists of 128 threads, and it is also a linear structure.

```
dim3 dimGrid(32, 1, 1);
dim3 dimBlock(128,  1, 1);
vecAddKernel<<<dimGrid, dimBlock>>>(...);
```

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202310122217866.png)

> About the more detail specifications please see [official technical specifications](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#features-and-technical-specifications)



## Software stack

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202311251423321.png)

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202309241804271.png)

## Kernel Function

> Because the execution of the kernel function is asynchronous, that means the subsequent codes don't know when the result will be returned by kernel funcion, so the type of returen value of kernel funciont must be void.

1. CPU以及系统内存成为主机，GPU及其内存成为设备
2. GPU设备上执行的函数称为核函数（Kernel）
3. 核函数调用时<<<para1，para2>>>中的para1表示设备在执行核函数时使用的并行线程块的数量，通俗来说总共将创建para1个核函数来运行代码，共para1个并行执行环境，para1个线程块。这para1个线程块称为一个线程格（Grid）
4. 核函数中存在一个CUDA运行时已经预先定义的内置变量blockIdx，表示当前执行设备代码的线程块索引

The difficulty of writing parallel programs comes from arranging the structure of grid、block and thread so that they can adapt the programs.
What we ought to know is that the kernel funtion is just like a big loop in logic, it will enumerate the whole grid in threads.

> Note: This perspective is just from code, it is not the true execution logic.

## 指针

主机指针只能访问主机代码中的内存，设备指针只能访问设备代码中的内存

### 设备指针

虽然`cudaMalloc()`同`malloc()`，`cudaFree()`同`free()`非常相似，但是设备指针同主机指针之间并不完全相同，设备指针的使用规则如下

1. `cudaMalloc()`分配的指针可以传递给设备函数，设备代码可以使用该指针进行内存读/写操作（解引用）
2. `cudaMalloc()`分配的指针可以传递给主机函数，主机代码不可以使用该指针进行内存读/写操作（解引用）

### 主机指针与设备指针数据拷贝

1. 主机->主机：`memcpy()`
2. 主机->设备：`cudaMemcpy()`指定参数`cudaMemcpyHostToDevice`
3. 设备->主机：`cudaMemcpy()`指定参数`cudaMemcpyDeviceToHost`
4. 设备->设备：`cudaMemcpy()`指定参数`cudaMemcpyDeviceToDevice`

The communication between CPU and GPU is asynchronous for high performance. So need to use the synchronous mechnisms for them.

## Function type

- `__host__`
- `__global__`
- `__device__`

|  function type \  action     |   Callable from   |   Executed on   |
| ---- | ---- | ---- |
|  `__global__`    |   host / divice(compute capability 5.0 or higher)   |   device   |
| `__device__`   |   device   |   device   |
|  `__host__`    |   host   |   host   |

Without any of the `__host__`, `__device__`, or `__global__` specifier is equivalent only the `__host__` specifier.

Some special usage:

- The `__global__` and `__device__` execution space specifiers **cannot** be used together.

- The `__global__` and `__host__` execution space specifiers **cannot** be used together.

- The `__device__` and `__host__` execution space specifiers **can** be used together.(This usage is used to decrease the verbose codes, the compiler will compile the function for host and device seperately)

More informations please see the [official station](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#function-execution-space-specifiers).

## Common Parallelization methods
### Grid-stride loop
This method is used to solve the problem, the parallelism(并行度) is more than the quantity of threads.

In some situations, we can create many threads so that satisfied the parallelism, that we can allocate a separate thread for every threads.

But if the parallelism is more than the quantity of threads and we still use the above strategy, we will get the following result.

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202309241915666.png)

> The parallelism is 32, but we just have 8 threads, we can't allocate a separate thread for every threads.

Grid-stride loop provide a new approach to solve this problem. At first, we studt the content of this method and we will think the core principle of this method.
The process of grid-stride loop looks like the following figure.

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202309241933388.png)

In short, the core approach to implement it is `for (size_t i = threadIdx.x; i < n; i += <total number of threads>`. When the number of threads is smaller than parallelism, we can't use the traditional method to implement the parallel, simply speaking, the distribution of thread can't satisfied the parallelism.

Grid-stride loop uses <total number of threads> to map the threads to tasks reasonably, the <total number of threads> is a magic number, it can ensure different thread will not intersect with each other and finish all tasks.

### Fixed location solve the data conflict
The most obvious answer is using mutex or atomic operation. But as we all know, whether it's mutex or atomic, they both have some consuming.

We know that the data conflict comes from shared data, different thread maybe use the same data at the same time. So an approach to avoid happening this problem is that control different threads use different data.

According to the process of Grid-stride loop, we notice that different threads use the different datas which have different locations. We can specify a fixed location to store a thread's data to avoid using the mutex or atomic.

A good example is array summation. As the following code shows.

```cuda
#include <iostream>
#include <vector>
#include <cuda_runtime.h>
#include "CudaAllocator.h"
#include "ticktock.h"
#include <stdio.h>

__global__ void parallel_sum(int *arr, int *sum, int n) {
	for (int i = blockDim.x * blockIdx.x + threadIdx.x;
		 i < n / 4;
		 i += gridDim.x * blockDim.x)
	{
		for (int j = 0; i + j < n; j += gridDim.x * blockDim.x) {
			sum[i] += arr[i + j];
		}
	}
}

int main() {
	int n = 1 << 4;

	// unified memory
	std::vector<int, CudaAllocator<int>> arr(n);
	std::vector<int, CudaAllocator<int>> sum(n / 4);

	for (size_t i = 0; i < n; i++) {
		arr[i] = i;
	}

	// 设置共n/4个thread，每个block为4个thread，因此block数量为n / 4 / 4
	dim3 blockSize(4);
	dim3 gridSize(n / 4 / 4);

	parallel_sum<<<gridSize, blockSize>>> (arr.data(), sum.data(), n);
	cudaDeviceSynchronize();

	int final_sum{0};
	for (int i = 0; i < n / 4; i++)
		final_sum += sum[i];

	std::cout << "sum = " << final_sum << std::endl;

	return 0;
}
```

## Synchronization

CPU programing needs synchronous mechanism, GPU programing also needs it.

### Atomic

We can learn about the execution logic by refering to [C++ atomic](https://www.cnblogs.com/hongyugao/p/17692121.html#atomic) and details of function by refering to [CUDA C++ Programming Guide](https://docs.nvidia.com/cuda/pdf/CUDA_C_Programming_Guide.pdf).

## Asynchronization (CUDA stream)
Serialize data transfer and GPU computation causes that PCIe idle and GPU idel appear interleavly.

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202312081112953.png)

Most CUDA devices support device overlap. Because data transmission is more timeless than computation, so simultaneously execute a kernel while performing a copy between device and host memory can cover up the time consumation of data transmission.

CUDA supports parallel execution of kernels and cudaMemcpy with streams. A stream is a sequence of commands that execute in order.

The commands issued on a stream may execute when all the dependencies of the command are met. The dependencies could be previously launched commands on same stream or dependencies from other streams.

About the PCIe transmission rate is shown as the following picture:
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202312081517982.png)

流的分类：
1. 根据发出方分类：
- Host端发出的流(主要讨论的是这种)
- Device端发出的流

2. 根据有无内容分类：
- 默认流(default stream) / 空流(null stream)
- 明确指定的非空流

### Execution order
同一个CUDA流中的操作是串行顺次执行的，不同stream中的operation随机执行，可能是并发交错执行的

### Related API
1. `__host__ ​cudaError_t cudaStreamCreate ( cudaStream_t* pStream )`
> Create an asynchronous stream. 

2. `__host__ ​__device__ ​cudaError_t cudaStreamDestroy ( cudaStream_t stream )`
> Destroys and cleans up an asynchronous stream.

3. `__host__ ​cudaError_t cudaStreamQuery ( cudaStream_t stream )`
> Queries an asynchronous stream for completion status.

4. `__host__​ cudaError_t cudaStreamSynchronize ( cudaStream_t stream )`
> Waits for stream tasks to complete.

### 如何理解流
从主机和设备两个视角的动作来分析

使用`cudaMemcpyAsync()`时，Host memory必须是[non-pageable memroy / pinned memory](https://gaohongy.github.io/blog/posts/%E5%B9%B6%E8%A1%8C%E8%AE%A1%E7%AE%97/gpu-structure-and-programing/#host-side-memory), 数据传输过程由GPU的DMA负责

如果是[pageable memroy](https://gaohongy.github.io/blog/posts/%E5%B9%B6%E8%A1%8C%E8%AE%A1%E7%AE%97/gpu-structure-and-programing/#host-side-memory)使用`cudaMemcpyAsync()`, 需要首先将pageable memory移动到pinned memory，这个过程中就会涉及到数据同步。

还有一个需要注意的事情就是PCIe，同一时刻H2D和D2H都只能进行1个操作。

## C++ Encapsulation

As we all know, the style of many CUDA APIs is C-style, we need to learn about how to use it conjunction with C++.

**How does the std::vector standard template library use the Device(GPU) memory ?**
Many examples use the original pointer to point a Device memory. But if we want to use a std::vector or other standard template library that locates in Device memory, we can't use the `cudaMalloc()` or `cudaMallocManaged()`.

Taking the `std::vector` as an example, next, we will discuss the method of allocating Device memory for containers.

> Whether it's principle or usage methods is too complex to understand in a short time. So pause it for a period of time. When we must need to learn its principle we study it again. We can learn about it from [一篇文章搞懂STL中的空间配置器allocator](https://www.coonote.com/cplusplus-note/space-allocator.html). In short, std::allocator integrates the memory management and object management by using four member function.

## GPU execution core

一个kernel函数在逻辑上以block为单位映射到SM中，物理上以warp为单位解析指令将指令分发到具体的运算单元(SP/core, SFU, DP)或访存单元(LD/ST)。
SM中活动的warp数量占物理warp数量的比率为occupancy(占用率)。

## Device API
1. `__host__ ​__device__ ​cudaError_t 	cudaGetDeviceCount ( int* count )`
> Returns the number of compute-capable devices.

2. `__host__ ​cudaError_t cudaGetDeviceProperties ( cudaDeviceProp* prop, int  device )`
> Returns information about the compute-device.

3. `__host__​ cudaError_t cudaSetDevice ( int  device )`
> Set device to be used for GPU executions.

More information please see the [official website](https://docs.nvidia.com/cuda/cuda-runtime-api/group__CUDART__DEVICE.html#group__CUDART__DEVICE).

Three are three ways to transfer data from one device to another:
1. `cudaMemcpyPeerAsync()`
2. `cudaMemcpy()`: rely on the unified address system
3. Implicit peer memory access performed by the driver

## CUDA Libraries
### cuBLAS

### cuSOLVER

### cuFFT

### Thrust
1. Three main functionalities
- The host and device vector containers
- A collection of parallel primitives such as, sort, reduce and transformations
- Fancy iterators

## CUDA Compilation

涉及到两部分内容，一部分是cuda面对编译问题时的设计架构，另一方面是cuda实际的编译流程

首先对CUDA程序的编译流程进行简要介绍，下图是[NVIDIA CUDA Compiler Driver NVCC - The CUDA Compilation Trajectory](https://docs.nvidia.com/cuda/cuda-compiler-driver-nvcc/index.html#the-cuda-compilation-trajectory)中给出的cuda编译流程。

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202311141110954.png)

上图可以结合实际流程和用到的指令来理解，这些可以通过`nvcc -dryrun <cuda program name>`来获取到

> `-dryrun`: List the compilation sub-commands without executing them.

生成的中间文件，可以通过`nvcc -keep`来获取到

> `-keep`: Keep all intermediate files that are generated during internal compilation steps.

下面结合编译流程主要说明以下几个问题：

1. 何为PTX(Parallel Thread Execution) ，为何会设计它
2. 何为SASS，为何会设计它

因为硬件在发展过程中，设计和架构可能会发生很大的改变，为了避免在硬件更新时软件发生较大的改变，一种常用的设计策略是**抽象**。即把真实的物理架构抽象为逻辑架构，开发者仅需要关注逻辑架构，从逻辑到物理的映射由框架开发商完成。

CUDA处理这个问题时采用的也是这种策略。其将结构分为两种：

1. 虚拟GPU结构（Virtual Architecture）
2. 真实GPU结构（Real Architecture)

PTX实际就是Virtual Architecture的汇编产物，它是一种指令集，由于考虑的只是逻辑架构，因此它可以在不同物理架构的GPU上使用。而SASS则是对应的Real Architecture，它是实际运行在物理设备上的指令集。在实际编译过程中，它们分别对应着生成.ptx和.cubin两个文件的过程，简图如下所示。

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202311141126447.png)

同时在编译时，也可以通过选项来指定不同的Virtual Architecture和Real Architecture。`-arch=compute_52`（`-arch` = `--gpu-architecture`）是指对虚拟GPU体系结构进行配置，生成相应的ptx。 `-code=sm_52`（`-code` = `--gpu-code`）是对实际结构进行配置。要求Virtual Architecture的版本要低于Real Architecture的版本，这一点是不难理解的。

```
cicc -arch=compute_52 "sample.cpp1.ii" -o "sample.ptx"
ptxas -arch=sm_52 "sample.ptx" -o "sample.sm_52.cubin" 
```

> 可以发现上述示例命令中的选项和描述并不对应，第2条指令使用-arch但是却指定了一个Real Architecture的版本。
> 当省略-code选项时，-arch选项指定的可以是Real Architecture的版本，此时由nvcc自行确定一个Virtual Architecture的合适版本
> 详见官方文档[--gpu-architecture (-arch)](https://docs.nvidia.com/cuda/cuda-compiler-driver-nvcc/index.html#gpu-architecture-arch)和[--gpu-code code (-code)](https://docs.nvidia.com/cuda/cuda-compiler-driver-nvcc/index.html#gpu-code-code-code)

### Reference

> - [1] [NVCC与PTX](https://zhuanlan.zhihu.com/p/432674688)

## GPGPU-Sim

> [Official site](http://gpgpu-sim.org/)

### How to run

1. Use the command `ldd` to make sure the application's executable file is dynamically linked to CUDA runtime library
2. Copy the contents of configs/QuadroFX5800/ or configs/GTX480/ to your application's working directory.

> These files configure the microarchitecture models to resemble the respective GPGPU architectures.

3. Run a CUDA application on the simulator

> source setup_environment <build_type>

### Source code organization structure

Gpgpu-sim的源码位于`gpgpu-sim_distribution/src/gpgpu-sim`。
目前，我们主要关注其中和配置相关的内容，我们通过修改gpgpu-simi的源码（增加一个配置项），重新编译并用其执行程序来简单理解gpgpu-sim对于配置项的设置方式。

1. 修改`gpu-sim.cc:gpgpu_sim_config::reg_options()`，在其中添加一个配置项

```
option_parser_register(opp, "-magic_number", OPT_INT32, &magic_number_opt, "A dummy magic number", "0");
```

2. 修改`gpu-sim.h`，在配置项对应结构体中添加对应字段

```
int magic_number_opt;
```

3. 重新编译gpgpu-sim项目
4. 将编译后生成的`gpgpusim.config`拷贝到待执行cuda程序路径下
5. 修改待执行cuda程序路径下的`gpgpusim.config`配置文件，添加配置项

```
-magic_number 25
```

6. 执行cuda程序，在输出信息中就可以看到新增的配置项

```
-magic_number                          25 # A dummy magic number
```

### Reference
> - [1] [Working with GPGPU-Sim - Introduction](https://coffeebeforearch.github.io/2020/03/30/gpgpu-sim-1.html)
> - [2] [Working with GPGPU-Sim - Adding Configuration Options](https://coffeebeforearch.github.io/2020/04/13/gpgpu-sim-2.html)
> - [3] [Improving GPGPU-Sim Performance](https://coffeebeforearch.github.io/2020/03/31/perf-gpgpu-sim.html)
> - [4] [ECE 695 GPGPU-Sim Tutorial 学习笔记](https://zhuanlan.zhihu.com/p/576442425)
> - [5] [GPGPU-SIM系列文章 | 科学网](https://blog.sciencenet.cn/home.php?mod=space&uid=1067211&do=blog&view=me)

附加内容：

1. If want to use ptxplus (native ISA) change the following options in the configuration file

> -gpgpu_ptx_use_cuobjdump 1
> -gpgpu_ptx_convert_to_ptxplus 1

2. If want to use GPUWatch change the following options in the configuration file

> -power_simulation_enabled 1 (1=Enabled, 0=Not enabled)
> -gpuwattch_xml_file <filename>.xml

## Related Programming Models
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202312131446434.png)

1. OpenCL
> Open Computing Language

2. OpenACC
> Open Accelerators

3. OpenMP
> Open Multi-Processing

Reference to the memroy modle of OpenMP, we can get the following information: "The OpenMP API provides a relaxed-consistency, shared-memory model."

   threaded parallelism

虽然OpenMP只能用于单机，但是可以处理单机上的多卡

4. MPI
> Message Passing Interface

MPI可以理解为是一种独立于语言的信息传递标准, 本身和代码没有关系，可以看为是一种规定。
OpenMPI和MPICH等编程模型是对这种标准的具体实现。也就是说，OpenMPI和MPICH这类库是具体用代码实现了MPI标准。因此我们需要安装OpenMPI或者MPICH去实现我们所学的MPI的信息传递标准。

   process parallelism


![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202311221659870.png)

## 如何使用CUDA加速程序

目前理解到的CUDA加速程序的两个关键问题是：

1. 任务并行化
   寻找到任务中可以并行完成的部分，制定某种策略将任务合理分配到每个线程中。此过程期望解决的是计算瓶颈(cpu-bound)问题。

1.1 udacity的视频主要讲解的就是这部分
1.2  小彭课程第6讲也是这部分
主要就是讲解一些并行原语

2. 访存优化
   此过程期望解决的是内存瓶颈(memory-bound)问题。

2.1 gpu的存储模型（《大众高性能》）
2.2 小彭课程第7讲

### 对任务划分的理解
想要实现并行化，很重要的一点是考虑“如何合理地将任务划分到不同的thread上”

如何选择grid和block的规模，除了考虑以上合理的任务划分之外，还可以从性能的角度进行考量。
如果仅从下图内容来看，block规模的确定要更为重要，grid的规模只需要根据任务划分和block规模来确定即可

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202312092139368.png)


### 矩阵乘法
1. SGEMM
> Single-precision General Matrix Multiply
单精度通用矩阵乘法

2. DGEMM
> Double-precision General Matrix Multiply 
双精度通用矩阵乘法

3. CGEMM
> Complex-single-precision General Matrix Multiply
复数单精度通用矩阵乘法

4. ZGEMM
> Complex-double-precision General Matrix Multiply
复数双精度矩阵乘法

目前感觉从具体的算子入门CUDA编程中的各种概念、并行算法、访存优化的手段是个非常好的方式，因为各种算法，访存优化一定都是基于实际的应用场景而出现的，都不是仅仅的概念本身

把thread都放置在同一个block中的缺点在于，SM无法对block进行调度，原本的两层调度：block调度和warp调度，现在就只剩下一个warp调度了

native implementation 的核心问题是：计算访存比过低，即使将global memory替换为shared memory，访存时间占比仍然远大于计算时间占比。所以才会考虑矩阵分块

#### Tiled Matrix Multiplication
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/tiled-matrix-multiplication.gif)

tiled matrix multiplication之所以减小了对内存带宽的要求，是因为一个thread读取的内容是可以被其他thread共享的。在分tile之前，一个thread从global memory读取的数据只会让它自己使用，但是分了tile之，一个thread从global memory加载的数据也可以被处于同一个tile中的其他thread所访问，增加了数据重用率。

More information please see the original passage [Tiled Matrix Multiplication](https://penny-xu.github.io/blog/tiled-matrix-multiplication).

#### Reference
> - [1] [Intel-maxas | SGEMM](https://github.com/NervanaSystems/maxas/wiki/SGEMM)

### Convolution
卷积这块有一个神奇的题目和神奇的公式：

问: kD convolution 过程中（不考虑ghost elements的运算），每个元素的平均访问次数

答: 平均访问次数=$\frac{output_{width}^k \times mask_{width}^k}{input_{width}^k}$

其中，在$stride=1$的情况下，满足$output_{width} = input_{width} - mask_{width} + 1$,即$input_{width} = output_{width} + mask_{width} - 1$

2D convolution上述公式的验证代码如下：
```c++
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
	int m, n;
	std::cin >> m >> n;

	int width;
	std::cin >> width;

	auto count = std::vector<std::vector<int>>(m + 10, std::vector<int>(n + 10, 0));

	for (int begin_x = 1; begin_x + width - 1 <= m; begin_x++) {
		for (int begin_y = 1; begin_y + width - 1 <= n; begin_y++) {
			for (int i = 0; i < width; i++) {
				for (int j = 0; j < width; j++) {
					int x = begin_x + i, y = begin_y + j;
					count[x][y]++;
				}
			}
		}
	}

	float sum = 0.0f;
	std::cout << std::endl;
	for (int i = 1; i <= m; i++) {
		for (int j = 1; j <= n; j++) {
			std::cout << count[i][j] << ' ';
			sum += count[i][j];
		}
		std::cout << std::endl;
	}

	std::cout << "\n" << sum << std::endl;
	std::cout << "\n" << (sum / (m * n)) << std::endl;

	return 0;
}
```


#### 1D Convolution
边界处理：
- 判断法
- 扩展法

**背景介绍**
1. 假定每个thread处理一个output element
2. 每一个block要处理的部分称为一个input tile, 生成的部分称为一个output tile

#### 2D Convolution

#### 3D Convolutoin

## GPU Microarchitecture(SIMT Core)
The microarchitecture of the GPU pipeline is divided into a SIMT front-end and a SIMD back-end.

The GPU pipeline consists of three scheduling “loops”: 

1. instruction fetch loop: Fetch, I-Cache, Decode, and I-Buﬀer
2. instruction issue loop: I-Buﬀer, Scoreboard, Issue, and SIMT Stack
3. register access scheduling loop: Operand Collector, ALU, and Memory

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202312171448912.png)
> 注意蓝色和橙色部分存在I-Buffer的交叉

在讲述One/Two/Three-Loop Approximation之前，需要明确的是我们要分析的是SIMT Core的结构，要考虑的问题是如何统筹规划每一个warp所要执行的指令。这一点前提认知很重要，会直接影响到对后面一些结构的理解。

还有一个比较重要的事情，就是弄清楚这里说的one,two,three到底指的是什么，目前理解指的是3种scheduler,分别是SIMT stack, Scoreboard 和 Operand Collector.

### One-Loop Approximation
#### SIMT stack
The SIMT stack is used to solve the thread divergence. It sends the target PC to the fetch unit and the active mask to the issue unit.
> - The fetch unit is used to control which instruction is fetched next.
> - The issue unit is used to control which lanes of the warp are active.

The mask is a bit vector with 1 for every thread that is active for the corresponding control flow branch.  When that control flow branch is being executed, only the threads with 1 in the corresponding branch bit execute those instructions.

A simple method to address the control divergence is PDOM mechanism(post-dominator stack-based reconvergence mechanism). The post-dominator active mask has 1 for every thread that is active in each of the divergent paths that reconverge at that point.

When we hit a divergent point, we push on the stack: 
- (1) the current active mask and the next PC at the reconverge point; 
> 虽然这里说的是current active mask，但是current active mask和 reconverge point 的active mask实际是一样的

- (2) the active mask, PC, and reconverge PC for every branch.  
> 如果有多个branch，入栈的先后顺序一般采取 the entry with the most active threads ﬁrst and then the entry with fewer active threads。我的理解是thread越多越有可能引入新的branch，所以优先让较少thread先执行，避免使局面变得更加混乱。由于栈是FILO，所以the entry with fewer active threads后入栈.不过这只是一般做法，并非强制要求。

For example, look at the following picture, we hit a divergent point A
- (1) the current active mask / reconverge point active mask is 1111, and the reconverge point is G
- (2) the branch of this divergent point A contains B and F

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202312181346451.png)

关于(2)中不同branch入栈的顺序，下图B分支点处采用的是一般的原则，A处则和一般原则相反

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202312181352896.png)

以上我们只讲述了SIMT stack是怎么使用的，现在思考一下它到底起到了什么作用，我们为何需要引入这样一种结构

观察上图中的(b)部分，不难发现程序的执行流从(a)那种复杂的形式，已经转变为了(b)中这种串行的方式。每个cycle执行一条指令即可，所以引入stack的目标就是 To achieve this serialization of divergent code paths one approach.

---

The SIMT stack helps eﬃciently handle two key issues that occur when **all threads can execute independently**:

1. nested control ﬂow
2. skipping computation

a warp is eligible to issue an instruction if it has a valid and ready (according to the scoreboard) in the I-Buffer. 


### Two-Loop Approximation
The problem of One-Loop Approximation is that it assumes that the warp will not issue another instruction until the first instruction completes execution, so maybe it will cause a long execution latencies.

A method to address this problem is that issue a subsequent instruction from a warp while earlier instructions have not yet completed, but it will face a new problem, we don't know whether the next instruction to issue for the warp has a dependency upon an earlier instruction that has not yet completed execution.

So a separate scheduler is introduced, it is used to decide which of several instructions in the instruction buﬀer should be issued next to the rest of the pipeline to avoid dependency problem.

总结一下，简单来说，所谓的two-loop approximation不过是面对one-loop approximation所遇到的问题，考虑额外添加一个调度器，在前一条指令还没有执行完毕时就能够发射其他指令以类似流水线的方式执行从而可以增加指令吞吐量，但是遇到一个依赖性的问题，可能还没有执行的指令和要发射的指令存在数据相关，所以就引入了计分板来尝试解决这个问题，然后又发现单纯使用计分板同样遇到了一些问题，然后就有一个大佬提出了一种解决方案。这整个过程就是一个发现问题，然后解决问题的循环。

#### Scoreboard
Scoreboards can be designed to support either in-order execution or out-of-order execution.

Scoreboarding keeps track of dependencies to make sure we do not allow an instruction to start executing if there is a dependency with a previous instruction that is still executing. As the following example shows:

```x86asm
add r3, r2, r1    // r3 = r2+r1
sub r5, r3, r4    // RAW
add r5, r2, r1    // WAW
```

1. After the first instruction issues, we mark r3 as unavailable.  
2. When the sub instruction arrives, it cannot issue since r3 is not ready (RAW).
3. After the first instruction completes, sub now can read the new value of r3 and issue, marking r5 which is the destination register as unavailable.
4. The third instruction cannot issue since it writes to r5 (WAW).

> Please note that in the above example, we issue in order: the read has already read the register values when we issue the write after it. So there is no WAR.

**The two problem and solve method of the above implementation of scoreboard is used in in-order execution**
1. The simple in-order scoreboard design needs too many storage space
> Solution: change the implementation of scoreboard
- The original way is hold a single bit per register per warp(每个warp都有一个完整的下图的结构), it looks like the following picture

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202312172233091.png)

- Now, the design contains a small number of entries per warp(每个warp一个bit vector), where each entry is the identifier of a register. It looks like the following picture

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202312172236391.png)

2. If an instruction that encounters a dependency must repeatedly lookup its operands in the scoreboard until the prior instruction it depends upon writes its results to the register file. It consumes too many computation resources

首先可以确定的一点在计分板结构改变后，维护计分板内容的方式也发生了改变。根据书上说的，改变了修改计分板的时机。我现在对于解决这个问题大致的一个理解是，计分板和指令buffer是两个分离的结构，当一条指令执行结束后会修改计分板的内容，然后顺便把指令buffer中对应存在依赖的寄存器标记清空，这样在从指令buffer中取指令的时候拿到的就是新的状态，如果从指令buffer中读取到的指令不存在对于某个寄存器的依赖时就去执行这一指令，如果存在就换别的执行，不过这块的逻辑还没有看的太明白。

### Three-Loop Approximation
#### Operand Collector
为了隐藏长时间的内存访问延迟，一种方法就是实现以周期为单位对warp进行切换，通过warp切换来掩盖延迟。

为了实现这一点，就需要使用较大的 registers file。而 registers file最朴素的实现方式就是 one port per operand per instruction issued per cycle， 但是这样我理解着是只能串行访问，吞吐量比较低。所以一种方式就是划分bank，不同bank可以做到并行访问，从而增大并行度。

引入了 bank，同时也就引入了bank conflict, 下图的naive microarchitecture就具有这种问题，设计operand collector也正是为了解决这种问题, operand collector就是引入的第3个scheduler，完成指令在并行访问寄存器堆时的调度工作。

naive microarchitecture for providing increased register file bandwidth(by single-ported logical banks of registers)

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202312181639756.png)


operand collector microarchitecture(the staging registers have been replaced with collector units)

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202312181640755.png)

operand collector究竟是怎么调度的似乎书上并没有详细描述，只是给了一种新的从 register 到 bank 的映射方式(如下图所示），确保不同 warp 原来会被分到同一个 bank 的register 现在会被分到不同bank，但是同一个 warp 内不同 thread 之间的 bank conflict 并没有解决, 也就是它只对减少不同 warp 间的 bank conflict 起到了作用。

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202312181709819.png)


具有 WAR hazard, 3种可能的解决方法：
1. release-on-commit warpboard: at most one instruction per warp to be executing
2. release-on-read warpboard: only one instruction at a time per warp to be collecting operands
3. instruction level parallelism 

#### Instruction Replay
当一条指令在GPU流水线上发生结构冒险了怎么办？对于一般的CPU来说我们可以简单的暂停当前指令，直到结构冒险消除再继续执行。但是这种方法在高吞吐量的系统中并不适用，停滞的指令可能处于任务的关键路径上进而影响到任务的完成时间，并且大量的停滞需要额外的缓冲区来存储寄存器信息。同时停滞一个指令可能会停滞其他完全不必要停滞的指令，降低了系统的吞吐量。

在GPU中我们可以尝试使用instruction replay来解决这个问题。instruction replay最早是在CPU的推测执行中作为一种恢复机制出现的，当我们执行了错误的分支，正确的指令会被重新取回并执行，消除错误分支的影响。在GPU中我们一般会避免推测执行，因为这会浪费宝贵的能源以及吞吐量。GPU实现instruction replay是为了减少流水线阻塞以及芯片面积和对应的时间开销。

在GPU上实现instruction replay可以通过在instruction buffer中保存当前指令直到这条指令已经执行完成，然后再将其移出。

### Warp

> The multiprocessor creates, manages, schedules, and executes threads in groups of 32 parallel threads called warps.

一个SM可能执行多个block。虽然说不同block之间可以并行执行（不过要求在不同SM上才可以并行），但是映射到同一个SM的block，它上面的warp是不能并行执行的，只能相互等待。

**How block’s threads get mapped to warps?**

We can get answer from [4.1. SIMT Architecture](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#simt-architecture).
> The way a block is partitioned into warps is always the same; each warp contains threads of consecutive, increasing thread IDs with the first warp containing thread 0.

从这个答案中，不难引发另一个疑问，即

**How thread ID can be calculated?**

We can get answer from [2.2. Thread Hierarchy](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#thread-hierarchy).
> The index of a thread and its thread ID relate to each other in a straightforward way: 
> - For a one-dimensional block, they are the same;
> - for a two-dimensional block of size $(D_x, D_y)$, the thread ID of a thread of index $(x, y)$ is $(x + y \times D_x)$; 
> - for a three-dimensional block of size $(D_x, D_y, D_z)$, the thread ID of a thread of index $(x, y, z)$ is $(x + y \times D_x + z \times D_x \times D_y)$.

(Editer replenishment): please note that the above comparison is between **index of thread** and **thread ID**, so dont's be confused about the first situation. i.e. "for a one-dimensional block, they are the same", it means for a one-dimensional block, the thread ID is equals to the index of this thread.

According to the question of "[Does CUDA think of multi-dimensional gridDim, blockDim and threadIdx just as a linear sequence?](https://stackoverflow.com/questions/31058001/cuda-griddim-blockdim-and-threadidx)", we can see the type of thread organization as the **row major ordered multi-dimensional arrays**. But please note the difference between the index in CUDA and the index of traditional array or matrix.

For traditional array or matrix, we are used to use the **(row_index, col_index)** to indicate the position of an element in an array or a matrix. But in CUDA, the coordinates seem to become adverse, CUDA uses the **(x = column_number, y = row_number)** to express a grid or block.

In fact, these two expressions don't create conflicts. The (row_index, col_index) is a perspective of actual storage mode. Now, if we place the array or the matrix into a coordinate system, we can also use the (x, y) to indicate an element of the array or matrix.

We can say that the (row_index, col_index) is a coordinate from storage structure perspective and the (x = column_number, y = row_number) is a coordinate from math coordinate system perspective.

Because the concept grid and block are just for programmer convenience, so they don't imply the actual storage structure, so the CUDA use the math coordinate to indicate the position of an element in an array or a matrix. For the thread index $(x, y)$, the x is the column number, y is the row number, it is like the following picture of block index.

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202312061851353.png)

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202312062024529.png)

**How to understand and calculate occupancy ?**

#### Warp Scheduling Strategy
1. Loose Round Robin (LRR)
处于Ready状态了就开始执行，否则跳过先发射下一个warp
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202312171630189.png)

2. Two-level (TL)
把warp分为两组，Pending warps 和 Active warps，warp在这两个组之间变换，当warp需要等待某些长延迟操作时，就切换到pending warp那一组，当条件就绪后，则转到active warp这一组，在active warp这一组采用LRR的调度策略
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202312171633345.png)

3. Greedy-then-oldest (GTO)
考虑到局部性，会贪婪地执行一个warp，直到它进入stall状态才会切换其他warp执行
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202312171635290.png)

## CUDA Related Documents

1. [NVIDIA CUDA Compiler Driver NVCC](https://docs.nvidia.com/cuda/cuda-compiler-driver-nvcc/index.html)

## Reference

> - [1] [CUDA C++ Programming Guide](https://docs.nvidia.com/cuda/pdf/CUDA_C_Programming_Guide.pdf)
> - [2] [Does NVCC include header files automatically?](https://forums.developer.nvidia.com/t/does-nvcc-include-header-files-automatically/48972)
> - [3] [网格跨步](https://lulaoshi.info/gpu/python-cuda/stride.html)
> - [4] [CUDA Runtime API Documentation](https://docs.nvidia.com/cuda/cuda-runtime-api/index.html) **(Please note the version of coda)**
> - [5] [CUDA编程方法论-知乎专栏](https://www.zhihu.com/column/c_1139113249399345152)
> - [6] [CUDA Crash Course - Youtube](https://www.youtube.com/playlist?list=PLxNPSjHT5qvtYRVdNN1yDcdSl39uHV_sU)
> - [7] [GPGPU架构优秀PPT(Teaching部分)](https://team.inria.fr/pacap/members/collange/)
> - [8] [Accelerated Computing - Programming GPUs](https://tschmidt23.github.io/cse599i/)
> - [9] [CUDA编程入门及优化 | 知乎](https://www.zhihu.com/column/c_1522503697624346624)
> - [10] [Tensor Core技术解析（上）](https://www.cnblogs.com/wujianming-110117/p/12992932.html)
> - [11] [Tensor Core技术解析（下）](https://www.cnblogs.com/wujianming-110117/p/12993096.html)
> - [12] [NVIDIA Developer Tools 汇总](https://developer.nvidia.com/tools-overview)
> - [13] [《General-Purpose Graphics Processor Architecture》中文翻译](https://zhuanlan.zhihu.com/p/522546744)