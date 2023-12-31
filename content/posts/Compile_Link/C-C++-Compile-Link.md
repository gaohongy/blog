---
title: C C++ Compile Link
subtitle:
description:
keywords:
summary:
license:
date: 2023-12-31T21:54:31+08:00
lastmod: 2023-12-31T23:15:17+08:00
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
> 关于编译选项的详细解释见 [GCC online documentation](https://gcc.gnu.org/onlinedocs/) -> GCC <version number> Manual -> GCC Command Options

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

### 链接[^link]
> 将多个可重定位目标文件和标准库函数合并为可执行目标文件

[^link]: 详细分析见[程序编译的链接过程](https://www.cnblogs.com/G-H-Y/p/17077224.html)

在链接之前，各个程序模块都是相互独立的，模块A所使用到的模块B的内容，在模块A的视角下仅仅是一个符号，并不清楚其具体内容。链接过程可以理解为把模块B的内容结合到A中。整个过程类似搭积木最后的模块拼接过程。

#### 易错点
1. 链接静态链接库 != 静态链接生成的可执行目标文件
2. 链接动态链接库 != 动态链接生成的可执行目标文件

- 静态链接库(libadd.a)的文件格式:
> libadd.a: current ar archive

- 动态链接库(libadd.so)的文件格式：
> libadd.so: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, BuildID[sha1]=644e95c7d3f9bd18796622c3041e7653e402d179, not stripped

- 静态链接生成的可执行目标文件的文件格式：
> main: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, for GNU/Linux 3.2.0, BuildID[sha1]=65422291d167d002123191a5f63d9a5503d6d670, not stripped

- 动态链接生成的可执行目标文件的文件格式：
> main: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=3e226365a1d11bf52e8c7f5b4b9a72bbecbd7007, not stripped

在使用.a时，加或者不加-static，对于.a的处理方式似乎没有发生变化，都是将.a中的内容直接复制到了可执行目标文件中
产生变化的实际是一些隐含使用的.so，即使在程序构建时没有添加任何`-l`，但是通过`ldd`可以查看到它也会使用一些动态链接库
而不加-static，就是正常在运行时使用这些动态链接库，加上-static，就会将使用到的所有库都复制到可执行目标文件中

在实际的测试过程中发现：静态链接库可以静态链接，也可以动态链接。动态链接库只能动态链接，不能静态链接（如果尝试这样做了，那么会得到输出信息：/usr/bin/ld: attempted static link of dynamic object `./libadd.so' collect2: error: ld returned 1 exit status)
但是对于一个不应用任何外部第三方库的（注意说的是第三方库，不代表不使用标准库），在编译时是可以添加-static选项的，但是不使用第三方库不代表不使用标准库，g++会隐含链接很多动态链接库，为何这些动态链接库就在使用-static时就不会出现问题呢？由于牵扯的知识面过广，此问题暂时搁置。

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

#### 动态链接库
动态链接库：Windows平台.dll (dynamic link library)，Linux平台.so (shared object)

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

使用动态链接库和静态链接库的一个显著区别在于：使用静态链接库时，程序构建完成了就可以直接执行了，但是使用动态链接库，程序构建完成并不一定表示可以正常执行。
In other words, 程序构建和程序执行是两个显著分离的过程。

在静态链接库的链接过程中，使用的的命令是g++ main.cpp -L. -ladd。
同样的，在使用动态链接库时也一样可以使用这一条命令，程序可以正常构建。但是一旦执行程序，会报以下错误：
> ./main: error while loading shared libraries: libadd.so: cannot open shared object file: No such file or directory

通过`ldd`命令检查可执行目标文件所需的动态链接库：
        linux-vdso.so.1 (0x00007ffe113c2000)
        **libadd.so => not found**
        libstdc++.so.6 => /usr/lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f4aa5de5000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f4aa59f4000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f4aa5656000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f4aa6370000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f4aa543e000)
可以发现，程序找不到libadd.so这一动态链接库。

在编译时已经通过`-L.`指定了库的搜索路径，为何此时仍然找不到？
原因在于，`-L. -ladd`这是告知链接器ld的内容，只能保证在链接过程中ld可以正确找到库所在目录，正确将库同其他模块进行“拼接”（这个拼接和静态链接库提到的本质上应当是不同的，因为具体的内容实际在运行过程中才会获知，所以似乎只是到了符号层面，具体细节涉及内容过多，暂时不深入）。
也就是说编译选项只能负责确保链接器可以正确找到所需的库，但是运行阶段的执行部件还并不清楚。
想说的就是区分好程序构建和程序执行这两个阶段

解决这个问题目前有几种方法：
1. 把生成的libadd.so移动到/usr/local/lib等默认搜索路径
2. 修改环境变量`LD_LIBRARY_PATH`，将libadd.so所在路径添加到环境变量中
3. 修改编译命令`g++ main.cpp ./libadd.so -o main`，此时在和libadd.so相同路径下即可正常执行可执行目标文件。通过`ldd`命令可以发现内容有所改变
        linux-vdso.so.1 (0x00007fff8a570000)
        **./libadd_dynamic.so (0x00007f5421cea000)**
        libstdc++.so.6 => /usr/lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f5421961000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f5421570000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f54211d2000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f54220ee000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f5420fba000)
> 关于第3点引发出的疑问还需要了解更多的关于动态链接库的知识才能解决，暂时不深入


动态链接库在构建和执行过程中的参与情况示意图：
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202311091147037.png)
> 注：从图上也可以验证上述的说法，相较于静态链接库，动态链接库在构建过程和执行过程中都会发挥作用

## Reference
> - [1] [Clang vs Other Open Source Compilers](https://opensource.apple.com/source/clang/clang-23/clang/tools/clang/www/comparison.html)
> - [2] [LLVM整体设计](https://www.bilibili.com/video/BV18j411B7TF/?spm_id_from=333.788.recommend_more_video.1&vd_source=98d46c524d240bd89f118ad90be17aef)
> - [3] [GCC vs. Clang/LLVM: An In-Depth Comparison of C/C++ Compilers](https://alibabatech.medium.com/gcc-vs-clang-llvm-an-in-depth-comparison-of-c-c-compilers-899ede2be378)
> - [4] [gcc Optimize Options](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html)
> - [5] [Linux下编译Opencv](https://blog.csdn.net/weixin_44966641/article/details/120659285)([备份](https://www.cnblogs.com/hongyugao/p/17822192.html))
> - [6] [ELF文件格式解析](https://blog.csdn.net/weixin_44966641/article/details/120631079?spm=1001.2014.3001.5501)([备份](https://www.cnblogs.com/hongyugao/p/17822204.html))