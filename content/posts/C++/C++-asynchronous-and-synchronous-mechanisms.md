---
title: C++ Asynchronous and Synchronous Mechanisms
subtitle:
description:
keywords:
summary:
license:
date: 2023-09-10T22:06:00+08:00
lastmod: 2024-01-31T18:26:04+08:00
tags:
categories:
  - C++
  - HPC
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

## Asynchronization (Multithreading)
The first thing we need to do is understanding the correlations between multithreading and parallel computing.
> Multithreading is one of the many ways to implement parallel computing.

**Why do we need asynchronous mechanisms ?**
If we excute some time-consuming operations in main thread, they will block main thread which causes the bad user experience.
An effective way to solve this problem is to use asynchronous mechanisms. Creating a child thread that is asynchronous with main thread and the time-consuming operation can be done by child thread, main thread can continue do its work.

The asynchronous mechanisms between main thread and child thread can improve parallelism and performance. But we need some mechanisms to adjust execution orders of multiple child threads to solve the resource competition issues and avoid data access conflict. That is synchronous mechanisms, it ensures the correctness of programs execution results by adjusting the exection order of multiple threads.

**How do we implement asynchronous mechanisms**
1. `std::thread`
C++11 provides thread support in language level, 
<details>
<summary>std::thread example code</summary>

```
#include <iostream>
#include <thread>
#include <functional>
#include <chrono>

void func2() {
	std::this_thread::sleep_for(std::chrono::seconds(2));
	std::cout << "func2" << std::endl;
}

int main() {
	std::function<void()> func1 = []() -> void { 
		std::this_thread::sleep_for(std::chrono::seconds(1));
		std::cout << "func1" << std::endl;
	};

	std::thread t1(func1);
	std::thread t2(func2);

	std::cout << "thread executing" << std::endl;

	t1.join();
	t2.join();

	return 0;
}
```
</details>

> **Why C++ use term "join" to represent blocking the current thread until the thread identified by \*this finishes its execution ?**
The term "join" in the context of threading comes from the concept of joining or combining the execution of multiple threads back into a single flow of control. When you call join() on a thread object, you are essentially saying that you want to wait for that thread to complete its execution before the current thread continues.
The above is the official explanation for term "join". But according to this statement, in my opinion, the usage `main_thread.join(son_thread)` is more resonable. 
So we can explain it from another aspect, i.e. visualization: Imagine threads as separate paths or streams of execution in a program. When you "join" a thread, it's as if you are waiting for that thread to merge or combine its results or progress with the current thread. This visual analogy helps in understanding the purpose of the function.

2. `std::promise`、`std::packaged_task` 、`std::async`
They all can create an asynchronous operation. 
`std::future`: `std::future` can help us to solve this problem, it has three states and three ways to get its state.
`std::promise`: It looks like the code promises it will provide a result in the future, at that time, you can get a result.
`std::packaged_task`: It looks like the code promise it will do something in the future, at that time, you can get a result.
We can note that the promise and packaged_task both mentions future, in fact, you both use `std::future` to provide a result.

<details>
<summary>std::promise example code</summary>

```
#include <thread>
#include <future>
#include <iostream>

int main()
{
    std::vector<int> numbers = { 1, 2, 3, 4, 5, 6 };
    std::promise<int> accumulate_promise;

	std::thread child_thread([&]() {
		int sum = 0;
		for (std::vector<int>::iterator pointer = numbers.begin(); pointer < numbers.end(); pointer++) {
			sum += *pointer;
		}
		accumulate_promise.set_value(sum);
	});
	
    std::future<int> accumulate_future = accumulate_promise.get_future();
	
	// get() function only ensures that future has a valid result, but child thread maybe don't finish at this time. So we also need to use join() function
    std::cout << "result=" << accumulate_future.get() << std::endl;
    child_thread.join();  // wait for thread completion

	return 0;
}
```
</details>


<details>
<summary>std::packaged_task example code</summary>

```
#include <iostream>
#include <future>
#include <functional>

int main() {
	// auto sum = [](int a, int b) -> int {return a + b;};
	std::function<int(int, int)> sum = [](int a, int b) -> int {return a + b;};

	std::packaged_task<int(int, int)> task(sum);
	std::future<int> result = task.get_future();

	task(2, 9);
	
	std::cout << "task result = " << result.get() << std::endl;

	return 0;
}
```
</details>

<details>
<summary>std::async example code</summary>

```

```
</details>

