---
title: C++ 对象初始化方式对比
subtitle:
description:
keywords:
summary:
license:
date: 2023-06-18T17:18:40+08:00
tags:
categories:
  - C++
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
toc: true
math: true
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

在了解创建对象的方式之前，首先了解一下**初始化**和**赋值**两个操作，
1. 初始化是创建变量时赋予其一个初始值，即初始化之前并不存在变量
2. 赋值是把对象的当前值擦除，用新值代替旧值，即赋值之前存在变量
让人困惑的是`=`既可以用于初始化，也可以用于赋值，不要认为初始化和赋值是相同的操作

关于默认值（默认初始化），值和变量类型和位置决定
1. 函数体外部的内置类型默认初始化为0
2. 函数体内部的内置类型不被默认初始化，数值为未定义
> 需要注意的是，这里的函数体是不包括main函数的，实测main函数内部的内置类型即使不被初始化，其值也为0

四种方式分别为 等号，圆括号，花括号,最后一种是等号和花括号一起使用
```c++
int a = 0;
int a(0);

int a{0};
int a = {0};
```
其中，等号和圆括号是以前就存在的，使用花括号进行初始化或赋值是C++11新标准引入的。
需要理解的是`=`在这里代表的是初始化而非赋值，而且在实际使用中我们通常会忽略"="和花括号组合初始化的语法，因为C++通常把它视作和只有花括号一样。所以说我们关键要看的就是下面这3种方式。
```c++
int a = 0;
int a(0);
int a{0};
```
其中，比较难区分的是圆括号和花括号，当然在上述代码中由于他们起到的效果是相同的，因此并不难区分。以vecotr为例，介绍他们的不同之处

1. 花括号内的值和容器内元素类型相同是为列表初始化，否则根据给定容量和值初始化。即花括号内的内容可以是以下几种类型
- 花括号内元素类型均为容器内元素类型
- 花括号内元素类型不完全相同 并且顺序为 数字 + 和容器内元素类型相同的元素
```c++
vector<int> v1{10, 1};       // 第一种情况，列表初始化，v1包含两个元素10和1
vector<string> v2{10, "1"}; // 第二种情况，列表初始化，v2包含10个值为“1”的元素
```

2. 圆括号只能根据给定容量和值进行初始化，即圆括号内的内容只能是以下两种之一
- 数字
- 数字 + 与容器内类型相同的元素
```c++
vector<int> v3(10);          // 正确，v3有10个元素，每个值为默认值0
vector<string> v4(10, "hi"); // 正确，v4有10个元素，每个值为“hi”
vector<string> v5("hi");     // 错误，必须包含数字
```

使用花括号具有2个优点:
1. 防止变窄转换: 大括号不支持变窄转换，等号和圆括号为了向下兼容支持变窄转换。
```c++
long double ld = 3.1415926;
int a{ld}, b = {ld}; // 错误，因存在变窄转换，花括号初始化禁止这种行为
int c(ld), d = ld;   // 正确，虽然存在信息丢失，但圆括号和等号并不会禁止这种行为
```
2. 免疫C++最令人头疼的解析: C++规定任何可以被解析为一个声明的东西必须被解析为声明，因此无法区分无参的构造函数和函数声明，此规则会默认其为函数声明

花括号初始化共包含2种用途
1. 变量的列表初始化[^initializer_list]
```c++
int a = {0};
int a{0};
```
[^initializer_list]: [std::initializer_list](https://en.cppreference.com/w/cpp/utility/initializer_list)

2. 构造函数初始值列表[^Constructors_and_member_initializer_lists]
```c++
class myClass {
    private:
        int a;
        int b;
    public:
        myClass(int para_a, int para_b) : a(para_a), b(para_b) {}
};
```
[^Constructors_and_member_initializer_lists]: [Constructors and member initializer lists](https://en.cppreference.com/w/cpp/language/constructor)

构造函数初始值列表由3部分组成：函数头、初始值列表和函数体，成员属性的初始化工作发生在函数体执行之前，因此**初始值列表执行的是初始化，而函数值执行的则是初始化后的赋值**。
在C++中，**const变量**、**引用**和**不具备默认构造函数的类类型**是无法首先进行默认初始化，然后再赋值，因此在这3种情况下，必须使用初始值列表

同时需要注意的是**初始值列表只用于初始化成员的值，并不会限定初始化的具体执行顺序**，在涉及到用一成员变量的值初始化另一个成员变量的值时要注意执行顺序

花括号初始化的缺点体现在构造函数的调用顺序方面

1. 直接初始化
并不能简单地把直接初始化理解为不需要使用现存的占据实际空间的对象来初始化其他对象,因为在示例代码中第2个例子虽然使用到了一个现存的对象dots,但是其依然属于直接初始化

<details>
<summary>直接初始化示例代码</summary>

```c++
// 好理解的直接初始化
string dots(10, '.');

// 不易理解的直接初始化
string s(dots);
```
</details>

2. 拷贝初始化
需要注意的是第1行示例代码同不易理解的直接初始化的区别,虽然两者都利用了现存的对象dots,但是它们属于不同的初始化方式.

<details>
<summary>拷贝初始化示例代码</summary>

```c++
string s2 = dots;
string s3 = "666";
```
</details>

涉及到拷贝初始化的场景
> - 使用`=`定义变量时
> - 一个对象作为非引用类型的形参时
> - 一个返回值类型非引用类型的函数返回一个对象时
> - 使用花括号初始化数组或聚合类

直接使用花括号和等号与花括号一同使用是直接初始化还是拷贝初始化?
如果说以stl中的insert/push同emplace相比,前者进行的是拷贝初始化,后者进行的是直接初始化,那么拷贝和直接的区别就在于是否多经历了一次变量的产生过程,因为push需要首先把待插入的对象真正创建出来然后将其拷贝到另一对象中,而emplace则是直接把数据构造成最终对象

拷贝初始化,拷贝构造函数,拷贝赋值运算符这三者之间的关系容易混淆,
需要明确初始化和赋值的区别,初始化是在对象定义的过程中顺便完成原始值的处理,而赋值是将一个对象赋给一个已存在的对象,初始化的对象是以前不存在的对象,赋值运算符的对象是已经存在的对象

拷贝初始化和直接初始化的区别是否仅仅是是否生成了中间变量

拷贝初始化和直接初始化，这两个说法是独立于 类的成员函数这一概念的。拷贝初始化和直接初始化的本质是初始化，强调的是同赋值这一操作之间的区别，但是他们究竟会调用何种类成员函数是不确定的，直接初始化可能调用构造函数，也可能调用拷贝构造函数。

# Reference
> - [1] [区别使用()和{}创建对象](https://cntransgroup.github.io/EffectiveModernCppChinese/3.MovingToModernCpp/item7.html)