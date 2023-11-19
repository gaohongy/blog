---
title: C++类型转换
subtitle:
date: 2023-11-19T21:33:22+08:00
draft: false
author:
  name:
  link:
  email:
  avatar:
description:
keywords:
license:
comment: false
weight: 0
tags:
categories:
  - C++
hiddenFromHomePage: false
hiddenFromSearch: false
summary:
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: true
lightgallery: false
password:
message:
repost:
  enable: false
  url:

# See details front matter: https://fixit.lruihao.cn/documentation/content-management/introduction/#front-matter
---

# 普通类型

# 类类型

对于类类型,编译器只能自动执行一步隐式类型转换.例如从字符串字面值转换为string类型,但是无法继续将string隐式转换为其他类型

# 新式显示类型转换
格式
> cast-name<type>(expression);

## static_cast
> Any well-deﬁned type conversion, other than those involving low-level const, can be requested using a `static_cast`.

```
// i and j are both int type,
double slope = static_cast<double>(j) / i; 

// perform a conversion that the compiler will not generate automatically.
// converts void * back to the original pointer type
void * p = &d;
double * dp = static_cast<double * >(p);
```

We should note that, the difference between `static_cast` and C style cast is that **if you try to cast an entity which is not compatible to another, then static_cast gives you an compilation time error unlike the implicit c-style cast**.

So, `static_cast` also can't handle the compatible type cast, we should not recognize that we can use it to implement any type conversion we want.

A real example, when I want to use a `int` type variable to edit the element in a `std::string`, I use the following commands, and I got an error which is difficult to find.
```c++
std::string str = "000";
str[0] = static_cast<char>(1);
```
When I execute `std::cout << str`, I got nothing. Until I use the gdb/lldb, I find the result of `static_cast<char>(1)` is `'\x01'`, it is not the '1' or 49(the ASCII of '1') which I expected.

**What is the `\x01`? &&  Why we get it?**

Before encountering this problem, I don't know the principle of `static_cast`. In fact, `static_cast` cut out the low 1 byte (the length of `char`) of the number waitring to be converted. The `\x01` is the content of that 1 byte.

Maybe we will confused with the difference between `1 + '0'` and `static_cast<char>(1)`. In fact, the former is not a type conversion, it just add 1 to the ASCII of the character '0'.

## const_cast
> `const_cast` only be used to change the constness of an expression.

It is different from the `static_cast`, which only be used to change the type of an expression. 
i.e. if we use `static_cast` to change the constness of an expression or `const_cast` to change the type of an expression, we will get a compile-time error.
```
const char * cp; 

// error: static_cast can’t cast away const
char * q = static_cast<char * >(cp);

// error: const_cast only changes constness
const_cast<string>(cp);
```

## reinterpret_cast
> performs a low-level reinterpretation of the bit pattern of its operands.

```
int * ip; 
char * pc = reinterpret_cast<char * >(ip);
```

It is worth noting that `reinterpret_cast` is dangerous. As the following code shows.
```
int * ip; 
char * pc = reinterpret_cast<char * >(ip);
string str(pc);
```
We ought to know that no matter what type of pointer it is, the pc itselt is just a address(a number) and the type shows the type of object addressed by this pointer.
So the actual object addressed by pc is an int, not a character. Any use of pc that assumes it’s an ordinary character pointer is likely to fail at run time.
But about `string str(pc)`, the compiler has no way of knowing that it actually holds a pointer to an int. Thus, the initialization of str with pc is absolutely correct—albeit in this case meaningless or worse.

## dynamic_cast