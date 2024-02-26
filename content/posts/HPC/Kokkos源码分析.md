---
title: Kokkos源码分析
subtitle:
description:
keywords:
summary:
license:
date: 2024-01-05T17:39:32+08:00
lastmod: 2024-02-26T09:20:16+08:00
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

## 对于Kokkos-core源代码结构的理解
注意只是对core这一部分的理解，从项目结构上可以看出，除了core这一部分外，还存在着例如algorithm, containers 和 simd等其他部分，其他部分也存在相关的代码。

在core/src下有一部分独立的文件，其中包含了最核心的Kokkos_Core，我们目前主要的关注点是当前目录下那些文件夹

这些文件夹，每个都对应了一个目标平台，起点是fwd文件夹，里面包含了涉及到不同目标平台的一些声明内容，主要是不同目标平台类的声明，以Kokkos::Serial为例
```cpp
namespace Kokkos {
class Serial;  ///< Execution space main process on CPU.
}  // namespace Kokkos
```
然后各个同名文件夹的则是对不同目标平台下涉及到的内容的具体定义


## DefaultExecutionSpace
关于DefaultExecutionSpace的定义：在core/src/Kokkos_Core_fwd.hpp中，需要根据编译选项来确定

和宏定义相关的一个文件: Kokkos_Macros.hpp

