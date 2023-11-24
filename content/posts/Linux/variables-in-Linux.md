---
title: Variables in Linux
subtitle:
description:
keywords:
summary:
license:
date: 2023-11-24T10:32:30+08:00
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
ruby: true
fraction: true
fontawesome: true

resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
---

## Types of variables
1. Environment/Global Variables 
2. Shell/Local Variables

The difference between these two variables is that environment variables can be interviewed by child processes but Shell variables can only be interviewed by current process.


## Related commands
1. `env`
2. `set`
3. `export`
4. `declare`

### View variables
1. `env` is used to print the **Environment Variables**.
2. `set`/`declare` is used to print the **Shell Variables and Environment**.
3. `echo $<variable name>` is used to print **any single variables**.

### Edit variables
1. `export` is used to transform a Shell Variable to a Environment Variable so that child processes can inhert the environment of parent process.
2. `declare -x` is used to transform a Shell Variable to a Environment Variable, it is like `export` command.

理解访问范围可以结合 shell脚本执行方式对比的文章
Linux下执行export出来的都是declare -x，但是macos就不一样


## Reference
> - [1] [The Bash environment | The Linux Documentation Project](https://tldp.org/LDP/Bash-Beginners-Guide/html/sect_03_02.html)
> - [2] [第十一章、认识与学习 BASH | 鸟哥的Linux私房菜](http://cn.linux.vbird.org/linux_basic/0320bash_2.php)