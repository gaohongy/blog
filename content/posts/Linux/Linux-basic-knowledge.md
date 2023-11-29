---
title: Linux Basic Knowledge
subtitle:
description:
keywords:
summary:
license:
date: 2023-01-18T23:51:41+08:00
lastmod: 2023-11-28T10:00:57+08:00
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

# Terminal 快捷键
- 历史命令：
  - Ctrl P : 上一条命令，可以一直按表示一直往前翻
  - Ctrl N : 下一条命令
  - Ctrl R，再按历史命令中出现过的字符串：按字符串寻找历史命令

- 命令行编辑：
  - Ctrl A ： 移动光标到命令行首
  - Ctrl E : 移动光标到命令行尾
  - Ctrl B : 光标后退
  - Ctrl F : 光标前进
  - Ctrl W : 删除光标前的单词
  - Ctrl K ：删除光标之后所有字符
  - Ctrl U : 清空当前键入的命令


#  用户相关命令
```
1. 创建用户：adduser username
2. 修改用户组: usermod -g groupname username
3. 删除组: groupdel groupname
4. 删除用户: deluser -r username(-r是为了顺带删除用户目录)
5. 强制结束进行: kill -9 processId
```

# 云服务器
## 本地免密登录
![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/202212290918239.png)

# 文件相关

## ls命令各字段分布
在使用ls获取文件的各项信息时，常用的选项包括如下几种：

- `-a`：展示包括隐藏文件在內的所有文件
- `-l`：打印文件的详细信息
- `-h`：以人类可以理解的格式输出文件大小
- `-G`：以彩色文本输出结果

**关于时间信息**
- `-U`：展示文件的创建时间
- `-l`：展示文件最后的修改时间
- `-c`：展示文件状态最后改变时间
- `-u`：用文件最后的访问时间代替原始`-l`展示的最后修改时间

**关于排序**

- `-t`：首先根据时间戳降序排序，当时间戳相同时，按照文件名字典序生序排列
- `-S`：字典序排序前按照文件大小降序佩列

![](https://cdn.jsdelivr.net/gh/G-ghy/cloudImages@master/202307181613610.png)

```
$ ls -l -a    # 也可以把选项合并成ls -la
total 56K
drwxr-xr-x  2 yzh yzh  4096 Sep 11 09:52 .  --> 当前目录
drwxr-xr-x 10 yzh yzh  4096 Sep 10 19:51 .. --> 父目录
-rw-r--r--  1 yzh yzh 34565 Sep  4 10:48 01.md
-rw-r--r--  1 yzh yzh  9314 Sep 11 09:36 02.md
|\./\./\./  | \./ \./ \.../ \........../   +--> 文件名（the pathname）
| |  |  |   |  |   |    |        +------------> 上次修改日期（last modified time）
| |  |  |   |  |   |    +---------------------> 文件大小(字节)（number of bytes in the file）
| |  |  |   |  |   +--------------------------> 文件所属组（group name）
| |  |  |   |  +------------------------------> 文件所属用户（owner name）
| |  |  |   +---------------------------------> 硬链接数量（number of links）
| |  |  +-------------------------------------> 其他用户权限
| |  +----------------------------------------> 组内用户权限
| +-------------------------------------------> 所属用户权限
+---------------------------------------------> 文件类型（file mode）
```

## 常见的文件类型
>   - `-`（短划线）：普通文件
>   - `d`：目录
>   - `l`：符号链接（软链接）
>   - `c`：字符设备文件
>   - `b`：块设备文件
>   - `s`：套接字文件
>   - `p`：命名管道（FIFO）

## 文件权限解释
文件分为普通文件和目录文件。
对于普通文件，读写执行权限是比较好理解的；
对于目录文件，具有读权限意味着可以读取当前目录下的文件，具有写权限意味着可以更改目录结构，更改目录结构和更改目录下文件的内容是不同的，修改目录下一个普通文件中的内容并不会改变目录结构，在目录下新增/删除文件/目录才会更改目录结构；具有可执行权限意味着可以进入当前目录

## 关于时间
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202308152222627.png)

