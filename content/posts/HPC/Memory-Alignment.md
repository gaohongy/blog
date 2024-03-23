---
title: Memory Alignment
subtitle:
description:
keywords:
summary:
license:
date: 2024-03-23T21:08:23+08:00
lastmod: 2024-03-24T00:11:46+08:00
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

内存的编址单位是字节

但是数据读取单位可能是双字节、四字节、8字节、16字节、32字节等，此单位被称为内存存取粒度。

> 对于上述内容，存在两点需要说明：
> 
> 1. 为何在编址单位之外还要设计数据读取单位？
>
> 答：以字节为单位进行读取效率太低，这和扇区的设计理念是差不多的
> 
> 2. 关于内存存取粒度，一个更数学化以及计算机化的表述应当为: **计算机仅会从地址为内存存取粒度的整数倍的位置** 进行数据读取

考虑在某种内存存取粒度下，假设并不存在内存对齐机制，那么数据将可以随意存放，但是计算机读取数据对于地址是存在要求的，那么当数据存放并不符合这种要求时，就需要进行一些额外的工作，从而降低性能

以上内容参考自[C/C++内存对齐详解](https://zhuanlan.zhihu.com/p/30007037)


我感觉理解内存对齐的难点实际在于：理解某种对齐机制到底会对访存造成什么影响，为何在这种机制下的访存性能就会更高

> 只有在了解现有存在的问题后才能想到优化方案

因为无论是 C++11 引入的 [alignas specifier](https://en.cppreference.com/w/cpp/language/alignas) 还是 

> 这块有一个从 C++11 发展到 C++17 的发展过程，同时涉及到对 alignas specifier，分布在栈上的struct，new分配的在堆上的struct，new分配的堆数据 这些概念的一个解读
>
> 具体内容参见 Cpp17.pdf 的 Chapter 30 使用 new 和 delete 管理超对齐数据
>
> 当提到超对齐数据时，就会涉及到同 [C/C++内存对齐详解](https://zhuanlan.zhihu.com/p/30007037) 中提到的 **有效对齐值** 之间的一个对比，同时有效对齐值的概念又牵扯出 [Pragmas](https://gcc.gnu.org/onlinedocs/cpp/Pragmas.html)关键词 的概念
>
> 总结来说，这里涉及到两层知识的扩展，暂时不展开

他们在本质上提出的所谓对齐，从数据上来讲，就是使得 最终整个结构体的大小 或者 分配的内存空间的大小 是 要求对齐量 的整数倍，关于这一点可通过下述示例进行验证：

```cpp
struct XYZ{
  double x;
  double y;
  double z;
};

struct alignas(16) XYZ16{
  double x;
  double y;
  double z;
};

std::cout << sizeof(XYZ) << std::endl; // 24
std::cout << sizeof(XYZ16) << std::endl; // 32
```

正如前面所提到的，这个示例也仅仅能够说明内存对齐在数值上的影响，并没有体现出对齐后会对性能产生哪些影响

