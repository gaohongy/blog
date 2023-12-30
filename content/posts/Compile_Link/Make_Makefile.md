---
title: Make_Makefile
subtitle:
description:
keywords:
summary:
license:
date: 2023-09-16T17:49:00+08:00
lastmod: 2023-12-30T22:30:32+08:00
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

## Wildcard
The wildcard in makefile is similar with macro in C/C++, it isn't similar with wildcard in linux shell, so it doesn't expend automatically.
```
object1 = *.c  // *.c

object2 = $(wildcard *.cpp) // main.cpp t1.cpp t2.cpp
```

## Automatically generate dependencies
Utilizing the `-M` and `-MM` options to get the dependencies of source code from gcc/g++.
> These options' document can be found at [Preprocessor-Options](https://gcc.gnu.org/onlinedocs/gcc-12.3.0/gcc/Preprocessor-Options.html). (About why these option in the preprocessor options, it is easy to understand. In the state of preprocessor, the preprocessor will judge the header file, so it knows which files are dependented by current file.)

##  Hide commands themself
Place  `@` front of the command, e.g. 
If you use `echo "compiling..."` in makefile, you will get the output
```
echo "compiling..."
compiling...
```

If you use `@echo "compiling..."`, you will get the output
```
compiling...
``` 

## Function
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202309171043153.png)

## Automation variables
- `$@` : 表示规则中的目标文件集
- `$%` : 仅当目标是函数库文件中，表示规则中的目标成员名
- `$<` : 依赖目标中的第一个目标名字
- `$?` : 所有比目标新的依赖目标的集合
- `$^` : 所有的依赖目标的集合
- `$+` : 这个变量很像 $^ ，也是所有依赖目标的集合。只是它不去除重复的依赖目标

## Example
<details>
<summary>Advanced makefile usage example code</summary>

```
# 编译工具信息
CXX = g++
CFLAGS = -std=c++11 -Wall

# 目录信息
SRC_DIR = src
BUILD_DIR = build

# 源文件列表
SRCS = $(wildcard $(SRC_DIR)/*.cpp)

# 生成目标文件列表(目标文件.o和源文件.cpp一一对应)
OBJS = $(patsubst $(SRC_DIR)/%.cpp,$(BUILD_DIR)/%.o,$(SRCS))

# 最终目标文件
TARGET = $(BUILD_DIR)/main

.PHONY: all clean

all: $(TARGET)

$(TARGET): $(OBJS)
	@mkdir -p $(dir $@) # $(dir $@) = $(BUILD_DIR)
	@$(CXX) $(CFLAGS) $^ -o $@

$(BUILD_DIR)/%.o: $(SRC_DIR)/%.cpp
	@mkdir -p $(dir $@)
	@$(CXX) $(CFLAGS) -c $< -o $@

clean:
	rm -rf $(BUILD_DIR)
```
</details>

The above makefile can judge the source code which locates in src directory, and output the object file into the build directory. The following image shows the project file structure.
```
% tree
.
├── Makefile
└── src
    ├── funcs.cpp
    └── main.cpp

2 directories, 3 files
```
After compiling, we will get the following file structure.
```
% tree
.
├── Makefile
├── build
│   ├── funcs.o
│   ├── main
│   └── main.o
└── src
    ├── funcs.cpp
    └── main.cpp

3 directories, 6 files
```
There are some noteworthy points. 
1. `$^` and `$<`
Using the same file structure and editing the makefile, we can understand the difference between these two automation variables.
```
# 编译工具信息
CXX = g++
CFLAGS = -std=c++11 -Wall

# 目录信息
SRC_DIR = src
BUILD_DIR = build

# 源文件列表
SRCS = $(wildcard $(SRC_DIR)/*.cpp)

# 生成目标文件列表(目标文件.o和源文件.cpp一一对应)
OBJS = $(patsubst $(SRC_DIR)/%.cpp,$(BUILD_DIR)/%.o,$(SRCS))

# 最终目标文件
TARGET = $(BUILD_DIR)/main

.PHONY: all clean

all: $(TARGET)

$(TARGET): $(OBJS)
	@echo "\n"
	@echo $^

$(BUILD_DIR)/%.o: $(SRC_DIR)/%.cpp
	@ echo $<

clean:
	rm -rf $(BUILD_DIR)
```

After we excute the `make` command, we will get the result.
```
src/funcs.cpp
src/main.cpp


build/funcs.o build/main.o
```
Based on the project file structure, we can understand the difference between them.

## Pattern-specific Variable Values
The most obvious and easy example is distinguishing the difference between `$<` and `$^` in Automation variables.
i.e. the difference between `$(BUILD_DIR)/%.o: $(SRC_DIR)/%.cpp` and `$(OBJS): $(SRCS)`

Although we know the `$<` is the name of the first prerequisite and `$^` is the names of all the prerequisites, if we don't know the difference of the above expression of `target: prerequisites`, we can't understand the result of the `g++ $(CFLAGS) -c $< -o $@`.

In short, if we still use the above file structure, when using `$(OBJS): $(SRCS)`, the `$(OBJS)` is the 'build/funcs.o build/main.o`, they are one expression, but using 
`$(BUILD_DIR)/%.o: $(SRC_DIR)/%.cpp`, the `$(BUILD_DIR)/%.o` is `build/funcs.o` and `build/main.o`, they are two expressions, it looks like a enumation, so we can use this to generate the corresponding .o object file and .c source file.

## Reference
> - [1] [GNU Make Manual](https://www.gnu.org/software/make/manual/)
> - [2] [What does 'make install' do?](https://superuser.com/questions/360178/what-does-make-install-do)