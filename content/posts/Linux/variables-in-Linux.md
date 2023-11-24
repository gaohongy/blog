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

The following picture summaries the usages of all commands mentioned above well.
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202311242004665.png)
> Note: The red words represent the command, the dotted lines represent the access to the variables.

Since we have learned about the usages and meanings of all these commands, so let us try to understand the difference between environment variables and shell variables with the picture.

How can we verify the sentence "environment variables can be interviewed by child processes but Shell variables can only be interviewed by current process" ?

Referring to the [Comparison of Shell Script Execution Modes](https://gaohongy.github.io/blog/posts/linux/comparison-of-shell-script-execution-modes/#%E6%98%AF%E5%90%A6%E4%BC%9A%E5%88%9B%E5%BB%BA%E5%AD%90%E8%BF%9B%E7%A8%8B), we can use shell script to verify it.

```bash
# a.sh
name="file a"
bash b.sh

# b.sh
echo "b out: $(set | grep name)"

# result
% bash a.sh

```

According to the [Comparison of Shell Script Execution Modes](https://gaohongy.github.io/blog/posts/linux/comparison-of-shell-script-execution-modes/#%E6%98%AF%E5%90%A6%E4%BC%9A%E5%88%9B%E5%BB%BA%E5%AD%90%E8%BF%9B%E7%A8%8B), becauce a.sh use the `bash`, so a.sh is in the parent process and b.sh is in the child process and the name variable is just a shell variable which is a private data in process where a.sh locates in. So b.sh can't access this variable.

```bash
# a.sh
name="file a"
export name
bash b.sh

# b.sh
echo "b out: $(set | grep name)"

# result
% bash a.sh
b out: name='file a'
```

Because a.sh use the `export`, so the name variable becomes to a environment variable, althought b.sh is in the child progress, it can access this variable.

## Reference
> - [1] [The Bash environment | The Linux Documentation Project](https://tldp.org/LDP/Bash-Beginners-Guide/html/sect_03_02.html)
> - [2] [第十一章、认识与学习 BASH | 鸟哥的Linux私房菜](http://cn.linux.vbird.org/linux_basic/0320bash_2.php)