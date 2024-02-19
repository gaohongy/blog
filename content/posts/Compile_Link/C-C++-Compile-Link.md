---
title: C C++ Compile Link
subtitle:
description:
keywords:
summary:
license:
date: 2023-12-31T21:54:31+08:00
lastmod: 2024-02-20T00:19:38+08:00
tags:
categories:
  - Compile_Link
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

## 名词辨析
### GNU
- GNU's Not Unix!的递归缩写
- 一个自由的**操作系统**,起源于GNU计划,希望发展出一套完整的开放源代码操作系统来取代Unix
- 基本组成包括：
  - [GNU编译器套装（GCC）]()
  - GNU的C库（glibc）
  - GNU核心工具组（coreutils）

### GCC
- [GNU Compiler Collection](https://gcc.gnu.org/index.html), GNU编译器套装，最初是为了GNU操作系统而编写的编译器。
- 有多种[语言前端](https://www.cnblogs.com/G-H-Y/p/17061801.html)，可用于解析不同的编程语言、操作系统、计算机系统结构，是GNU计划的关键部分，也是GNU工具链的主要组成部分之一
- 可以编译C、C++、JAV、Fortran、Pascal、Object-C、Ada，Go等语言

### gcc/g++/MinGW
- gcc: GCC中的GUN C Compiler（C 编译器）
- g++: GUN C++ Compiler（C++编译器）
- MinGW: Minimalist GNU for Windows，是将GCC编译器和GNU Binutils移植到Win32平台下的产物

但根据[GCC的gcc和g++区别](https://www.cnblogs.com/samewang/p/4774180.html)的说法，gcc和g++并不是编译器，它们只是一种驱动器[^driver]，它们会根据参数中要编译的文件的类型，调用对应的GUN编译器。以编译C语言为例，包含以下过程。
> Step1：Call a preprocessor, like cpp.
> 
> Step2：Call an actual compiler, like cc or cc1.
> 
> Step3：Call an assembler, like as.
> 
> Step4：Call a linker, like ld

因此gcc命令只是上述后台程序的包装，根据不同的参数调用不同的程序，例如预编译程序、编译器、汇编器和链接器

[^driver]: 关于驱动器的说法，目前只在[gcc/g++链接选项](https://gcc.gnu.org/onlinedocs/gcc/Link-Options.html#:~:text=Therefore%2C%20the-,G%2B%2B%20driver,-automatically%20adds%20%2Dshared)一文中看到相关说法

**两者的联系和区别**
> 对于 *.c文件，gcc当做c文件看待，g++当做cpp文件看待

虽然gcc和g++都可以编译*.c文件，但是二者会以不同的语言来对待c文件，而C++ 标准和 C 语言标准的语法要求是有区别的。
```
#include <stdio.h>
int main()
{
    const char * a = "abc";
    printStr(a);
    return;
}
int printStr(const char* str)
{
    printf(str);
}
```
以上代码使用gcc进行编译，其会看为c语言，编译结果为
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/202301200805118.png)
以上代码使用g++进行编译，其会看为c++，编译结果为
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/202301200806353.png)
由此可见，c++的语言要求会更高一些

> 对于 *.cpp文件，gcc当做cpp文件看待，g++当做cpp文件看待

 虽然二者都会以cpp文件来对待，但是对于调用某些标准库中现有的函数或者类对象的c++程序，而单纯的 gcc 命令无法自动链接这些标准库文件，无法完成编译

### MSVC
- Microsoft Visual C++，is a compiler for the C, C++ and C++/CX programming languages by Microsoft

### LLVM
LLVM最初是指Low Level Virtual Machine，是类似但不同于jvm的一种虚拟机，现在来说，有很多理解方式，可以说LLVM是编译器的工具链的[集合](https://llvm.org/docs/CommandGuide/)，Clang是使用LLVM的编译器；又或者说LLVM是一个优秀的编译器框架，它也采用经典的三段式设计
根据[编译原理](https://www.cnblogs.com/G-H-Y/p/17061801.html)可以了解到，在GCC中前端和后端的分界并非明显，这就导致出现下面的情况，一种语言的前端对对应多个后端
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/202301201625299.png)
而LLVM架构通过引入`LLVM IR`(Intermediate Representation)解决了这一问题，形成的LLVM架构如下图所示
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/202301201629881.png)

### clang/clang++
是LLVM项目中的一个子项目，是基于LLVM架构的轻量级编译器，属于整个LLVM架构中的[编译器前端](https://www.cnblogs.com/G-H-Y/p/17061801.html)(由LLVM架构图可得知)
创造目的是为了替代GCC，提供更快的编译速度

### make
`make`工具可以看成是一个智能的批处理工具，它本身并没有编译和链接的功能，而是用类似于批处理的方式—通过调用makefile文件中用户指定的命令利用`gcc(或g++)`来进行编译和链接。当程序只有一个源文件时，可以直接使用用`gcc(或g++)`命令进行编译。但当程序包含多个源文件时，逐文件去编译，编译顺序可能出现混乱同时工作量较大

### cmake
makefile在一些简单的工程中可以人工书写，但当工程较大时，手写makefile较为麻烦，同时更换平台需要修改makefile,cmake工具可以根据CMakeLists.txt文件去生成makefile，过程如下图所示
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/20211102161633.png)

More details can be found in [CMake Knowledges Summary](https://gaohongy.github.io/blog/posts/compile_link/cmake/)

> **参考**
> - [1] [GNU的发展史](https://blog.csdn.net/qq_43617936/article/details/104504992)
> - [2] [GCC的gcc和g++区别](https://www.cnblogs.com/samewang/p/4774180.html)
> - [3] [编译器 cc、gcc、g++、CC 的区别](https://www.linuxidc.com/Linux/2019-01/156200.htm)
> - [4] [Linux环境中gcc和g++的区别详解](https://www.linuxidc.com/Linux/2018-10/155027.htm)
> - [5] [GCC、LLVM、Clang区别](https://www.cnblogs.com/Fingerprint/p/11249709.html)
> - [6] [业界主流3大编译器](https://www.cnblogs.com/findumars/p/14213309.html)
> - [7] [区分gnu的gcc/g++, mingw/msvc, llvm的clang/clang++, make,cmake](https://zhuanlan.zhihu.com/p/448884264)
> - [8] [LLVM架构](https://b23.tv/pdhEbrN)([相关资料](https://github.com/chenzomi12/DeepLearningSystem/tree/main/Compiler/AICompiler))
> - [9] [CMake入门](https://www.bilibili.com/video/BV1bg411p7oS/?vd_source=98d46c524d240bd89f118ad90be17aef)

## 编译流程
> 以gcc为例
> 集成开发环境一键式完成的过程，将编译和链接进行合并，此过程称为构建（Build）

![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/202301191738098.png)

> 关于编译选项的详细解释见 [GCC online documentation](https://gcc.gnu.org/onlinedocs/) -> [GCC Manual](https://gcc.gnu.org/onlinedocs/gcc-13.2.0/gcc/) -> [GCC Command Options](https://gcc.gnu.org/onlinedocs/gcc-13.2.0/gcc/#toc-GCC-Command-Options)

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202308241020458.png)

### 预处理(预编译)
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202311090842630.png)

> - 头文件包含: 处理 `#include`
> - [条件编译](https://baike.baidu.com/item/%E6%9D%A1%E4%BB%B6%E7%BC%96%E8%AF%91): 处理 `#if`, `#else`, `#endif` 等等条件编译指令
> - 宏替换: 处理#define, 将宏展开
> - 删除注释

```
cpp hello.c > hello.i
gcc -E hello.c -o hello.i
```
若要检查宏定义或头文件包含是否正确时，可查看预编译后的文件

使用`file`命令可以查看预处理后文件类型如下：
> main.i: C source, ASCII text

### 编译
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202311090844783.png)

> 词法，语法，语义分析，生成汇编代码

```
gcc -S hello.i -o hello.s
gcc -S hello.c -o hello.s
```

使用`file`命令可以查看编译后文件类型如下：
> main.s: assembler source, ASCII text

### 汇编
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202311090845168.png)

> 将汇编语言转化为相应的机器语言(二进制目标文件)

```
as hello.s -o hello.o
gcc -c hello.s -o hello.o
gcc -c hello.c -o hello.o
```
使用`file`命令可以查看汇编后文件类型如下：
> main.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped

注：关于"not stripped"，和`strip`命令有关，表示没有Removes symbols and sections of files.

### 链接
> 将多个可重定位目标文件和标准库函数合并为可执行目标文件

在链接之前，各个程序模块都是相互独立的，模块A所使用到的模块B的内容，在模块A的视角下仅仅是一个符号，并不清楚其具体内容。链接过程可以理解为把模块B的内容结合到A中。整个过程类似搭积木最后的模块拼接过程。

**说明**

Windows 下的链接过程涉及的知识和 Linux 下并不完全相同，以下内容仅面向 Linux 下的链接过程

#### 相关概念解析
1. ELF格式文件分类

|  文件类型    |   说明   |  实例    |
| ---- | ---- | ---- |
|  可重定位文件(Relocatable File)    |      |   Linux的.o (对应Windows的.obj)   |
|   共享目标文件(Shared Object File)   |      |   Linux的.so (对应Windows的.dll)   |
|   可执行文件(Executable File)   |      |   Linux的/bin目录下的程序 (对应Windows的.exe)   |
|   核心转储文件(Core Dump File)   |   当进程意外终止时，系统将进程的地址空间的内容及终止时的信息转储到该文件中   |   Linux的core dump   |

> Note: The **ELF** (Executable and Linkable Format) file format corresponds to **Linux**, the **PE** (Portable Executable) file format corresponds to **Windows** and the **Mach-O** (Mach Object) corresponds to **macOS**.

2. library分类

- 静态链接库：static library, 一种文件归档(archive). relocatable Files + 索引(index) -> static library

> 静态链接库(libadd.a)的文件格式:
> 
> libadd.a: current ar archive

- 动态链接库：shared library(Linux) / dynamic link library(Windows)

> 动态链接库(libadd.so)的文件格式：
> 
> libadd.so: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, BuildID[sha1]=644e95c7d3f9bd18796622c3041e7653e402d179, not stripped

从本质上来说就是以上这两种，但是如果采用的链接方式是显示运行时链接，那么利用到的library又被称作dynamic loading library，使用的仍然是xxx.so文件，只是换了一种使用方式

3. 链接方式

- 静态链接：使用`-static`选项，仅使用静态链接器，在编译链接过程中就将其他模块装入可执行文件中

> 静态链接生成的可执行目标文件的文件格式：
> 
> main: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, for GNU/Linux 3.2.0, BuildID[sha1]=65422291d167d002123191a5f63d9a5503d6d670, not stripped

- 动态链接：默认的链接方式, 同时使用静态链接器和动态链接器，动态链接器在**运行前**将共享模块装载进内存并进行重定位操作

> 动态链接生成的可执行目标文件的文件格式：
> 
> main: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=3e226365a1d11bf52e8c7f5b4b9a72bbecbd7007, not stripped

- 显示运行时链接(Explicit Run-time Linking): 通过某些机制在运行时将共享模块装载进内存并进行重定位操作，让程序在运行时加载或卸载共享模块

**关于library种类和链接方式之间的关系，容易产生的一个误解**

静态链接库，动态链接库 和 静态链接，动态链接，这些名称很容易带给人一种误解: 采用静态链接库时采用的就是静态链接，使用动态链接库时采用的就是动态链接。

首先从链接方式的分类来说，静态还是动态链接是由编译选项`-static`决定的，并不是采用了静态链接库还是动态链接库决定的。

但是它们之间并不是没有关系，从另一个角度来说，根据GNU文档[Options for Linking - static](https://gcc.gnu.org/onlinedocs/gcc/Link-Options.html#index-static)对于static选项的描述
> prevents linking with the shared libraries

可以看出，当我们使用static选项进行静态链接时，使用的只能是静态链接库。

所以，关于库和链接方式的对应关系，需要从两个方向进行讲述：

- 使用静态链接库时，进行的可以是静态链接 或 动态链接； 使用动态链接库，只能进行动态链接
- 进行静态链接时，只能使用静态链接库；进行动态链接时，可以使用静态链接库或动态链接库

i.e. **使用静态链接库是进行静态链接的必要不充分条件, 使用动态链接库是进行动态链接的充分不必要条件**

> 注：后半句的表述并不是非常准确，因为在现行系统下，进行动态链接都会利用到一个特殊的动态链接库-动态链接器，同时C/C++程序无论是否调用一些函数，都会链接libc.so(这一点判断还没有寻找确切的证据，仅是根据实际测试结果)，从这个角度来说，使用动态链接库是进行动态链接的充要条件，而上面的表述仅仅是为了表述库和链接方式的那2点对应关系，所以给出的是充分不必要条件

重新分析上述给出的那种误解，采用静态链接库时，如果没有给出`-static`选项，那么这时候进行的是动态链接，这正验证了使用静态链接库对于进行静态链接的不充分性

除了上述提出的各类问题，还会涉及到一个比较隐含的问题。上面提到使用静态链接库可以进行静态链接，但是在注中又提到一个事实，即虽然我们人为仅仅指定了一些静态链接库，但是背后会隐含利用一些特殊的动态链接库。动态链接库是不可以进行静态链接的，但是为何在进行静态链接时，这些特殊的动态链接库并没有报错。

事实上，这些动态链接库，都存在着与之对应的静态链接库，在我们添加`-static`选项后，链接使用的就是这些静态链接库。e.g. libc.so就存在一个libc.a的静态链接库（可通过命令`find /usr/lib /usr/local/lib -name "libc.a"`查找到）

4. 链接到底在完成一件什么事情？

简单来说就是重定向。

- 静态链接 -> 静态地址重定位， 地址在编译时就已经确定
- 动态链接 -> 装载时地址重定位，地址在编译时是相对地址，具体的绝对地址在加载时由装载器（Loader）进行计算和修改
- 显示运行时链接 -> 运行时地址重定位

#### 静态链接库
静态链接是可执行目标文件在构建过程中完成的，使用链接器将多个.o可重定位目标文件结合（实际上也可以将.so动态链接库结合进来，在动态链接部分详细说明），生成可执行目标文件。

静态链接库：Windows平台.lib (library)，Linux平台.a (archive)

假设编写一个包含加法运算的静态链接库，供main函数调用
```
// add.cpp
int fun(int a, int b) {
     return a + b;
}
```

```
// main.cpp
#include <iostream>

int fun(int, int);

int main() {
     std::cout << fun(1, 2) << std::endl;
     return 0;
}
```

使用`ar`命令将汇编过程生成的.o可重定位目标文件生成静态链接库
```
g++ -c add.cpp -o add.o
ar -rsv libadd.a add.o
```
> `ar -rsv`: 以输出较多信息的方式，完成下述任务：创建归档文件的同时，创建归档索引

使用`file`命令可以查看生成的文件类型如下：
> libadd.a: current ar archive

在链接环节链接该静态链接库（假设libadd.o和main.cpp在同一路径下)
```
g++ main.cpp -L. -ladd
```
以上就是一个创建及使用静态链接库的全流程。
值得注意的是Linux下静态链接库的本质，查询一下`ar`命令会发现，其不过是一个创建归档文件的命令，和目前的`tar`作用是类似的。
因此，所谓静态链接库不过是把一些.o可重定位目标文件集中起来放置到一个文件中，以便链接环节将各模块结合在一起。
不过`tar`和`ar`创建的归档格式并不相同，仅`ar`才能用于创建静态链接库。

为何要用.a这种归档格式作为静态链接库？
把这些零散的目标文件直接提供给库的使用者，很大程度上会造成文件传输、管理和组织方面的不便，于是通常人们使用“ar”压缩程序将这些目标文件压缩到一起，并且对其进行编号和索引，以便于查找和检索，就形成了.a这种归档格式的静态库文件


`tar`创建的归档格式如下：
> libadd.a: POSIX tar archive (GNU)

静态链接库在构建过程中的参与情况示意图：

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202311091009376.png)

> 注：从图中也可以看出.a文件本身只是.o文件的一个容器，实际参与链接过程的仍然是.o可重定位目标文件

**静态链接的缺点**
静态链接是将所需的所有库文件内容进行整合，全部装入到可执行文件中，这种方式会带来以下两种问题：
1. 内存和磁盘空间浪费严重：共用相同库文件的，不同的可执行程序中都存在着相同的库文件，如下图所示

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202401011609985.png)

2. 程序更新不便：更新可执行程序所用的其中一个库文件需要重新下载整个可执行程序

#### 动态链接库
**动态链接的动态指的是哪个环节是动态的？/ 什么是动态链接库？**

所谓动态链接，这里有两种不同的描述，在《程序员的自我修养-链接、装载与库》中的说法大概描述是：不在编译链接环节对那些组成程序的目标文件进行链接，而是等到程序运行时才进行链接“。但是根据下面的动态链接流程图，动态链接大致可以理解为：动态链接 = 编译链接环节的部分链接 + 程序运行时的完全链接。

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202401011657893.png)

我目前对这一点的理解是，编译链接环节的输出结果只是保存了程序需要使用到哪些库（也就是`ldd`能够查询到的那些），然后在程序运行过程中由动态链接器来完成实际的链接过程。所以说以上两种说法其实都没什么问题，真实链接的过程确实是在运行阶段由动态链接器完成的，但是编译链接环节确实也使用到链接器经历了一次链接环节，所以说成部分链接倒是也合理

动态链接库：Windows平台动态链接库.dll (dynamic link library)，Linux平台共享对象文件.so (shared object file)

仍然采用静态链接库的场景，构建动态链接库。
```
// add.cpp
int fun(int a, int b) {
     return a + b;
}
```

```
// main.cpp
#include <iostream>

int fun(int, int);

int main() {
     std::cout << fun(1, 2) << std::endl;
     return 0;
}
```

创建动态链接库：
与创建静态链接库不同，由于静态链接库本质上就是可执行重定位文件的一个归档，因此必须首先生成.o。
动态链接库似乎经历了完整的构建流程，所以是对add.cpp还是对add.o都是可以的
```
g++ add.cpp -fpic -shared -o libadd.so
```

在链接环节链接该动态链接库（假设libadd.so和main.cpp在同一路径下)
```
g++ main.cpp -L. -ladd
```

##### 动态链接包含2个链接过程

使用动态链接库和静态链接库的一个显著区别在于：使用静态链接库时，程序构建完成了就可以直接执行了，但是使用动态链接库，程序构建完成并不一定表示可以正常执行。
In other words, 程序构建和程序执行是两个显著分离的过程。

在静态链接库的链接过程中，使用的的命令是g++ main.cpp -L. -ladd。
同样的，在使用动态链接库时也一样可以使用这一条命令，程序可以正常构建。但是一旦执行程序，会报以下错误：
> ./main: error while loading shared libraries: libadd.so: cannot open shared object file: No such file or directory

通过`ldd`命令(MacOS下可以使用`otool -L`)检查可执行目标文件所需的动态链接库：

> linux-vdso.so.1 (0x00007ffe113c2000) 
> 
> **libadd.so => not found**
> 
> libstdc++.so.6 => /usr/lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f4aa5de5000)
> 
> libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f4aa59f4000)
> 
> libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f4aa5656000)
> 
> /lib64/ld-linux-x86-64.so.2 (0x00007f4aa6370000)
>
> libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f4aa543e000)

可以发现，程序找不到libadd.so这一动态链接库。

##### 在编译时已经通过`-L.`指定了库的搜索路径，为何此时仍然找不到?

We mentioned above that using dynamic libraries is divided into two stages: the first is using static linker, the second is using dynamic linker.

The compile command option `-L. -ladd` helps the static linker to find the libraries file, but it doesn't work for dynamic linker. 

So, the first linking stage can complete successfully and the second linking stage will encounter "No such file or directory" error.

##### 动态链接器如何查找到所需的库文件?[^dynamic_linker_search_way_of_libraries]

[^dynamic_linker_search_way_of_libraries]: [Linux / Unix Command: ld.so](https://en.wikipedia.org/wiki/Rpath#cite_note-1:~:text=The%20dynamic%20linker%20of%20the%20GNU%20C%20Library%20searches%20for%20shared%20libraries%20in%20the%20following%20locations%20in%20order%3A)

动态链接器搜索库文件的顺序如下：

**1. 使用指定的路径名**

对于最开始引入的动态链接库的应用示例，修改编译命令`g++ main.cpp ./libadd.so -o main`，此时在和libadd.so相同路径下即可正常执行可执行目标文件。通过`ldd`命令可以发现内容有所改变

> linux-vdso.so.1 (0x00007fff8a570000)
> 
> **./libadd_dynamic.so (0x00007f5421cea000)**
> 
> libstdc++.so.6 => /usr/lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f5421961000)
> 
> libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f5421570000)
> 
> libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f54211d2000)
> 
> /lib64/ld-linux-x86-64.so.2 (0x00007f54220ee000)
> 
> libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f5420fba000)

