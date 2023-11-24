---
title: Comparison of Shell Script Execution Modes
subtitle:
description:
keywords:
summary:
license:
date: 2023-11-24T17:45:31+08:00
lastmod: 2023-11-24T17:45:31+08:00
tags:
categories:
  - Linux
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

resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
---

## 方法概览
```bash
% echo "echo 'Hello Script'" > script.sh
```
### 方式1：直接运行可执行文件
```bash
% chmod +x script.sh
% ./script.sh
Hello Script
```
### 方式2：使用命令 sh 或 bash
```bash
% sh script.sh
Hello Script

% bash script.sh
Hello Script
```
### 方式3：使用命令 source 或 .
```bash
% source script.sh
Hello Script

% . ./script.sh
Hello Script
```

## 两种分类方法
### 是否需要执行权限
只有方式1需要执行权限。这是因为方式1把脚本作为可执行文件，自然需要执行权限，但方式2和方式3都是把脚本作为命令的参数，可以不具备执行权限

### 是否会创建子进程
只有方式3不会创建子进程，会直接在当前shell环境中执行（主要区别在于变量的作用域，父子进程变量的继承关系）

**实验验证**
- 实验原理：子进程能够共享父进程数据，但父进程无法获取子进程设置的数据
- 实验方法：通过两个文件A和B，A负责设置一个变量，由B以不同方式执行A。若A和B处在相同的shell环境中，则B能够访问到A设置的变量，反之则不同。通过查看B能否访问到A设置的变量来确定A的执行方式是否会创建子进程
- 实验流程（以source和bash执行A为例）：
```bash
# 使用source命令执行A，由于source并不会让A在子进程中执行，因此B能够访问到同一进程中的变量

# f_a.sh
var=show

#f_b.sh
source f_a.sh
echo $var

# result
% bash f_b.sh
show
```

由于f_b文件在执行f_a时使用的是`source`，并不会在子进程中执行f_a中的command，f_a和f_b处于同一个进程下，所以f_b可以访问到var

```bash
# 使用sh命令执行A，由于sh会让A在子进程中执行，由于B在父进程中，因此无法访问到变量值

# f_a.sh
var=show

# f_b.sh
bash f_a.sh
echo $var

# result
% bash f_b.sh

```

由于f_b文件在执行f_a时使用的是`bash`，会在子进程中执行f_a中的command，f_b由于在父进程中，所以f_b无法访问到子进程中的变量var

> 注：在此过程中，需要了解单引号和双引号在终端中的以下几点
> - 双引号内的变量会被解析，转义字符会被处理，并且可以嵌套使用单引号
> - 单引号内的变量不会被解析，转义字符会被视为普通字符，并且可以嵌套使用双引号

以上的两个执行已经验证了之前的判断，不过有一个容易被忽视的点，即执行f_b.sh时选择的方式。根据f_b.sh和f_a.sh的执行方式的组合（以`source`和`bash`为例），可以得到下图
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202311241812397.png)
也就是说，参与进来的进程数量最少是1，即当前进程在执行f_b.s时没有创建子进程，f_b.sh在执行f_a.sh时也没有创建子进程；参与进程数量最多是3，即两个执行过程都创建了子进程。
理解到2个执行阶段的情况，这一整个实验的验证原理才算完整。

## 采用何种解释器执行脚本?
在脚本文件的第一行可以添加shebang（也称为hashbang）行，指定要用于解释和执行该脚本的解释器
在方式2中，sh命令和bash命令并不受shebang行的影响，sh命令则使用sh解释器，bash命令使用bash解释器
在方式1和3中，添加了shebang行则按照shebang指定的解释器执行，若未添加，则使用系统默认的解释器（根据环境变量$SHELL）来执行脚本

shebang不仅可以指定使用shell脚本采用的解释器，也可以指定使用python解释器，例如
```
#!/usr/local/bin/python
```
这样的问题在于不具备可移植性，当环境发生改变时，python的路径可能会发生改变，此时解释器的路径就不再奏效
一种解决方案是采用`env`命令，将shebang修改为`#!/usr/local/bin/python`（尚未了解shebang的核心机制），能够根据$PATH定位当前环境下的python解释器路径

## 注意事项
1. 假设在当前路径下存在一个`script.sh`，点命令的正确且唯一的执行方式是`. ./script.sh`，其他命令则可以采用`command script.sh`