**时间有哪些分类？**
1. 访问时间（access time）- atime
2. inode 创建时间（inode creation time）
3. 改变时间（change time）- ctime
4. 修改时间（modification time）- mtime

> 通过`ls`命令的不同选项可以查询到不同的时间，查询创建时间采用`ls -lU`，查询访问时间采用`ls -lu`，查询改变时间采用`ls -lc`，查询修改时间采用`ls -l`[^difference_atime_ctime_mtime]

[^difference_atime_ctime_mtime]:[Difference between atime、ctime and mtime](https://en.wikipedia.org/wiki/Stat_(system_call))

在以上4种时间中，inode创建时间是很好理解的，问题在于如何区分其他三种时间类型，
- 向文件写入会修改atime，ctime和mtime
- 文件权限和属主的变更会修改atime和ctime
- 读文件会修改atime

从以上三种情形可以看出，改变是指文件状态的改变，修改是指文件内容的修改，访问操作涉及的范围则相对宽泛，无论读、写还是状态的改变都会涉及到文件的访问

# shell
> 更多内容见[shell 十四问](https://www.cnblogs.com/hongyugao/p/17601142.html)

## 查看当前可用shell
`cat /etc/shells`


## 修改默认shell
1. 使用 `chsh` 工具
`chsh --shell /bin/zsh username`

2. 修改`/etc/passwd`文件

## 特殊变量
- $0 - 脚本名
- \\$1 到 \\$9 - 脚本的参数。$1 是第一个参数，依此类推
- $@ - 所有参数
- $# - 参数个数
- $? - 前一个命令的返回值（返回码）。返回值0表示正常执行，非0表示有错误发生
- \$\$ - 当前脚本的进程识别码
- !! - 完整的上一条命令，包括参数。常见应用：当你因为权限不足执行命令失败时，可以使用 sudo !!再尝试一次。
- $_ - 上一条命令的最后一个参数。如果你正在使用的是交互式 shell，你可以通过按下 Esc 之后键入 . 来获取这个值。

> 返回码可以和逻辑运算符（`||` 和 `&&`）在脚本中搭配使用

**非常用变量**
- $* - 把所有命令行参数看为一个字符串
- $@ - 把所有命令行参数看为一个个独立的字符串

## 替换技术
1. 参数替换 `${}`
在`${}`中，通过`#`、`%`和`/`符号能够实现字符串处理
```
% var="hello"
% echo ${var}
hello

# 单变量时花括号可省略
% echo $var
hello
```

常见的字符串操作根据**修改位置**分为：左侧和右侧，根据**动作类型**分为：删除和替换，根据**匹配长度**分为：最短匹配和最长匹配
实际的可选操作是以上几种分类方法的组合，具体方式如下
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202308152348443.png)

> 注：以下示例使用的变量为var="/dir1/dir2/dir3/my.file.txt"

1. 左侧删除最短匹配
`${var#pattern}`：#表示左侧、删除操作、进行最短匹配，即删除左侧和pattern匹配的最短部分
`${var#*/}`：pattern为*/，结果为dir1/dir2/dir3/my.file.txt

2. 左侧删除最长匹配
`${var##pattern}`：##表示左侧、删除操作、进行最长匹配，即删除左侧和pattern匹配的最长部分
`${var##*/}`：pattern为*/，结果为my.file.txt

3. 右侧删除最短匹配
`${var%pattern}`：%表示右侧、删除操作、进行最短匹配，即删除右侧和pattern匹配的最短部分
`${var%/*}`：pattern为/*，结果为dir1/dir2/dir3

4. 右侧删除最长匹配
`${var%%pattern}`：%%表示右侧、删除操作、进行最长匹配，即删除右侧和pattern匹配的最长部分
`${var%%/*}`：pattern为/*，结果为空值

5. 替换
`${var/pattern/replace}`：表示用replace替换字符串中的pattern，且只替换左侧第一个匹配项
`${var/dir/path}`：把左侧第一个dir替换为path，结果为/path1/dir2/dir3/my.file.txt
`${var//pattern/replace}`：表示用replace替换字符串中的pattern，且替换全部匹配项
`${var//dir/path}`：把全部的dir替换为path，结果为/path1/path2/path3/my.file.txt

**String slicing is used to extract substrings from a string. Its syntax shows below:**
```
${string:start:length}
```
- start: Represents the starting position of the slice (inclusive). If not specified, it defaults to 0.
- length: Repersents the length of substring what you want to get.

2. 算术替换 `(())` 和 `$(())`  
```
% res=$((3+5))
% echo res
8

% ((res = 3 * 5))
% echo $res
15
```

3. 文件名替换
通过通配符匹配文件名
```
% ls
a.c		b.c		c.c		d.c		mkdir_script.sh
% echo *.c
a.c b.c c.c d.c
```

4. 命令替换 \`\` 或 `$()`
shell会首先执行反引号或`$()`包裹的内容，将执行结果作为整个替换部分的值
```
% out=`ls`
% echo $out
a.c
b.c
c.c
d.c
mkdir_script.sh

% out=$(pwd)
%.echo $out
/tmp/missing/lecture2
```

5. 进程替换 `<()` 和 `>()`
进程替换在临时文件和管道操作符之外提供了一种新的方式，这种方式能够更加方便地处理命令之间的输入和输出关系
小括号内包含着命令，进程替换会将小括号内命令的执行结果存入一个临时文件中，同时返回该临时文件的路径


```
% mkdir dir1; touch dir1/{a,b}.c
% mkdir dir2; touch dir2/{b,c}.c
% diff <(ls dir1) <(ls dir2)
1d0
< a.c
2a2
> c.c
```
> diff需要两个文件作为输入，因此需要两个ls命令首先将执行结果放入一个文件中，再把文件路径作为输入值传递给diff命令，这一执行顺序刚好符合进程替换的执行顺序

**如何确定临时文件的位置？**
由于进程替换的返回值就是临时文件的路径，因此只需要在小括号内输入任意命令，通过`echo`命令输出进程替换的返回值即可显示临时文件的路径

```
% echo <(true)
/dev/fd/11

% ls -l <(true)
prw-rw----  0 root  staff  0  7 30 22:50 /dev/fd/11
```

> 注：
> 1. true也是一条命令，不进行操作只返回true值
> 2. /dev/fd 是一个特殊的目录，用于表示进程的文件描述符（File Descriptors），后面的11并不是固定值，只是在执行命令时获得的临时文件描述符为11


## 流
在了解具体的流重定向方法之前，我们需要首先了解Linux下3种特殊的设备：`stdin`, `stdout`, `stderr`。它们都位于`/dev/`目录下，详细信息如下：
```
lr-xr-xr-x  1 root  wheel     0B Nov 26 08:20 stderr -> fd/2
lr-xr-xr-x  1 root  wheel     0B Nov 26 08:20 stdin -> fd/0
lr-xr-xr-x  1 root  wheel     0B Nov 26 08:20 stdout -> fd/1
```
下面要使用的0，1，2实际指代的就是这3类设备

更详细的内容请见[3.6 Redirections | Linux manual](https://www.gnu.org/software/bash/manual/html_node/Redirections.html), 以下只是部分内容。

1. 输入流重定向`<`
```
% cat < hello.txt
hello

% cat 0< hello.txt
```
> - 0<表示将`cat`命令的标准输入（stdin）从名为hello.txt的文件中读取数据，默认情况下，直接使用`<`实现输入流重定向。之所以用0表示，是因为stdin实质上是一个链接文件/dev/stdin，其指向/dev/fd/0这个字符设备文件

2. 输出流重定向`>`
```
% echo hello > hello.txt
% cat hello.txt
hello

% echo hello 1> hello.txt
% cat hello.txt
hello

% echo hello 2> hello.txt
hello
% cat hello.txt
```
> - `1>`表示将**标准输出（stdout）**从终端重定向至文件，由于被重定向因此终端不再显示结果。之所以用1表示是因为stdout->链接文件/dev/stdout->字符设备文件/dev/fd/1
> - `2>`表示将**标准错误输出（stderr）**从终端重定向至文件。`echo hello`的stdout结果是hello，stderr结果是空。stdout未发生重定向，因此终端显示hello，stderr的空结果被重定向至hello.txt。在一般情况下，stdout和stderr都是定向到终端的，通过`1>`和`2>`可以区分二者。之所以用2表示是因为stderr->链接文件/dev/stderr->字符设备文件/dev/fd/2
> - `&>` 和 `>&`表示将**标准输出和标准错误输出**统一重定向至文件

以上的`>`、`1>`和`2>`都是将命令看为黑盒，将命令执行结果划分开

如果考虑命令内部，期望将不同结果输出到stdout或stderr，通过使用`>&1`和`>&2`实现这一点
> - `>&1`表示将结果重定向至标准输出（stdout）
> - `>&2`表示将结果重定向至标准错误（stderr）

通过以下例子，更好地理解以上4种重定向方式
```
# 将以下内容写入script.sh
echo "right" >&1
echo "error" >&2

% ./script.sh 1> right_res 2> error_res
% cat right_res
right
% cat error_res
error
```
 简单来理解，`>&1`和`>&2`用于内部，`1>`和`2>`用于外部

输入流和输出流合用
```
# cat < hello.txt > hello2.txt
# cat hello2.txt
hello
```

3. 向文件追加内容`>>`
> 与输出流重定向类似，存在`1>>`和`2>>`，表示以追加的形式进行stdout和stderr的重定向

```
# echo hello1 > hello.txt
# echo hello2 >> hello.txt
hello1
hello2
```

> 注：需要明确的是在shell中执行都都是一个个的程序，即使是`sudo`也只是一个特殊的程序而已。程序的调用执行是由shell负责的，
> 在使用重定向和管道时，前后要执行的程序是不知情的，重定向和管道是由shell负责的。
> 因此对于命令`sudo echo x > file.txt`，假设当前用户并不具备file.txt的写权限，即使前面添加了`sudo`，执行结果依然是无权限，这是因为该命令的执行逻辑为，shell以`echo x`作为参数执行`sudo`程序，然后将结果输入到file.txt中，实际的输入过程shell仍然是以当前用户身份进行的，并不是以root身份进行的，因此操作仍然无权限。
> 在这种情况下，`echo x | sudo tee file.txt`就可以正常执行，这是因为实际是以root身份执行写入操作的

## 条件判断

- `test`
- `[`
- `[[`

其中`[`和`test`是等效的命令，`[`也称为`test`的别名。三者具体的区别见如下站点

[What is the difference between test, \[ and \[\[ ?](http://mywiki.wooledge.org/BashFAQ/031)

## 通配符

- `*`：匹配任意长度的字符，包括空字符
*.txt 可以匹配所有以.txt结尾的文件


- `?`：匹配单个字符
file?.txt 可以匹配file1.txt、file2.txt等

- `[]`：匹配方括号内的任意一个字符
[abc]file.txt 可以匹配afile.txt、bfile.txt、cfile.txt

- `{}`：用于生成多个相关字符串
file{1,2}.txt -> file1.txt file2.txt
file{a..d}.txt -> filea.txt fileb.txt filec.txt filed.txt
> `..`为范围操作符
 
## Emacs 和 vi 模式切换
如何在terminal下使用vi的操作方式？
见[Chapter 4: The Z-Shell Line Editor 4.1.1: The simple facts](https://zsh.sourceforge.io/Guide/zshguide04.html)

## job control
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202308152348566.png)

**如何管理background job？**
> 使用`jobs`命令可以列出当前终端下的所有job \
> 使用`kill`命令结合各种信号，以下是几种常用的信号及其解释

- `SIGSTOP` 和 `SIGCONT`
用于暂停/恢复进程的执行，`SIGSTOP`特殊在于其**无法被捕获、处理或忽略**
恢复进行执行还可以使用 `fg`（foreground）和`bg`（background）命令，分别控制进程在前台和后台继续执行

- `SIGHUP`
用于通知进程其终端连接已断开的一种信号，当一个终端连接被关闭时，该终端上运行的所有进程都会收到 SIGHUP 信号(和此信号相关的有一个`nohup`命令(Allows for a process to live when the terminal gets killed)，通过`nohup`和`&`的结合使用可以实现后台运行进程且终端关闭后进程不被终止的效果)

- `SIGINT`
通常是由用户在终端上按下Ctrl+C发送给程序的。它用于请求中断程序的执行。默认情况下，大多数程序会响应SIGINT信号并进行清理工作后正常退出

- `SIGTERM`
通常不是由用户手动触发，而是由系统或其他进程发送给程序，用于请求程序停止执行并正常退出

- `SIGKILL`
用于强制终止进程的执行，当进程收到信号时，无论其当前状态如何，该进程都会立即被终止，并且没有机会进行任何清理或处理操作


对于以上内容，难点在于如何理解后4种信号，他们都会让进程终结，但是会应用于不同场景，更加常用的是`SIGINT` 和 `SIGKILL`

## process control

## SSH
If you want a remote mechine excute a command or a script , you have two ways. 
One is that you get a connection to the remote mechine firstly, and then excute a command or a script.
The other is that you can use a special format command. The command which you want to excute follows the ssh command. It looks like `ssh user@hostnames <command>`.

More details can be found in [ssh 教程](https://www.cnblogs.com/hongyugao/p/17624929.html)

The configuration file of ssh is the `~/.ssh/config`, in this file, you can use a hostname, which can simplies the connection sentence for every time.

## Common scenarios
1. move files in this directory to an another directory
`find . -type f -d 1 | xargs -I {} mv {} <target directory>`

## 常见sheel脚本示例
1. 统计当前目录下各文件行数以及总行数
```
#!/bin/bash

total_lines=0

for file in *; do
    if [ -f "$file" ]; then
        lines=$(wc -l < "$file")
        total_lines=$((total_lines + lines))
        echo "File: $file, Lines: $lines"
    fi
done

echo "Total Lines: $total_lines"
```
> 注：该示例除了基本的变量替换以及条件判断，值得注意的一点就是`wc`在使用重定向符`<`时，就无法再确定输入来自何文件，因此输出只会包含计算出的行数，不再包含文件名，刚好符合这里的用法

# 命令别名
1. 直接`alias [alias name]='[command]'`，仅在当前会话中有效
2. 修改`/root/.bashrc`，永久生效

> 注意：alias的`=`两侧不能包含空格

# Symbolic link and Hard link
Please note that when you create a symbolic link, you ought to use the absolute path. If you use the relative path, when you excute the command `ls -l`, you will find the target file of symbolic link is not the file what you want it to point to.

But you can use either relative path or absolute path to create a hard link.

# Search command
- `man`
- `apropos`: Use keyword to search command
- [tldr](https://tldr.inbrowser.app/)

# Reference
> - [1] [shell 十三问原帖](http://bbs.chinaunix.net/thread-218853-1-1.html)
> - [2] [shell 十三问markdown版](https://github.com/wzb56/13_questions_of_shell)
> - [3] [A User's Guide to the Z-Shell](https://zsh.sourceforge.io/Guide/zshguide.html)