关于这种方式的实现机理，通过 `readelf -d main` 命令可以查询到 ELF 文件的 .dynamic section 内容， 同 `g++ main.cpp -o main` 编译得到的文件的 `readelf` 命令的执行结果的关键差别如下所示：

```bash
# g++ main.cpp -o main 结果
Tag        Type                         Name/Value
0x0000000000000001 (NEEDED)             Shared library: [libadd.so]

# g++ main.cpp ./libadd.so -o main 
Tag        Type                         Name/Value
0x0000000000000001 (NEEDED)             Shared library: [./libadd.so]
```

**2. 使用 `DT_RPATH` 中指定的目录**

We can use compile option `-Wl,-rpath,/custom/rpath/` to specify the path in the executable file for the dynamic linker.

It is similar to the first method, because it affects the dynamic linker also with the help of ELF file format's .dynamic section. We can use the command `readelf -d <binary_name> | grep 'R.*PATH'` to verify it.

**3. 使用环境变量 `LD_LIBRARY_PATH`**

修改环境变量`LD_LIBRARY_PATH`（macOS下为`DYLD_LIBRARY_PATH`），将libadd.so所在路径添加到环境变量中

> 需要注意的一点是，根据[Comparison of Shell Script Execution Modes](https://gaohongy.github.io/blog/posts/linux/comparison-of-shell-script-execution-modes/)的知识，可执行文件的执行会在子进程中进行，因此此处的变量需要设置为[Environment/Global Variables](https://gaohongy.github.io/blog/posts/linux/variables-in-linux/)以便子进程可以访问到。

**4. 使用 `DT_RUNPATH` 中指定的目录**

这种方式和滴2种方法有着一定联系，暂时没有遇到相关应用场景，暂时按下不表待后续完善

如果二进制文件中存在 DT_RUNPATH 动态段属性，则动态链接器会搜索这些目录。这些目录仅用于查找 DT_NEEDED（直接依赖项）条目所需的对象，不适用于这些对象的子对象，这些子对象必须自己有自己的 DT_RUNPATH 条目。这与 DT_RPATH 不同，后者适用于依赖树中所有子对象的搜索。

**5. 从缓存文件 `/etc/ld.so.cache` 中搜索**

按照目前理解，如果期望使用这种方式，需要首先将期望被搜索的路径添加到 `/etc/ld.so.conf` 文件中，然后通过 `sudo ldconfig` 构建出新的 `/etc/ld.so.cache`，然后 dynamic linker 就会根据 `ld.so.cache` 来进行搜索

由于未涉及到相关应用常见，此方法还未经测试，等待后续测试完善

**6. 在默认路径中搜索**

默认路径包括 /lib 和 /usr/lib（某些 64 位架构中，默认路径为 /lib64 和 /usr/lib64）

在上述示例中，把生成的libadd.so移动到/usr/local/lib等默认搜索路径（根据现有理解，`make install`所做的工作就是将相关文件复制到这些默认的搜索路径当中，但是个人并不是很推荐这种做法，因为直接采用默认搜索路径就类似黑盒，在不同设备中默认设置并不一定相同，显式给出各种信息会使得编译链接过程更加清晰明了）


##### 动态链接库在构建和执行过程中的参与情况示意图

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202401011657893.png)