关于文档中，下面这句话的代码表述如以下代码所示[Kokkos::DefaultExecutionSpace is an alias of ExecutionSpace() type pointing to an ExecutionSpace based on the current configuration of Kokkos.](https://kokkos.github.io/kokkos-core-wiki/API/core/execution_spaces.html#kokkos-defaultexecutionspace:~:text=Kokkos%3A%3ADefaultExecutionSpace%20is%20an%20alias%20of%20ExecutionSpace()%20type%20pointing%20to%20an%20ExecutionSpace%20based%20on%20the%20current%20configuration%20of%20Kokkos.)

DefaultExecutionSpace是具体目标平台的别名，通过配置项结合预编译指令实现到具体平台的映射。

DefaultExecutionSpace还影响到了默认的memory space。在KokkosTutorial_02_ViewsAndSpaces.pdf中给出了如下观点
> Each execution space has a default memory space

使用默认memory space等价于指定memory space为[Kokkos::DefaultExecutionSpace::memory_space](https://kokkos.github.io/kokkos-core-wiki/API/core/view/view.html#parameters:~:text=Kokkos%3A%3ADefaultExecutionSpace%3A%3Amemory_space)(这一点在API文档的View中给出的，这是合理的，因为就是在创建View时才需要指定memory space)

初次看到Kokkos::DefaultExecutionSpace::memory_space来指定默认memory space是有些疑惑的，想象中的做法是应该像execution space一样存在一个Kokkos::DefaultMemorySpace，但是memory的属性却是放在execution属性下面的。结合前面所说的，这样做是合理的，之前说每一个execution space都对应一个默认的memory space，就是通过这种方式实现的。

```cpp
#if defined(KOKKOS_ENABLE_DEFAULT_DEVICE_TYPE_CUDA)
using DefaultExecutionSpace KOKKOS_IMPL_DEFAULT_EXEC_SPACE_ANNOTATION = Cuda;
#elif defined(KOKKOS_ENABLE_DEFAULT_DEVICE_TYPE_OPENMPTARGET)
using DefaultExecutionSpace KOKKOS_IMPL_DEFAULT_EXEC_SPACE_ANNOTATION =
    Experimental::OpenMPTarget;
#elif defined(KOKKOS_ENABLE_DEFAULT_DEVICE_TYPE_HIP)
using DefaultExecutionSpace KOKKOS_IMPL_DEFAULT_EXEC_SPACE_ANNOTATION = HIP;
#elif defined(KOKKOS_ENABLE_DEFAULT_DEVICE_TYPE_SYCL)
using DefaultExecutionSpace KOKKOS_IMPL_DEFAULT_EXEC_SPACE_ANNOTATION =
    Experimental::SYCL;
#elif defined(KOKKOS_ENABLE_DEFAULT_DEVICE_TYPE_OPENACC)
using DefaultExecutionSpace KOKKOS_IMPL_DEFAULT_EXEC_SPACE_ANNOTATION =
    Experimental::OpenACC;
#elif defined(KOKKOS_ENABLE_DEFAULT_DEVICE_TYPE_OPENMP)
using DefaultExecutionSpace KOKKOS_IMPL_DEFAULT_EXEC_SPACE_ANNOTATION = OpenMP;
#elif defined(KOKKOS_ENABLE_DEFAULT_DEVICE_TYPE_THREADS)
using DefaultExecutionSpace KOKKOS_IMPL_DEFAULT_EXEC_SPACE_ANNOTATION = Threads;
#elif defined(KOKKOS_ENABLE_DEFAULT_DEVICE_TYPE_HPX)
using DefaultExecutionSpace KOKKOS_IMPL_DEFAULT_EXEC_SPACE_ANNOTATION =
    Kokkos::Experimental::HPX;
#elif defined(KOKKOS_ENABLE_DEFAULT_DEVICE_TYPE_SERIAL)
using DefaultExecutionSpace KOKKOS_IMPL_DEFAULT_EXEC_SPACE_ANNOTATION = Serial;
```

## KOKKOS_DEVICES的内容到底影响了什么
以Serial为例，如果在KOKKOS_DEVICES中未添加Serial，但是在代码中使用了Kokkos::Serial会报错

加上之后，编译包含main的文件的编译命令的-I选项并没有发生变化，只是多编译了2个和Serial有关的cpp文件。

所以只是影响了链接过程，但是头文件的问题是如何解决的

Kokkos_Core_fwd.hpp的fwd解释：
"Fwd" 在这里通常是"forward"的缩写，用于表示前向声明（forward declaration）。在C++中，前向声明是一种声明但不定义实体的技术。这通常用于避免引入完整的定义，从而提高编译速度和减少依赖关系。

## KOKKOS_LAMBDA
Use the KOKKOS_LAMBDA macro to replace a lambda’s capture clause when giving the lambda to Kokkos for parallel execution.

KOKKOS_LAMBDA is the same as [=] 体现在Kokkos_Macros.hpp 中

```cpp
#if !defined(KOKKOS_LAMBDA)
#define KOKKOS_LAMBDA [=]
#endif
```

## KOKKOS_INLINE_FUNCTION
Use the KOKKOS_INLINE_FUNCTION macro to mark a functor’s methods that Kokkos will call in parallel

在Kokkos_Macros.hpp中

```cpp
#define KOKKOS_INLINE_FUNCTION KOKKOS_IMPL_INLINE_FUNCTION

#if !defined(KOKKOS_IMPL_INLINE_FUNCTION)
#define KOKKOS_IMPL_INLINE_FUNCTION inline
#endif
```
在KokkosTutorial_02_ViewsAndSpaces.pdf中有一点介绍，上面的代码找的应该是不全，实际会根据其他宏的内容还存在修改
![](https://img2024.cnblogs.com/blog/1898659/202401/1898659-20240106163903443-652671311.png)


## 为何只采用value-copy
 In particular, the functor might need to be copied to a different execution space than the host. For this reason, it is generally not valid to have any pointer or reference members in the functor. 

## Functors
 A functor is one way to define the body of a parallel loop. It is a class or struct1 with a public operator() instance method.

## 疑惑/可能的改进点
Kokkos代码中存在着大量的例如`#ifdef`这类的预处指令，Kokkos本身的可移植恰恰是通过这一点实现的（想要做到可移植，抽象是必须的，要在多种不同的硬件之上构建起一个逻辑层，在抽象层之下，需要解决的就是编译问题，例如使用Openmp和使用Cuda，编译器和编译选项显然是存在区别的，Kokkos解决这个问题的方法就是通过各种宏，首先通过配置项生成宏，然后宏会渗入到cpp代码当中，根据宏对类型等内容进行选择）。问题在于很多代码的宏定义之间甚至存在逻辑关系，这这给代码带来的极大的不易读性，因为代码的真实执行逻辑取决于宏的定义，在没有执行的情况下想要理清逻辑，甚至需要手动推导宏。


## 关于异步机制的疑问
遇到问题的场景是这样的：

正在尝试使用View和Mirror，下述代码可以正常编译，但是运行时会报错

> terminate called after throwing an instance of 'std::runtime_error'
> 
> what():  Kokkos allocation "d_x" is being deallocated after Kokkos::finalize was called
> 
> Aborted (core dumped)

意思就是，在Kokkos::finalize调用之后，又去销毁d_x对应的存储空间。很自然就能够想到是同步的问题，因此查询文档，找到`Kokkos::fence()`同步函数。
但是使用函数之后，依然会报这个错误


文档中这样描述这个函数
> Blocks on completion of all outstanding asynchronous Kokkos operations.

后来想明白之后，猜测它这个同步，只是保证了在其他设备中的操作完成了，但是同步之后存储资源仍然是没有销毁的，所以仍然会导致上述问题。

想要解决，就需要保证在Kokkos::fence()之前把对应的存储资源销毁，但是从示例代码中并没有看到相关的API，那么这个过程应当是自动完成的。所以就可以利用作用域这个机制来实现了。

```cpp
#include <Kokkos_Core.hpp>

int main(int argc, char **argv) {
  Kokkos::initialize( argc, argv );

  {
    int N = 2;

    Kokkos::View<int *, Kokkos::Cuda::memory_space> d_x("d_x", N);
    Kokkos::View<int *, Kokkos::Cuda::memory_space>::HostMirror h_x = Kokkos::create_mirror_view(d_x);

    Kokkos::parallel_for(
      "Loop in CPU", 
      Kokkos::RangePolicy<Kokkos::Serial>(
            Kokkos::Serial(), 0, N
      ),
      KOKKOS_LAMBDA(int i) {
        h_x(i) = 1;
      }
    );

    Kokkos::deep_copy(d_x, h_x);

    Kokkos::parallel_for(
      "Loop in GPU", 
      Kokkos::RangePolicy<Kokkos::Cuda>(
            Kokkos::Cuda(), 0, N
      ),
      KOKKOS_LAMBDA(int i) {
        d_x(i) = 1;
        printf("%d ", d_x(i));
      }
    );
    std::printf("\n");
  
    // Kokkos::fence(); // 同步
  }

  Kokkos::finalize();

  return 0;
}
```

在Kokkos中哪些操作是异步的，在[fence](https://kokkos.github.io/kokkos-core-wiki/API/core/parallel-dispatch/fence.html?highlight=fence)这里有提到一些

## deep_copy
core/src/Kokkos_CopyViews.hpp下包含多种`inline void deep_copy(){}`

部分`deep_copy(){}`调用了`Kokkos::Impl::DeepCopy`

DeepCopy似乎是一个struct，可查询`struct DeepCopy`

此结构体的声明和定义出现在Kokkos_Core_fwd.hpp以及不同目标平台文件夹下的xxx_DeepCopy.hpp中

