---
categories:
- 并行计算
comment: false
date: '2023-05-31T11:17:35+08:00'
description: null
draft: false
fontawesome: true
fraction: true
hiddenFromHomePage: false
hiddenFromSearch: false
keywords: null
lastmod: 2023-11-23T10:40:38.652+8:0
license: null
lightgallery: force
math: true
message: null
password: null
repost:
  enable: false
  url: null
resources:
- name: featured-image
  src: featured-image.jpg
- name: featured-image-preview
  src: featured-image-preview.jpg
ruby: true
subtitle: null
summary: null
tags: []
title: GPU Structure and Programing(CUDA)
toc: true
weight: 0
---
> - CUDA C只是对标准C进行了语言级的扩展，通过增加一些修饰符使编译器可以确定哪些代码在主机上运行，哪些代码在设备上运行
> - GPU计算的应用前景很大程度上取决于能否从问题中发掘出大规模并行性

## SIMD & SIMT

涉及到AVX指令，正在尝试
说明两种方式的区别

The two most important things about SIMD and SIMT are:

1. How is the SIMT to implement ?
2. How is the SIMD to calculate ?

## Kernel hardware mapping

kernel function -> GPU
block -> SM（one block can only be executed by one SM, but one SM can execute multiple blocks)
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
7. sp

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

### Register & Local Memory


### Shared Memory

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

### Global Memory
#### coalesced & uncoalesced

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

### Warp

> The multiprocessor creates, manages, schedules, and executes threads in groups of 32 parallel threads called warps.

一个SM可能执行多个block。虽然说不同block之间可以并行执行（不过要求在不同SM上才可以并行），但是映射到同一个SM的block，它上面的warp是不能并行执行的，只能相互等待。

## Softwarhhe stack

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

When you don't specify the type of function, the default is the `__host__`
Host can call `__global__`, `__global` can call `__device__`, `__device__` can call `__device__`

## Memory

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

## Common Parallelization methods

1. Grid-stride loop
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

2. Another way to solve the data conflict
   The most obvious answer is using mutex or atomic operation. But as we all know, whether it's mutex or atomic, they both have some consuming.
   We know that the data conflict comes from shared data, different thread maybe use the same data at the same time. So an approach to avoid happening this problem is that control different threads use different data.
   According to the process of Grid-stride loop, we notice that different threads use the different datas which have different locations. We can specify a fixed location to store a thread's data to avoid using the mutex or atomic.
   A good example is array summation. As the following code shows.

```
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

## C++ Encapsulation

As we all know, the style of many CUDA APIs is C-style, we need to learn about how to use it conjunction with C++.

**How does the std::vector standard template library use the Device(GPU) memory ?**
Many examples use the original pointer to point a Device memory. But if we want to use a std::vector or other standard template library that locates in Device memory, we can't use the `cudaMalloc()` or `cudaMallocManaged()`.

Taking the `std::vector` as an example, next, we will discuss the method of allocating Device memory for containers.

> Whether it's principle or usage methods is too complex to understand in a short time. So pause it for a period of time. When we must need to learn its principle we study it again. We can learn about it from [一篇文章搞懂STL中的空间配置器allocator](https://www.coonote.com/cplusplus-note/space-allocator.html). In short, std::allocator integrates the memory management and object management by using four member function.

## GPU execution core

一个kernel函数在逻辑上以block为单位映射到SM中，物理上以warp为单位解析指令将指令分发到具体的运算单元(SP/core, SFU, DP)或访存单元(LD/ST)。
SM中活动的warp数量占物理warp数量的比率为occupancy(占用率)。

## CUDA Compilation

涉及到两部分内容，一部分是cuda面对编译问题时的设计架构，另一方面是cuda实际的编译流程

首先对CUDA程序的编译流程进行简要介绍，下图是[NVIDIA CUDA Compiler Driver NVCC - The CUDA Compilation Trajectory](https://docs.nvidia.com/cuda/cuda-compiler-driver-nvcc/index.html#the-cuda-compilation-trajectory)中给出的cuda编译流程。

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202311141110954.png)

上图可以结合实际流程和用到的指令来理解，这些可以通过`nvcc -dryrun <cuda program name>`来获取到

> `-dryrun`: List the compilation sub-commands without executing them.

生成的中间文件，可以通过`nvcc -keep`来获取到

> `-keep`: Keep all intermediate files that are generated during internal compilation steps.

下面结合编译流程主要说明以下几个问题：

1. 何为PTX，为何会设计它
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
> 这一点内容详见官方文档[--gpu-architecture (-arch)](https://docs.nvidia.com/cuda/cuda-compiler-driver-nvcc/index.html#gpu-architecture-arch)和[--gpu-code code (-code)](https://docs.nvidia.com/cuda/cuda-compiler-driver-nvcc/index.html#gpu-code-code-code)

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

## Related Programming Models

1. OpenCL
2. OpenACC
3. MPI
   process parallelism
4. OpenMP
   threaded parallelism

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

### Reference

> - [1] [Working with GPGPU-Sim - Introduction](https://coffeebeforearch.github.io/2020/03/30/gpgpu-sim-1.html)
> - [2] [Working with GPGPU-Sim - Adding Configuration Options](https://coffeebeforearch.github.io/2020/04/13/gpgpu-sim-2.html)
> - [3] [Improving GPGPU-Sim Performance](https://coffeebeforearch.github.io/2020/03/31/perf-gpgpu-sim.html)
> - [4] [ECE 695 GPGPU-Sim Tutorial 学习笔记](https://zhuanlan.zhihu.com/p/576442425)

附加内容：

1. If want to use ptxplus (native ISA) change the following options in the configuration file

> -gpgpu_ptx_use_cuobjdump 1
> -gpgpu_ptx_convert_to_ptxplus 1

2. If want to use GPUWatch change the following options in the configuration file

> -power_simulation_enabled 1 (1=Enabled, 0=Not enabled)
> -gpuwattch_xml_file <filename>.xml

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
