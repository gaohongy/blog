---
title: Kokkos Source Code Analysis
subtitle:
description:
keywords:
summary:
license:
date: 2024-01-05T17:39:32+08:00
lastmod: 2024-02-29T15:48:06+08:00
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

## Usage

### Install & CMake
```bash
cd some_software-1.4.2
mkdir build
cd build

cmake .. 
cmake --build . # It is equivalent to make
cmake --build . --target install # It is equivalent to make install
```

After the above flow, we can use Kokkos by CMake directly.

### Source Code & CMake[^Using-Kokkos-in-tree-build]
[^Using-Kokkos-in-tree-build]: [Using Kokkos in-tree build](https://kokkos.org/kokkos-core-wiki/ProgrammingGuide/Compiling.html#using-kokkos-in-tree-build)

file sturcture:
```bash
.
├── CMakeLists.txt
├── kokkos-4.1.2
├── main.cpp
└── Makefile
```

```cmake
# CMakeLists.txt 内容

cmake_minimum_required(VERSION 3.16)
project(Example CXX)

# add_subdirectory(/home/hongyu_gao2001/kokkos/kokkos-tutorials/Exercises/01/My/kokkos /home/hongyu_gao2001/kokkos/kokkos-tutorials/Exercises/01/My/build/kokkos)
add_subdirectory(/home/hongyu_gao2001/kokkos/kokkos-tutorials/Exercises/01/My/kokkos-4.1.2)

add_executable(example main.cpp)

target_link_libraries(example kokkos)
```

有几个注意事项：

1. `add_subdirectory`有两个形式。这两个形式有两个关键的不同点，一是第一个参数不同，二是是否存在第二个参数。第二个参数的作用是指明输出的路径，如果没有此参数，则输出的名称和第一个参数相同。关于这个名称，我以为会和target_link_libraries的第2个参数相关，但是即使不同也可以正常编译

2. 正如上面所提到的，target_link_libraries的第2个参数，目前不知道是和什么相关的，不知道是写明在项目哪里的

3. 在[官方](https://kokkos.org/kokkos-core-wiki/ProgrammingGuide/Compiling.html#using-kokkos-in-tree-build)给出的指示中，提到了另一个语句`include_directories(${Kokkos_INCLUDE_DIRS_RET})`，但是我通过`MESSAGE( STATUS "${Kokkos_INCLUDE_DIRS_RET}")`发现此变量为空，但是编译并没有报错，在一篇博客中发现一个可能的原因是target_link_directories链接的库包含了头文件，所以这里不添加也可以[^no-Kokkos_INCLUDE_DIRS_RET]

[^no-Kokkos_INCLUDE_DIRS_RET]: [为何不用显示调用target_include_directories](https://brightxiaohan.github.io/CMakeTutorial/FindPackage/#:~:text=%23%20%E7%94%B1%E4%BA%8Eglog%E5%9C%A8%E8%BF%9E%E6%8E%A5%E6%97%B6%E5%B0%86%E5%A4%B4%E6%96%87%E4%BB%B6%E7%9B%B4%E6%8E%A5%E9%93%BE%E6%8E%A5%E5%88%B0%E4%BA%86%E5%BA%93%E9%87%8C%E9%9D%A2%EF%BC%8C%E6%89%80%E4%BB%A5%E8%BF%99%E9%87%8C%E4%B8%8D%E7%94%A8%E6%98%BE%E7%A4%BA%E8%B0%83%E7%94%A8target_include_directories)

### Source Code & Raw Makefile

There is a confused point: How to chose the right backend? If we use the makefile which is provided by Kokkos-tutorials we can edit the `KOKKOS_DEVICES`. But we don't know what does this option do. When we use `g++ -I -L -l` to compile code, even though we can get the executable file, but we can't get the ideal execute performance.


目前对于 Kokkos 是如何调用 backend 的接口这件事情存有疑问，尚不明确 kokkos-tutorials/Exercises 中给出的各个示例中的 Makefile 具体都做了哪些工作，从而使得可以正常使用各种 hetetogenous backends(因为他们都包含一个`include $(KOKKOS_PATH)/Makefile.kokkos`, 目前还不太清楚这个文件具体都完成了什么工作). 因为直接使用 `g++` 进行编译似乎无法达到这种效果。

## Initialization

An important thing which initialization progress does is initializing `Kokkos::DefaultExecutionSpace;` and `Kokkos::DefaultHostExecutionSpace;`. [^initialization]

[^initialization]: [initialization aliases](https://kokkos.org/kokkos-core-wiki/ProgrammingGuide/Initialization.html#:~:text=During%20initialization%2C%20one%20or%20more%20execution%20spaces%20will%20be%20initialized%20and%20assigned%20to%20one%20of%20the%20following%20aliases.)

## 对于Kokkos-core源代码结构的理解
注意只是对core这一部分的理解，从项目结构上可以看出，除了core这一部分外，还存在着例如algorithm, containers 和 simd等其他部分，其他部分也存在相关的代码。

在core/src下有一部分独立的文件，其中包含了最核心的Kokkos_Core

core/src下的文件夹，每个都对应了一个目标平台，起点是core/src/fwd文件夹，里面包含了涉及到不同目标平台的一些前向声明内容，主要是不同目标平台类的声明

> "Fwd" 在这里通常是"forward"的缩写，用于表示前向声明（forward declaration）。在C++中，前向声明是一种声明但不定义实体的技术。这通常用于避免引入完整的定义，从而提高编译速度和减少依赖关系。 

以Kokkos::Serial为例, 前向声明内容为：

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


## Parallel Loop Body

### Functor
A functor is one way to define the body of a parallel loop. It is a class or struct1 with a public operator() instance method.

Use the `KOKKOS_INLINE_FUNCTION` or `KOKKOS_INLINE` macro to mark a functor’s methods that Kokkos will call in parallel

在Kokkos_Macros.hpp中

```cpp
#define KOKKOS_INLINE_FUNCTION KOKKOS_IMPL_INLINE_FUNCTION

#if !defined(KOKKOS_IMPL_INLINE_FUNCTION)
#define KOKKOS_IMPL_INLINE_FUNCTION inline
#endif
```
在KokkosTutorial_02_ViewsAndSpaces.pdf中有一点介绍，上面的代码找的应该是不全，实际会根据其他宏的内容还存在修改

![](https://img2024.cnblogs.com/blog/1898659/202401/1898659-20240106163903443-652671311.png)

### Lambda

Use the `KOKKOS_LAMBDA` macro to replace a lambda’s capture clause when giving the lambda to Kokkos for parallel execution.

KOKKOS_LAMBDA is the same as [=] 体现在Kokkos_Macros.hpp 中

```cpp
#if !defined(KOKKOS_LAMBDA)
#define KOKKOS_LAMBDA [=]
#endif
```

### Lambda 为何采用 value-copy[^lambda-value-copy]

[^lambda-value-copy]: [Kokkos semantics to capture by value [=]](https://kokkos.org/kokkos-core-wiki/ProgrammingGuide/ParallelDispatch.html#functors:~:text=It%20is%20a%20violation%20of%20Kokkos%20semantics%20to%20capture%20by%20reference%20%5B%26%5D%20for%20two%20reasons.)

1. portability

In particular, the functor might need to be copied to a different execution space than the host. For this reason, it is generally not valid to have any pointer or reference members in the functor. 

2. correctness
- Capturing by reference allows the programmer to violate the const semantics of the lambda.

// TODO
// 如何理解?

- Capturing by reference enables many more possibilities of writing non-threads-safe code

```cpp
int val = 0;

Kokkos::parallel_for("for", 10, [&](const int i) -> void {
    val += i;
});

std::cout << val << std::endl;
```

The right result is $55$, because `parallel_for` is asynchronous, so the `val += i` is non-thread-safe.

## parallel_reduce

根据目前的理解，`parallel_reduce`可以解决在`parallel_for`中需要使用 capture-by-reference 但是又存在 non-thread-safe 的问题。

核心执行方式为：each iteration produces a value and these iteration values are accumulated into a single value with a user-specified associative binary operation[^Parallel_reduce-execution-way]

[^Parallel_reduce-execution-way]: [Parallel_reduce执行方式](https://kokkos.org/kokkos-core-wiki/ProgrammingGuide/ParallelDispatch.html#parallel-reduce:~:text=each%20iteration%20produces%20a%20value%20and%20these%20iteration%20values%20are%20accumulated%20into%20a%20single%20value%20with%20a%20user%2Dspecified%20associative%20binary%20operation.)

这句话中有两个重点：

1. "a single value"，这代表着 `parallel_reduce` 参数中的 result，也就是最后结果存储的位置
2. "a user-specified associative binary operation"，这代表着 `parallel_reduce` 参数中的 reducer，目前理解就是表示执行方式

相较于 `paralle_for`，lambda 中参数发生了变化:[^lambda-operator-content]

[^lambda-operator-content]: [two arguments of lambda](https://kokkos.org/kokkos-core-wiki/ProgrammingGuide/ParallelDispatch.html#parallel-reduce:~:text=The%20lambda%20or%20the%20operator()%20method%20of%20the%20functor%20takes%20two%20arguments.)

- The first argument is the parallel loop “index”，这点没变化
- The second argument is a **non-const reference** to a **thread-local variable** of the same type as the reduction result. （其中的重点已经加粗）

这是一段计算 $A \* x \* y$ 的代码，其中 $A$ 是 $N \* M$ 的矩阵，$x$ 是 $M \* 1$ 的矩阵，$y$ 是长度为 $N$ 的系数矩阵
```cpp
for ( int i = 0; i < N; ++i ) {
  double temp2 = 0;

  for ( int j = 0; j < M; ++j ) {
    temp2 += A[ i * M + j ] * x[ j ];
  }

  result += y[ i ] * temp2;
}
```

当采用 `parallel_for` 时，需要采用 capture-by-reference，这将会导致 non-thread-safe问题，最终的结果会出现错误，代码如下所示

```cpp
Kokkos::parallel_for("compute", N, [&](const int j) -> void {
  double temp2 = 0;
  
  for (int i = 0; i < M; i++) {
    temp2 += A[j * M + i] * x[i];
  }

  result += y[j] * temp2;
});
```

得到正确结果的方案是采用 `parallel_reduce`，代码如下所示

```cpp
Kokkos::parallel_reduce("matrix multiplication", N, KOKKOS_LAMBDA (const int j, double &update) -> void {
  double temp2 = 0.0;

  for (int i = 0; i < M; i++) {
    temp2 += A[j * M + i] * x[i];
  }
  
  update += y[j] * temp2;
}, result);
```

### `parallel_reduce` thread-safe 的实现机理

> The source code is located in `core/src/Kokkos_Parallel_Reduce.hpp`

`ParallelReduceReturnValue`相关声明和定义（为了便于区分不同类型，将结构体内容删除，仅留下模版参数）：

```cpp
template <class T, class ReturnType, class ValueTraits>
struct ParallelReduceReturnValue;

template <class ReturnType, class FunctorType>
struct ParallelReduceReturnValue<
    std::enable_if_t<Kokkos::is_view<ReturnType>::value>, 
    ReturnType,
    FunctorType> 
    {};

template <class ReturnType, class FunctorType>
struct ParallelReduceReturnValue<
    std::enable_if_t<!Kokkos::is_view<ReturnType>::value &&
                     (!std::is_array<ReturnType>::value &&
                      !std::is_pointer<ReturnType>::value) &&
                     !Kokkos::is_reducer<ReturnType>::value>,
    ReturnType, 
    FunctorType> 
    {};

template <class ReturnType, class FunctorType>
struct ParallelReduceReturnValue<
    std::enable_if_t<(std::is_array<ReturnType>::value ||
                      std::is_pointer<ReturnType>::value)>,
    ReturnType, 
    FunctorType> 
    {};

template <class ReturnType, class FunctorType>
struct ParallelReduceReturnValue<
    std::enable_if_t<Kokkos::is_reducer<ReturnType>::value>,
    ReturnType,
    FunctorType> 
    {};
```

从后续函数模版`parallel_reduce`的实现中，不难看出，`parallel_reduce`去调用了两个函数:

1. `Impl::ParallelReduceAdaptor<policy_type, FunctorType, ReturnType>::execute()`

`Impl::ParallelReduceAdaptor`是一个`struct`,其中包含了

```cpp
template <typename Dummy = ReturnType>
static inline std::enable_if_t<!(is_array_reduction &&
                                 std::is_pointer<Dummy>::value)>
execute(const std::string& label, const PolicyType& policy,
        const FunctorType& functor, ReturnType& return_value) {
  execute_impl(label, policy, functor, return_value);
}
```

> `std:;enable_if_t`作用为条件编译，大致可以理解为只有在满足条件时才会示例化此函数模版，主要用于为函数重载提供便利

从`execute_impl`会导向具体 backend 的 Impl::ParallelReduce 的实现

以上整体的调用流程如下：

1. Kokkos_Parallel_Reduce.hpp 的 parallel_reduce （入口）
2. Kokkos_Parallel_Reduce.hpp 的 Impl::ParallelReduceAdaptor::execute() (看为execute的声明)
    2.1 Kokkos_Parallel_Reduce.hpp 的 Impl::ParallelReduceAdaptor::execute_impl() (看为execute的定义)
        2.1.1 不同 backend 下的 Impl::ParallelReduce (从 execute_impl 到不同的 backend 实现机理？依靠模版类中模版参数不同对应到不同的模版类中？)
3. Impl::ParallelReduceFence::fence()

从 Impl::ParallelReduceAdaptor::execute_impl() 转到正确 backend 下的 Impl::ParallelReduce，在模版参数上有一个关键点是 Impl::FunctorPolicyExecutionSpace<FunctorType, PolicyType>::execution_space
这一内容位于 Kokkos_Parallel.hpp 中的 struct FunctorPolicyExecutionSpace，其中包含对于 execution_space 的下述表述

```cpp
using execution_space = detected_or_t<
    detected_or_t<
        std::conditional_t<
            is_detected<device_type_t, Functor>::value,
            detected_t<execution_space_t, detected_t<device_type_t, Functor>>,
            Kokkos::DefaultExecutionSpace>,
        execution_space_t, Functor>,
    execution_space_t, Policy>;
```

我们最后可以得到一个结论：kokkos的规范侧重于声明和定义分离。parallel_reduce的实现中包含一个 ParallelReduceAdaptor 和 ParallelReduce，之所以叫为 Adaptor，现在看来是实现了一个任务分发的工作，先把任务集中于此，然后派发到不同的 backend 下

2. `Impl::ParallelReduceFence<typename policy_type::execution_space, ReturnType>::fence()`

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

## Reference