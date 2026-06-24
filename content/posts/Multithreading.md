---
date: '2026-06-24T09:21:39+05:30'
draft: false 
title: 'Multithreading'
---
## What is Multithreading?

Morden Computers has at least 4 core and some high end proccessor has 16, 32, 64, ... core.

Now, What is core? core is compleate cpu unit on its own. Means it can completate computation on its own like getting data from memory to chache and computing using ALU. When we write general cpp code (without using thread libaray) , it remain single threaded means it use only one core. But if we divide our code and run on diffrent core it compute in parralel and give better performance.

## How to use Multithreading in cpp?

To use multithreading in C++, we have to use the C++ standard thread library, which provides threads as classes and thread functionality as class functions.

You just need to create a thread class object that takes a C++ function as an input parameter, along with any arguments that the function requires. As soon as the thread class object is created, it starts to run as a separate thread and utilizes parallel computing

```
#include <thread>

void somerandomfunction(){ // do somting }

int main(){
	thread t1(somerandomfuntion);
	t1.join();
	return 0;
}
```

The `t1.join()` method will make the main thread wait for the t1 thread to finish execution. On the other hand, `t1.detach()` will free the thread from the C++ runtime, leaving the Operating System (OS) to handle it independently.

## What are Mutex?

### The Problem: Race Condition

When two threads t1 and t2 run in parallel and attempt to access a shared variable (`int counter = 0`), unexpected behavior can occur. 

For example, if both threads try to increment `counter` to 100,000, they might interfere with each other:

* **At 1 ms:** The `counter` value is 5,000. Both t1 and t2 read this value simultaneously.
* **Thread 1 (t1):** Increments the value from 5,000 to 8,000 in its CPU cycle and writes it back to memory. 
* **Thread 2 (t2):** Increments its own cached value from 5,000 to 7,000 and writes it back *after* t1, overwriting t1's work.

**The Result:** Instead of the sequential updates combining to make the counter 10,000, the final value is incorrectly saved as 7,000. 

---

### The Solution: Mutex

To prevent this data corruption, a **Mutex (Mutual Exclusion)** lock is used. Let variable (where data is stroed) is room. when you(thread) need change something you first lock the room so that other can't enter the room untill your work is done.

you can also cpp lock_gaurd functionality. It will automatically unlock the room when room goes out of scope. 

## Mutex vs Atomic

### Atomic
As the name suggests, we say something is atomic if it cannot be further divided into smaller units. In C++, if you declare a variable as atomic, it means all three steps (read -> compute -> write) are combined into a single step that cannot be broken up across different CPU cycles.

If t1 reads an atomic variable, t2 cannot even read it until t1 finishes writing its value back.

### Difference 

A Mutex is a multithreading concept used to prevent race conditions by locking blocks of code, whereas an Atomic variable is a fundamental property given to a specific variable ensuring it performs its entire read-modify-write cycle in a single, uninterrupted operation.

## What's ahead

I have already coded matrix multiplication and improved its performance using tiling (blocking) and multithreading. Since this blog post is already getting big, I will explain that implementation in the next one!

## Some Code

#### mutex.cpp
```
#include <iostream>
#include <thread>
#include <mutex>

using namespace std;

int counter=0;
mutex mx;

void add(){
  lock_guard<mutex> lock(mx);
  for(int i=0; i<1000000; i++){
    counter++;
  }
}

int main(){
  thread t1(add);
  thread t2(add);

  t1.join();
  t2.join();

  cout<<counter<<endl;
  return 0;
}
```

#### atomic.cpp
```
#include <iostream>
#include <thread>
#include <atomic>

using namespace std;

atomic<int> counter(0);

void add(){
  for(int i=0; i<2000000; i++){
    counter++;
  }
}

int main(){
  thread t1(add);
  thread t2(add);

  t1.join();
  t2.join();

  cout<<counter<<endl;
  return 0;
}
```