The second way is convienter than `std::thread`, and them solve a problem of thread.
 If we want to get the result of a child thread, maybe we need to use global variable. So c++ create a `std::future` to provide a mechanism to access the result of asynchronous operations. In other words, promise、packaged_task and async all can provide a future object to creator of asynchronous operations.


## Synchronization
There are some common approaches to implement the synchronous mechanisms, they are 
- semaphore
- mutex
- condition variable
- atomic

**What is the relationship between semaphore、mutex and condition value ?**
semaphore -> mutex
mutex + semaphore -> condition variable
condition variable + mutex -> semaphore

**Why we need to use the synchronous mechanisms ?**
When multiple threads or processes use the shared variables, writing at the same time will cause the wrong result.
We need to use the synchronous mechanisms to adjust different execution index of threads and processes.

**How can we understand the role of mutex and atmotic ?**
We can use mutex to wrap many statements and use atmotic to implement one operation.
And mutex is a heavy operation and atomic is a light operation, we ought to select the suitable tool to finish our work.

### Locking (Mutex)
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202309182200311.png)

1. Using common-style locks, which ought to be released manually by `this.lock()` and `this.unlock()`
- [std::mutex](https://en.cppreference.com/w/cpp/thread/mutex)
- [std::recursive_mutex](https://en.cppreference.com/w/cpp/thread/recursive_mutex)
- [std::timed_mutex](https://en.cppreference.com/w/cpp/thread/timed_mutex)
- [std::recursive_timed_mutex](https://en.cppreference.com/w/cpp/thread/recursive_timed_mutex)

These common-style locks have disadvantages, they must be released manually, if we forget this process, program will appear problems. So we can use the second type of locks.

2. Using RAII-style locks, which can manage locks automatically. They like a package for common-style locks.
- [std::lock_guard](https://en.cppreference.com/w/cpp/thread/lock_guard)
- [std::scoped_lock]()
- [std::unique_lock](https://en.cppreference.com/w/cpp/thread/unique_lock)

1. The simplest way
Using `std::mutex` to create a lock object, `this.lock()` to lock this mutex, `this.unlock()` to unlock this mutex.

### Atomic
What we need to do to get an atomic is just finish this subsitutation `int counter = 0` -> `std::atomic<int> counter = 0`. Then we can use the counter as a ordinary variable. It look like the following code.
```
#include <iostream>
#include <thread>
#include <atomic>

int main() {
	//int counter{0};
	std::atomic<int> counter{0};	

	std::thread t1([&]() -> void {
		for (size_t i = 0; i < 10000; i++)
			counter += 1;
	});

	std::thread t2([&]() -> void {
		for (size_t i = 0; i < 10000; i++)
			counter += 1;
	});

	t1.join();
	t2.join();
	
	std::cout << "counter = " << counter << std::endl;

	return 0;
}
```

**Which operators does atomic support ?**

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202309251315078.png)

Another two important functions are `compare_exchange_weak` and `compare_exchange_strong`, which are abbreviated as CAS(Compare And Swap) in many places. We can use them to implement above any one fetch_xxx functions.

## Note
The performance of compiling programs in Linux and macOS is different, e.g. in Linux, we need to use the `-pthread` option when preprocessing and linking. But in macOS, we needn't use this option. The reason is that Linux uses the POSIX thread library and macOS uses the Grand Central Dispatch（GCD）. The compiler in macOS will link GCD automatically, so we needn't link it manually.

## Reference
> - [1] [如何理解std::async、std::future、std::promise、std::packaged_task](https://www.cnblogs.com/qicosmos/p/3534211.html)
> - [2] [Concurrency support library - cppreference](https://en.cppreference.com/w/cpp/thread)

----
**a development process:**
1. use std::thread directly（when main thread ends, although son thread is excuting, it also ends）
2. use std::thread::join to command main thread to wait for son thread
3. when son thread is used in function, if function ends, the thread object as a class, it has a deconstruction function, it will destruction the son thread, so you can use detach to avoid thread object to control the excution of son thread
4. use global variable to store son thread created by function, and at the last of the main thread, use std::thread::join to block the main thread until all son threads finish their execution.
5. a problem of the forth way is that we need to excute the std::thread::join manually. In the final analysis, thread is also a type of resource, so we can refer to the [RAII](https://www.cnblogs.com/hongyugao/p/17489579.html#:~:text=RAII,-Resource%20Acquisition%20Is) thought, use a class to control this resource.
