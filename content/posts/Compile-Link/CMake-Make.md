---
title: CMake Make
subtitle:
description:
keywords:
summary:
license:
date: 2023-06-16T23:23:00+08:00
lastmod: 2024-02-27T21:53:31+08:00
tags:
categories:
  - Compile-Link
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

## 说明

cmake的定义是什么 ？-----高级编译配置工具

当多个人用不同的语言或者编译器开发一个项目，最终要输出一个可执行文件或者共享库（dll，so等等）这时候神器就出现了-----CMake！

所有操作都是通过编译CMakeLists.txt来完成的—简单

官方网站是 [www.cmake.org](http://www.cmake.org/)，可以通过访问官方网站获得更多关于 cmake 的信息

学习CMake的目的，为将来处理大型的C/C++/JAVA项目做准备

## CMake安装

1、绝大多数的linux系统已经安装了CMake

2、Windows或某些没有安装过的linux系统，去[http://www.cmake.org/HTML/Download.htm](http://www.cmake.org/HTML/Download.html)l  可以下载安装

## CMake一个HelloWord

1、步骤一，写一个HelloWord

```cpp
#main.cpp

#include <iostream>

int main(){
std::cout <<  "hello word" << std::endl;
}
```

2、步骤二，写CMakeLists.txt

```cpp
#CMakeLists.txt

PROJECT (HELLO)

SET(SRC_LIST main.cpp)

MESSAGE(STATUS "This is BINARY dir " ${HELLO_BINARY_DIR})

MESSAGE(STATUS "This is SOURCE dir "${HELLO_SOURCE_DIR})

ADD_EXECUTABLE(hello ${SRC_LIST})
```

3、步骤三、使用cmake，生成makefile文件

```cpp
cmake .

输出：
[root@localhost cmake]# cmake .
CMake Warning (dev) in CMakeLists.txt:
  Syntax Warning in cmake code at

    /root/cmake/CMakeLists.txt:7:37

  Argument not separated from preceding token by whitespace.
This warning is for project developers.  Use -Wno-dev to suppress it.

-- The C compiler identification is GNU 10.2.1
-- The CXX compiler identification is GNU 10.2.1
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- This is BINARY dir /root/cmake
-- This is SOURCE dir /root/cmake
-- Configuring done
-- Generating done
-- Build files have been written to: /root/cmake
```

目录下就生成了这些文件-CMakeFiles, CMakeCache.txt, cmake_install.cmake 等文件，并且生成了Makefile.
现在不需要理会这些文件的作用，以后你也可以不去理会。最关键的是，它自动生成了Makefile.

4、使用make命令编译

```cpp
root@localhost cmake]# make
Scanning dependencies of target hello
[100%] Building CXX object CMakeFiles/hello.dir/main.cpp.o
Linking CXX executable hello
[100%] Built target hello
```

5、最终生成了Hello的可执行程序

## CMake一个HelloWord-的语法介绍

### PROJECT关键字

可以用来指定工程的名字和支持的语言，默认支持所有语言

PROJECT (HELLO)   指定了工程的名字，并且支持所有语言—建议

PROJECT (HELLO CXX)      指定了工程的名字，并且支持语言是C++

PROJECT (HELLO C CXX)      指定了工程的名字，并且支持语言是C和C++

该指定隐式定义了两个CMAKE的变量

<projectname>_BINARY_DIR，本例中是 HELLO_BINARY_DIR

<projectname>_SOURCE_DIR，本例中是 HELLO_SOURCE_DIR

MESSAGE关键字就可以直接使用者两个变量，当前都指向当前的工作目录，后面会讲外部编译

问题：如果改了工程名，这两个变量名也会改变

解决：又定义两个预定义变量：PROJECT_BINARY_DIR和PROJECT_SOURCE_DIR，这两个变量和HELLO_BINARY_DIR，HELLO_SOURCE_DIR是一致的。所以改了工程名也没有关系

### SET关键字

用来显示的指定变量的

SET(SRC_LIST main.cpp)    SRC_LIST变量就包含了main.cpp

也可以 SET(SRC_LIST main.cpp t1.cpp t2.cpp)

### MESSAGE关键字

向终端输出用户自定义的信息

主要包含三种信息：

- SEND_ERROR，产生错误，生成过程被跳过。
- SATUS，输出前缀为—的信息。
- FATAL_ERROR，立即终止所有 cmake 过程.

### ADD_EXECUTABLE关键字

生成可执行文件

ADD_EXECUTABLE(hello ${SRC_LIST})     生成的可执行文件名是hello，源文件读取变量SRC_LIST中的内容

也可以直接写 ADD_EXECUTABLE(hello main.cpp)

上述例子可以简化的写成

PROJECT(HELLO)
ADD_EXECUTABLE(hello main.cpp)

注意：工程名的 HELLO 和生成的可执行文件 hello 是没有任何关系的

## 语法的基本原则

- 变量使用${}方式取值，但是在 IF 控制语句中是直接使用变量名

- 指令(参数 1 参数 2...) 参数使用括弧括起，参数之间使用空格或分号分开。 以上面的 ADD_EXECUTABLE 指令为例，如果存在另外一个 func.cpp 源文件

  就要写成：ADD_EXECUTABLE(hello main.cpp func.cpp)或者ADD_EXECUTABLE(hello main.cpp;func.cpp)

- 指令是大小写无关的，参数和变量是大小写相关的。但，推荐你全部使用大写指令

### 语法注意事项

- SET(SRC_LIST main.cpp) 可以写成 SET(SRC_LIST “main.cpp”)，如果源文件名中含有空格，就必须要加双引号
- ADD_EXECUTABLE(hello main) 后缀可以不行，他会自动去找.c和.cpp，最好不要这样写，可能会有这两个文件main.cpp和main

## 内部构建和外部构建

- 上述例子就是内部构建，他生产的临时文件特别多，不方便清理
- 外部构建，就会把生成的临时文件放在build目录下，不会对源文件有任何影响强烈使用外部构建方式

### 外部构建方式举例

```cpp
//例子目录，CMakeLists.txt和上面例子一致
[root@localhost cmake]# pwd
/root/cmake
[root@localhost cmake]# ll
total 8
-rw-r--r--. 1 root root 198 Dec 28 20:59 CMakeLists.txt
-rw-r--r--. 1 root root  76 Dec 28 00:18 main.cpp
```

1、建立一个build目录，可以在任何地方，建议在当前目录下

2、进入build，运行cmake ..    当然..表示上一级目录，你可以写CMakeLists.txt所在的绝对路径，生产的文件都在build目录下了

3、在build目录下，运行make来构建工程

注意外部构建的两个变量

1、HELLO_SOURCE_DIR  还是工程路径

2、HELLO_BINARY_DIR   编译路径 也就是 /root/cmake/bulid

## 让Hello World看起来更像一个工程

- 为工程添加一个子目录 src，用来放置工程源代码
- 添加一个子目录 doc，用来放置这个工程的文档 hello.txt
- 在工程目录添加文本文件 COPYRIGHT, README
- 在工程目录添加一个 [runhello.sh](http://runhello.sh/) 脚本，用来调用 hello 二进制
- 将构建后的目标文件放入构建目录的 bin 子目录
- 将 doc 目录 的内容以及 COPYRIGHT/README 安装到/usr/share/doc/cmake/

### 将目标文件放入构建目录的 bin 子目录

每个目录下都要有一个CMakeLists.txt说明

```cpp
[root@localhost cmake]# tree
.
├── build
├── CMakeLists.txt
└── src
    ├── CMakeLists.txt
    └── main.cpp
```

外层CMakeLists.txt

```cpp
PROJECT(HELLO)
ADD_SUBDIRECTORY(src bin)
```

src下的CMakeLists.txt

```cpp
ADD_EXECUTABLE(hello main.cpp)
```

#### ADD_SUBDIRECTORY 指令

ADD_SUBDIRECTORY(source_dir [binary_dir] [EXCLUDE_FROM_ALL])

- 这个指令用于向当前工程添加存放源文件的子目录，并可以指定中间二进制和目标二进制存放的位置

- EXCLUDE_FROM_ALL函数是将写的目录从编译中排除，如程序中的example

- ADD_SUBDIRECTORY(src bin)

  将 src 子目录加入工程并指定编译输出(包含编译中间结果)路径为bin 目录

  如果不进行 bin 目录的指定，那么编译结果(包括中间结果)都将存放在build/src 目录

#### 更改二进制的保存路径

SET 指令重新定义 EXECUTABLE_OUTPUT_PATH 和 LIBRARY_OUTPUT_PATH 变量 来指定最终的目标二进制的位置

SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)

思考：加载哪个CMakeLists.txt当中

哪里要改变目标存放路径，就在哪里加入上述的定义，所以应该在src下的CMakeLists.txt下写

## 安装

- 一种是从代码编译后直接 make install 安装
- 一种是打包时的指定 目录安装。
  - 简单的可以这样指定目录：make install DESTDIR=/tmp/test
  - 稍微复杂一点可以这样指定目录：./configure –prefix=/usr

### 如何安装HelloWord

使用CMAKE一个新的指令：INSTALL

INSTALL的安装可以包括：二进制、动态库、静态库以及文件、目录、脚本等

使用CMAKE一个新的变量：CMAKE_INSTALL_PREFIX

```cpp
// 目录树结构
[root@localhost cmake]# tree
.
├── build
├── CMakeLists.txt
├── COPYRIGHT
├── doc
│   └── hello.txt
├── README
├── runhello.sh
└── src
    ├── CMakeLists.txt
    └── main.cpp

3 directories, 7 files
```

#### 安装文件COPYRIGHT和README

INSTALL(FILES COPYRIGHT README DESTINATION share/doc/cmake/)

FILES：文件

DESTINATION：

1、写绝对路径

2、可以写相对路径，相对路径实际路径是：${CMAKE_INSTALL_PREFIX}/<DESTINATION 定义的路径>

CMAKE_INSTALL_PREFIX  默认是在 /usr/local/

cmake -DCMAKE_INSTALL_PREFIX=/usr    在cmake的时候指定CMAKE_INSTALL_PREFIX变量的路径

#### 安装脚本runhello.sh

PROGRAMS：非目标文件的可执行程序安装(比如脚本之类)

INSTALL(PROGRAMS runhello.sh DESTINATION bin)

说明：实际安装到的是 /usr/bin

#### 安装 doc 中的 hello.txt

- 一、是通过在 doc 目录建立CMakeLists.txt ，通过install下的file

- 二、是直接在工程目录通过

  INSTALL(DIRECTORY doc/ DESTINATION share/doc/cmake)


DIRECTORY 后面连接的是所在 Source 目录的相对路径

注意：abc 和 abc/有很大的区别

目录名不以/结尾：这个目录将被安装为目标路径下的

目录名以/结尾：将这个目录中的内容安装到目标路径

#### 安装过程

cmake ..

make

make install

## 静态库和动态库的构建

任务：

１，建立一个静态库和动态库，提供 HelloFunc 函数供其他程序编程使用，HelloFunc 向终端输出 Hello World 字符串。 

２，安装头文件与共享库。

静态库和动态库的区别

- 静态库的扩展名一般为“.a”或“.lib”；动态库的扩展名一般为“.so”或“.dll”。
- 静态库在编译时会直接整合到目标程序中，编译成功的可执行文件可独立运行
- 动态库在编译时不会放到连接的目标程序中，即可执行文件无法单独运行。

### 构建实例

```cpp
[root@localhost cmake2]# tree
.
├── build
├── CMakeLists.txt
└── lib
    ├── CMakeLists.txt
    ├── hello.cpp
    └── hello.h
```

hello.h中的内容

```cpp
#ifndef HELLO_H
#define Hello_H

void HelloFunc();

#endif
```

hello.cpp中的内容

```cpp
#include "hello.h"
#include <iostream>
void HelloFunc(){
    std::cout << "Hello World" << std::endl;
}
```

项目中的cmake内容

```cpp
PROJECT(HELLO)
ADD_SUBDIRECTORY(lib bin)
```

lib中CMakeLists.txt中的内容

```cpp
SET(LIBHELLO_SRC hello.cpp)
ADD_LIBRARY(hello SHARED ${LIBHELLO_SRC})
```

#### ADD_LIBRARY

ADD_LIBRARY(hello SHARED ${LIBHELLO_SRC})

- hello：就是正常的库名，生成的名字前面会加上lib，最终产生的文件是libhello.so
- SHARED，动态库    STATIC，静态库
- ${LIBHELLO_SRC} ：源文件

#### 同时构建静态和动态库

```cpp
// 如果用这种方式，只会构建一个动态库，不会构建出静态库，虽然静态库的后缀是.a
ADD_LIBRARY(hello SHARED ${LIBHELLO_SRC})
ADD_LIBRARY(hello STATIC ${LIBHELLO_SRC})

// 修改静态库的名字，这样是可以的，但是我们往往希望他们的名字是相同的，只是后缀不同而已
ADD_LIBRARY(hello SHARED ${LIBHELLO_SRC})
ADD_LIBRARY(hello_static STATIC ${LIBHELLO_SRC})
```

#### SET_TARGET_PROPERTIES

这条指令可以用来设置输出的名称，对于动态库，还可以用来指定动态库版本和 API 版本

同时构建静态和动态库

```cpp
SET(LIBHELLO_SRC hello.cpp)

ADD_LIBRARY(hello_static STATIC ${LIBHELLO_SRC})

//对hello_static的重名为hello
SET_TARGET_PROPERTIES(hello_static PROPERTIES  OUTPUT_NAME "hello")
//cmake 在构建一个新的target 时，会尝试清理掉其他使用这个名字的库，因为，在构建 libhello.so 时， 就会清理掉 libhello.a
SET_TARGET_PROPERTIES(hello_static PROPERTIES CLEAN_DIRECT_OUTPUT 1)

ADD_LIBRARY(hello SHARED ${LIBHELLO_SRC})

SET_TARGET_PROPERTIES(hello PROPERTIES  OUTPUT_NAME "hello")
SET_TARGET_PROPERTIES(hello PROPERTIES CLEAN_DIRECT_OUTPUT 1)

```

#### 动态库的版本号

一般动态库都有一个版本号的关联

```cpp
libhello.so.1.2
libhello.so ->libhello.so.1
libhello.so.1->libhello.so.1.2
```

CMakeLists.txt 插入如下

`SET_TARGET_PROPERTIES(hello PROPERTIES VERSION 1.2 SOVERSION 1)`

VERSION 指代动态库版本，SOVERSION 指代 API 版本。

#### 安装共享库和头文件

本例中我们将 hello 的共享库安装到<prefix>/lib目录，

将 hello.h 安装到<prefix>/include/hello 目录

```cpp
//文件放到该目录下
INSTALL(FILES hello.h DESTINATION include/hello)

//二进制，静态库，动态库安装都用TARGETS
//ARCHIVE 特指静态库，LIBRARY 特指动态库，RUNTIME 特指可执行目标二进制。
INSTALL(TARGETS hello hello_static LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
```

注意：

安装的时候，指定一下路径，放到系统下

`cmake -DCMAKE_INSTALL_PREFIX=/usr ..`

#### 使用外部共享库和头文件

准备工作，新建一个目录来使用外部共享库和头文件

```cpp
[root@MiWiFi-R4CM-srv cmake3]# tree
.
├── build
├── CMakeLists.txt
└── src
    ├── CMakeLists.txt
    └── main.cpp
```

main.cpp

```cpp
#include <hello.h>

int main(){
	HelloFunc();
}
```

#### 解决：make后头文件找不到的问题

PS：include <hello/hello.h>  这样include是可以，这么做的话，就没啥好讲的了

关键字：INCLUDE_DIRECTORIES    这条指令可以用来向工程添加多个特定的头文件搜索路径，路径之间用空格分割

在CMakeLists.txt中加入头文件搜索路径

INCLUDE_DIRECTORIES(/usr/include/hello)

#### 解决：找到引用的函数问题

报错信息：undefined reference to `HelloFunc()'

关键字：LINK_DIRECTORIES     添加非标准的共享库搜索路径

指定第三方库所在路径，LINK_DIRECTORIES(/home/myproject/libs)

关键字：TARGET_LINK_LIBRARIES    添加需要链接的共享库

TARGET_LINK_LIBRARIES的时候，只需要给出动态链接库的名字就行了。

在CMakeLists.txt中插入链接共享库，主要要插在executable的后面

查看main的链接情况

```cpp
[root@MiWiFi-R4CM-srv bin]# ldd main 
	linux-vdso.so.1 =>  (0x00007ffedfda4000)
	libhello.so => /lib64/libhello.so (0x00007f41c0d8f000)
	libstdc++.so.6 => /lib64/libstdc++.so.6 (0x00007f41c0874000)
	libm.so.6 => /lib64/libm.so.6 (0x00007f41c0572000)
	libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00007f41c035c000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f41bff8e000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f41c0b7c000)
```

链接静态库

`TARGET_LINK_LIBRARIES(main libhello.a)`

#### 特殊的环境变量 CMAKE_INCLUDE_PATH 和 CMAKE_LIBRARY_PATH

注意：这两个是环境变量而不是 cmake 变量，可以在linux的bash中进行设置

我们上面例子中使用了绝对路径INCLUDE_DIRECTORIES(/usr/include/hello)来指明include路径的位置

我们还可以使用另外一种方式，使用环境变量export CMAKE_INCLUDE_PATH=/usr/include/hello

补充：生产debug版本的方法：
cmake .. -DCMAKE_BUILD_TYPE=debug

## 头文件
> 需要首先明确的一点是，一个.h头文件对应一个.cpp文件并非严格要求，只是一种较为规范的写法。由于其他文件需要引用，因此一定是存在.h头文件的，头文件中函数的具体实现可以放置于头文件中，也可以放置于.cpp文件中

### 引入方式
1. `<>`是只在系统目录中搜索
2. `""`是优先在当前目录搜索，未找到则进入系统目录搜索

### Search Path
We can use `g++ -xc++ -E -v -` command to get the processing flow of the preprocess, which contains the search paths of the header file.

> - `-x language` is used to specify explicitly the language for the following input files (rather than letting the compiler choose a default based on the file name suffix). In my opinion, now, we want to get the processing flow of the c++ file's proprocess with no input file, so we must specify the input file type manually.
> - `-E` is the option for prepocessing
> - `-v` is used to output detail information
> - `-` 表示接受来自标准输入的输入。此处，未提供实际的源文件，只关心获取编译器信息。

> Refer to the [official document](https://gcc.gnu.org/onlinedocs/gcc-13.2.0/cpp/Include-Syntax.html) for detailed information.

### 效果
由于include头文件的实际效果是把头文件的内容插入到文件中，因此
1. 在不支持重载机制的c语言中，当头文件中的声明语句和.cpp中实现语句不同时会报错，可以在编译时就发现错误
2. 当.cpp中实现了多个函数，且函数之间存在相互调用时，添加.h文件可以无需关心函数的实现顺序，因为添加了.h头文件，因此会将声明语句复制到.cpp文件中

> 引入可以使用相对路径

## 引入第三方库
cmake不过是一个构建工具，其仍然需要完成[程序的编译链接过程](https://gaohongy.github.io/blog/posts/compile_link/c-c++-compile-link/#%E7%BC%96%E8%AF%91%E6%B5%81%E7%A8%8B)。
因此这里涉及到的所有函数的作用或者意义都会对应着程序的编译链接中的某一个过程。

因为程序的编译链接无非是要解决头文件和链接库的问题，所以下面讨论的这些情况也都离不开这两个问题。
例如纯头文件库就只需要考虑如何引入头文件；下述例子的子模块不过是在提供了一个头文件之外，还生成了一个静态链接库；第三方库则更进一步，在提供头文件和静态链接库之外，还提供了动态链接库，而且由于纯头文件库和子模块往往都是和原项目位于同一目录下的，因此寻找这些头文件和静态链接库往往是容易的，但是第三方库往往是位于系统的其他某个位置，如何找到它提供的头文件和库就是一个需要重点考虑的问题。

### 纯头文件库（header-only）
通过 `include_directories()` 或 `target_include_directories()` 即可直接使用
`target_include_directories()`起到的作用是通过指定头文件的搜索路径，可以把自定义头文件当作系统头文件使用

#### 单文件include头文件不使用`target_include_directories()`示例

![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/202306171954070.png)
> functions模拟的是第三方纯头文件库

<details>
<summary>CMakeLists.txt</summary>

```
cmake_minimum_required(VERSION 3.12)
project(header_only_demo LANGUAGES CXX)

# 构建可执行文件                                                                                                    
add_executable(main main.cpp)
```
</details>

<details>
<summary>func.h</summary>

```
#ifndef FUNC_H
#define FUNC_H

#include <iostream>
void func() {
    std::cout << "call func()" << std::endl;
}

#endif
```
</details>

<details>
<summary>main.cpp</summary>

```
#include "functions/func.h"

int main() {
    func();

    return 0;
}
```
</details>

#### 单文件include头文件使用`target_include_directories()`示例

![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/202306171954070.png)
> functions模拟的是第三方纯头文件库

<details>
<summary>CMakeLists.txt</summary>

```
cmake_minimum_required(VERSION 3.12)
project(header_only_demo LANGUAGES CXX)

# 构建可执行文件                                                                                                    
add_executable(main main.cpp)

# 指定头文件搜索路径
target_include_directories(main PRIVATE ./functions)
```
</details>

<details>
<summary>func.h</summary>

```
#ifndef FUNC_H
#define FUNC_H

#include <iostream>
void func() {
    std::cout << "call func()" << std::endl;
}

#endif
```
</details>

<details>
<summary>main.cpp</summary>

```
#include <func.h>

int main() {
    func();

    return 0;
}
```
</details>

### 子模块
把第三方项目源码置于工程根目录下，通过`add_subdirectory()`和`target_link_libraries()`

![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/202306172347233.png)

<details>
<summary>main.cpp</summary>

```
#include <func.h>

int main() {
    func();

    return 0;
}
```
</details>

<details>
<summary>funclib/func.h</summary>

```
#ifndef FUNC_H
#define FUNC_H

void func();

#endif
```
</details>

<details>
<summary>funclib/func.cpp</summary>

```
#include <func.h>
#include <iostream>

void func() {
    std::cout << "func()" << std::endl;
}
```
</details>

对于最简单的子模块，同header_only相比，子模块不仅提供了.h头文件，还提供了.cpp的实现文件
因此需要在子模块的CMakeLists.txt中通过`add_library()`生成静态或动态库，然后在项目根目录中通过`target_link_libraries()`link项目中的多个子模块。并且，对于期望构建代码的子模块，需要在项目根路径下的CMakeLists.txt中使用`add_subdirectory()`指定该模块。

除此之外，同header_only相比，值得关注的一点是include子模块中的.h问题,通过`target_include_directories()`可以简写include语句,但CMakeLists.txt只能对相同路径下的.cpp文件起到作用,由于main.cpp和funclib/func.cpp中都需要引入func.h,因此一种简单的实现方式是,在项目根路径和子模块下的CMakeLists.txt中都需要添加`target_include_directories()`语句,如以下代码所示

<details>
<summary>CMakeLists.txt</summary>

```
cmake_minimum_required(VERSION 3.12)
project(mul_module_demo LANGUAGES CXX)

add_subdirectory(funclib)

add_executable(main main.cpp)
target_include_directories(main PRIVATE ./funclib)
target_link_libraries(main PUBLIC funclib)
```
</details>


<details>
<summary>funclib/CMakeLists.txt</summary>

```
add_library(funclib STATIC func.cpp)
target_include_directories(funclib PRIVATE .)
```
</details>

这样虽然能够解决问题,但是当子模块数量过多时,项目根路径下的CMakeList.txt文件会和每个子模块的CMakeLists.txt中都存在重复代码,
cmake通过设置`target_include_directories()`语句中的一个参数解决了上述问题
将项目根路径下的CMakeLists.txt中的`target_include_directories()`语句删除,在每个子模块中添加该语句,同时将以上示例代码该语句中的PRIVATE修改为PUBLIC,
PUBLIC表示包含的头文件能够在子模块和link该子模块的父模块之间传播,从而只需要在子模块中添加该语句,父模块依然可以把自定义头文件作为系统头文件使用,实现代码如下所示

<details>
<summary>CMakeLists.txt</summary>

```
cmake_minimum_required(VERSION 3.12)
project(mul_module_demo LANGUAGES CXX)

add_subdirectory(funclib)

add_executable(main main.cpp)
target_link_libraries(main PUBLIC funclib)
```
</details>


<details>
<summary>funclib/CMakeLists.txt</summary>

```
add_library(funclib STATIC func.cpp)
target_include_directories(funclib PUBLIC .)
```
</details>

### 第三方库
通过 `find_package()` 和 `target_link_libraries()`
需要了解包(package)、库和组件的概念，包包含多个库/组件，库和组件的概念是相同的，例如TBB包提供了多个库(组件)

为了便于描述，通过OpenCV的实际例子来解释如何利用cmake来引入第三方库。

在使用OpenCV之前，需要首先进行安装。而OpenCV的安装同样需要使用cmake。参照[OpenCV: Installation in Linux](https://docs.opencv.org/4.x/d7/d9f/tutorial_linux_install.html)官方站点给出的安装流程进行安装。When we finish installing the opencv, we will get [some files](https://pastebin.ubuntu.com/p/X6qZN6kZhJ/).

程序的编译链接过程，无非就是处理头文件和依赖库的问题。使用cmake也是完成程序的编译链接工作，它也需要调用gcc/g++。那么在理论上，通过cmake可以完成的工作，人工操作都能够完成，只不过在一些大型复杂项目中，需要做的工作太多，使用cmake可以简化这个过程。所以我们不妨首先考虑在一个小例子中，人工操作和cmake操作的对应关系，以便推导到大型项目下的cmake是如何处理的。

假设现在需要编译一个RGB图像转灰度图像的单程序。人工操作只需要在编译时候指明头文件的搜索路径，以及所需库的搜索路径及所需要链接的库。

```bash
g++ xxx.cpp -o yyy -I/usr/local/include/opencv4 \
                    -L/usr/local/lib\
                    -lopencv_calib3d \
                    -lopencv_core \
                    -lopencv_dnn \
                    -lopencv_features2d \
                    -lopencv_flann \
                    -lopencv_gapi \
                    -lopencv_highgui \
                    -lopencv_imgcodecs \
                    -lopencv_imgproc \
                    -lopencv_ml \
                    -lopencv_objdetect \
                    -lopencv_photo \
                    -lopencv_stitching \
                    -lopencv_video \
                    -lopencv_videoio
```

使用上述命令即可正常完成编译。但是产生的一个问题就是，指明的头文件的搜索路径还比较简单，但是下方的这些`-l`所要链接的库过多比较麻烦。

cmake就通过以下命令可以解决这个问题

```cmake
find_package( OpenCV REQUIRED )
include_directories(${OpenCV_INCLUDE_DIRS})
target_link_libraries(<project name> ${OpenCV_LIBS})
```

其中`target_include_directories`负责指定头文件搜索路径，也就对应着`-I/usr/local/include/opencv4`；`target_link_libraries`负责指定所要链接的库，对应着上述g++选项中众多的`-l`选项。关于这一点，通过以下两条命令即可验证

```cmake
MESSAGE( STATUS "OpenCV_INCLUDE_DIRS = ${OpenCV_INCLUDE_DIRS}.")
MESSAGE( STATUS "OpenCV_LIBS = ${OpenCV_LIBS}.")

-- OpenCV_INCLUDE_DIRS = /usr/local/include/opencv4.
-- OpenCV_LIBS = opencv_calib3d;opencv_core;opencv_dnn;opencv_features2d;opencv_flann;opencv_gapi;opencv_highgui;opencv_imgcodecs;opencv_imgproc;opencv_ml;opencv_objdetect;opencv_photo;opencv_stitching;opencv_video;opencv_videoio.
```

- **我们虽然理解`target_include_directories`和`target_link_libraries`的作用，但是`${OpenCV_INCLUDE_DIRS}`和`${OpenCV_LIBS}`又是从何而来，`find_package`又起到了什么作用？**

`${OpenCV_INCLUDE_DIRS}`和`${OpenCV_LIBS}`显然是两个环境变量，但是在cmake之前找到环境变量，是没有这两个值的，那么可以猜想`find_package`所做的工作就是设置这两个环境变量，实际也正是如此。

**也就是说`find_package`需要找到第三方库的头文件和库的路径。但是第三方库的数量众多，是如何能够做到这一点的？**

这就涉及到`find_package`搜索文件的两种模式和对应的两种文件：[^find_package-official-documentation]
[^find_package-official-documentation]: [find_package官方文档](https://cmake.org/cmake/help/latest/command/find_package.html)

1. Module mode: Find<LibaryName>.cmake，(CMake官方预定义的一些依赖包所对应的查找文件，所处路径为: `/usr/share/cmake-<version>/Modules`)
2. Config mode: <LibraryName>Config.cmake，(通过CMake编译安装的第三方库，所处路径和构建 CMakeLists.txt 时的选项相关，如果给定了`-DCMAKE_INSTALL_PREFIX`选项，那么就会在该路径下的`lib/cmake`，如果没有给定该选项，那么就会在`/usr/local/lib/cmake`下)

`find_package`根据这两种文件实现环境变量的设置。

## 示例
### 构建流程

A simple but typical use of cmake with a fresh copy of software source code is to create a build directory and invoke cmake there:[^cmake-build-flow]

[^cmake-build-flow]: [Command Line cmake tool](https://cmake.org/cmake/help/latest/guide/user-interaction/index.html#command-line-cmake-tool)

```bash
cd some_software-1.4.2
mkdir build
cd build

cmake .. -DCMAKE_INSTALL_PREFIX=/opt/the/prefix
cmake --build . # It is equivalent to make
cmake --build . --target install # It is equivalent to make install
```

> The role of cmake is just generate Makefile for make, it doesn't compile the project truely. e.g. `cmake .. -DCMAKE_INSTALL_PREFIX=/opt/the/prefix`.
> 
> `cmake --build` or `make` compile the project. This step will invoke some compiler and linker.
> 
> `cmake --build . --target install` or `make install`. A popular understanding of installing is that move the executable file, header files and libraries to the path which is specified by `-DCMAKE_INSTALL_PREFIX` option, if we don't use this option, in general, the default intall prefix is `/usr/local/`. In general, it will generate some directories: **bin**, **include**, **lib** and **share**.

```
// cmake最低版本
cmake_minimum_required(VERSION 3.0.0)

// 工程文件名
project(experiment2 VERSION 0.1.0)

// 寻找第三方库
// c++包管理工具vcpkg(类似pip，使用时再查找)
find_package(库名称 REQUIRED(库是必须的，未安装则报错))

// 匹配所有源文件添加到变量SRC_FILES中
file(GLOB SRC_FILES
    "${PROJECT_SOURCE_DIR}/src/*.h"
    "${PROJECT_SOURCE_DIR}/src/*.cpp"
    "${PROJECT_SOURCE_DIR}/src/*.cc"
    "${PROJECT_SOURCE_DIR}/src/*.c"
)

# 这里只是设置两个变量，并没有指定这个路径
set(INC_DIR  include目录路径) # 设置include路径变量
set(LINK_DIR 库目录路径) # 设置library路径变量

# 指定include和lib路径
include_directories(${INC_DIR})
link_directories(${LINK_DIR})

# 链接静态库，在下一步构建可执行文件之前
# 需要理解的是上一步只是指示了头文件和库的路径，程序还需要显示包含
# 头文件往往都是在程序中再include，lib可以在程序中指明，但一般是直接在这里写上，程序中就不再添加代码了
link_libraries(库名称)

# 构建可执行文件
# ${CMAKE_PROJECT_NAME}：被替换为project指定的名称
# ${SRC_FILES}：file命令找到的所有源文件
add_executable(${CMAKE_PROJECT_NAME} ${SRC_FILES})

# 链接动态库(Link)，需要在上一步构建可执行文件之后
target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE 库名称)