> 注：从图上也可以验证上述的说法，相较于静态链接库，动态链接库在构建过程和执行过程中都会发挥作用
> 
> 静态链接器和动态链接器都分别完成了哪些工作 / 从动态链接的流程图中可以看出，xxx.so文件同时参与静态链接库和动态链接库两个链接过程，在这两个过程中这个library分别起到了什么作用?
>
> 答：假设程序P.cpp使用到了一个其他库中定义的函数fun()。当程序P.cpp被编译为P.o之后，编译器是不知道fun()函数的地址的。如果fun()是static library中的函数，那么静态链接器会根据所用的static library，直接将P.o中的fun()函数的地址进行重定位。如果fun()是shared library中的，那么静态链接器会将其标记为动态链接的符号，等到装载时由动态链接器完成重定位。可见，xxx.so需要被用到两次

## Reference
> - [1] [Clang vs Other Open Source Compilers](https://opensource.apple.com/source/clang/clang-23/clang/tools/clang/www/comparison.html)
> - [2] [LLVM整体设计](https://www.bilibili.com/video/BV18j411B7TF/?spm_id_from=333.788.recommend_more_video.1&vd_source=98d46c524d240bd89f118ad90be17aef)
> - [3] [GCC vs. Clang/LLVM: An In-Depth Comparison of C/C++ Compilers](https://alibabatech.medium.com/gcc-vs-clang-llvm-an-in-depth-comparison-of-c-c-compilers-899ede2be378)
> - [4] [gcc Optimize Options](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html)
> - [5] [Linux下编译Opencv](https://blog.csdn.net/weixin_44966641/article/details/120659285)([备份](https://www.cnblogs.com/hongyugao/p/17822192.html))
> - [6] [ELF文件格式解析](https://blog.csdn.net/weixin_44966641/article/details/120631079?spm=1001.2014.3001.5501)([备份](https://www.cnblogs.com/hongyugao/p/17822204.html))
> - [7] [How to create and use program libraries on Linux](https://tldp.org/HOWTO/Program-Library-HOWTO/index.html)
> - [8] [一文读懂Linux下动态链接库版本管理及查找加载方式](https://blog.ideawand.com/2020/02/15/how-does-linux-shared-library-versioning-works/)