---
title: C C++ Const Keyword Unscramble
subtitle:
description:
keywords:
summary:
license:
date: 2024-02-27T09:09:43+08:00
lastmod: 2024-02-27T21:53:31+08:00
tags:
categories:
  - C-C++
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

## Embellish Raw Pointer

The collocation between const and original pointer is confused to many people. There are two usages of it. The key difference is that if the pointer is prohibited to modify or the data which is pointed by pointer is prohibited to modify.

### Pointer To Const(指向常量的指针)

The first one is a variable pointer that points a constant data. i.e. `const int* p`

```
#include <iostream>

int main() {
	int a = 1, b = 2;
	const int *p = &a;

	p = &b;  // true
	*p = 3;  // false

	return 0;
}
```

### Const Pointer(常量指针) 

The second one is a contant pointer that points a variable data. i.e. `int* const p`

```
#include <iostream>

int main() {
	int a = 1, b = 2;
	int* const p = &a;

	p = &b; // false
	*p = 3; // true

	return 0;
}
```

There is a good way to distinguish these two usages. You can judge them by the position of `const` and `*`.

- If the `const` locates the left of the `*`, it means that the const keyword modifies the data `*p`, i.e. a constant data. 
- If the `const` locates the right of the `*`, it means that the const keyword modifies the data `p`, i.e. a constant pointer.

## Embellish C++ Reference

In addition, `const` can also collocates with C++ reference. But there is a litter difference between them. 

That is because the difference between pointer and reference, which is that you can modify pointer pointing later, but you can't modify a reference pointing, which is decided by C++ grammar. A reference must be initialize when we declare it, we can't declare it firstly and then define it, e.g. we can't modify a reference pointing later.

So there is no such situation that we modify the reference. We can just modify the data which is pointed by the reference.

Therefore, there is only one usage of reference, that is the `const` locates the left of the `&`. i.e. `const int &p` or `int const &p`, which means we can't modify the data which pointed by the reference.

## Embellish Member Function Of Class

Although we say the const is used to embellish the mumber function of class, in essence, it embellish the `this` pointer of this class.

There is a tacit fact, `this` is a **const pointer**. For example, the `this` pointer of class `Animal` is `Animal* const`.

We can't let a non-const pointer to point to a const data, so some const object can't invoke some non-const member functions, so we add a `const` qualifier after the parameter list to edit the `this` pointer to become a pointer to const. e.g. `const Animal* const`.

```cpp
Class Animal {
	public:
		int get_number() const {

		}
};
```