// 支持c++17
target_compile_features(${CMAKE_PROJECT_NAME} PRIVATE cxx_std_17)

// 自动化工作
// 需要使用再详细查找
add_custom_command(
  TARGET ${CMAKE_PROJECT_NAME}
  POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy_directory
    "")
```
### CMakeLists示例
> 此示例用于winPcap开发

```
cmake_minimum_required(VERSION 3.0.0)
project(experiment2 VERSION 0.1.0)

file(GLOB SRC_FILES
    "${PROJECT_SOURCE_DIR}/*.cpp"
)

set(INC_DIR  E:/useful/cmake_project/WpdPack/Include)
set(LINK_DIR E:/useful/cmake_project/WpdPack/Lib/x64)

include_directories(${INC_DIR})
link_directories(${LINK_DIR})

link_libraries(Packet wpcap)

add_executable(${CMAKE_PROJECT_NAME} ${SRC_FILES})
```

## Reference
> - [1] [CMake Reference Documentation](https://cmake.org/cmake/help/latest/)
> - [2] [CMake FAQ](https://gitlab.kitware.com/cmake/community/-/wikis/FAQ)
> - [3] [理解find_package()](https://zhuanlan.zhihu.com/p/97369704)
> - [4] [Cmake中文实战教程](https://brightxiaohan.github.io/CMakeTutorial/)