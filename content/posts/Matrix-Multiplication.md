---
date: '2026-06-25T20:59:21+05:30'
draft: false
title: 'Matrix Multiplication'
---
## Why Matrix Multiplication?

Because it's the most important operation in today's world. All of AI and LLMs run because of matrix multiplication. Optimizing it is an extremely important task, so give it a try.

I tried to find the square (A x A) of a 1000x1000 matrix.

## What is Tiling?

Tiling or blocking is just an approach to use the L1 and L2 cache in the CPU as much as we can. In my code, I broke the rows and columns into different blocks and multiplied them together.

Suppose you have an n x n matrix. Choose `k = n/10` or anything lower than `n`. Multiply them block by block—the first block of `k x n` size with another block of `k x n` size. Choose `k` such that all the elements of these blocks can easily fit into the L2 cache of the CPU.

## What Mistake I Made in Tiling

Normally, we use three loops for regular matrix multiplication and six loops for tiling-based matrix multiplication.

Inside the three loops of the tiling method, I used `min(ii+sb, n)` inside the loop. The mistake is that it computes the minimum every time we compare `i` with it. This is unnecessary; since it does not depend on `i`, we can precompute it.

## How I Used Multithreading

For multithreading, I divided the n x n matrix into n/2 x n/2 matrices and created 4 threads to handle all 4 of these blocks.


## Results

| name of Methord | time taken |
|-----------------|------------|
| normal matrix multiplication |17.207 Sec|
| with tiling aproch 1| 26.407 Sec |
| with tiling aproch 2|  14.223 Sec|

#### With multithreading 

| name of Methord | time taken |
|-----------------|------------|
| normal matrix multiplication |5.417 Sec|
| with tiling aproch 1| 7.03 Sec |
| with tiling aproch 2|  4.88 Sec|

As we can see we got aprox 3.5x speedup with multithreading (when i used 4 thread).

#### Result with -O1 flag

## Compiler Optimization Flags

First of all, it's not `-01` (zero one); it's `-O1` (capital O, one).

This flag tells the C++ compiler to optimize the code while compiling. Without it, the compiler uses the default flag `-O0` (capital O, zero), which tells the compiler not to optimize it. When using `-O0`, the code remains exactly as we wrote it, and we can easily set breakpoints for debugging. 

However, we cannot easily debug an optimized binary. This is because the compiler changes our code and can remove or add variables, functions, and even loops when we tell it to optimize the code.

There are three main levels of optimization: `-O1`, `-O2`, and `-O3`.

We are using `-O1`.

| name of Methord | time taken |
|-----------------|------------|
| normal matrix multiplication |2.768 Sec|
| with tiling aproch 1| 3.684 Sec |
| with tiling aproch 2|  0.903 Sec|

#### With multithreading 

| name of Methord | time taken |
|-----------------|------------|
| normal matrix multiplication |1.248 Sec|
| with tiling aproch 1| 0.593 Sec |
| with tiling aproch 2|  0.527 Sec|


## Code

#### tiling methord without threading 

```
#include <chrono>
#include <iostream>
#include <vector>
#include <cstdlib>

using namespace std;

int n=1000; // size of matrix
int k=100; // range of element of matrix 0 to k 
int bs=64; // block size


int main(){
  auto start = chrono::high_resolution_clock::now();

  vector<vector<int>> v(n, vector<int> (n));

  for(auto &x: v){
    for(auto &y: x){
      y=rand()%k; // so number remain in range 0 to k
    }
  }

  vector<vector<int>> a(n, vector<int> (n)); // a = v X v

  for(int ii=0; ii<n; ii+=bs){
    for(int jj=0; jj<n; jj+=bs){
      for(int kk=0; kk<n; kk+=bs){

        int im=min(ii+bs,n);
        int jm=min(jj+bs,n);
        int km=min(kk+bs,n);

        for(int i=ii; i<im; i++){
          for(int j=jj; j<jm; j++){
            int sum=0;
            for(int k=kk; k<km; k++){
              sum+=v[i][k]*v[k][j];
            }
            a[i][j]=sum;
          }
        }

      }
    }
  }

  auto end = chrono::high_resolution_clock::now();

  auto ms = chrono::duration_cast<chrono::milliseconds>(end - start).count();

  float time_taken = ms;
  time_taken/=1000;

  cout<<"Time taken by tiling 2 matrix multiplication: "<<time_taken<<" Sec \n";
  return 0;
}
```

#### tiling with multithreading 

```
#include <chrono>
#include <iostream>
#include <vector>
#include <cstdlib>
#include <thread>

using namespace std;

int n=1000; // size of matrix
int k=100; // range of element of matrix 0 to k 
int bs=64; // block size

void solve(vector<vector<int>> &a, vector<vector<int>> &v, int sr, int er, int sc, int ec){
  for(int ii=sr; ii<er; ii+=bs){
    for(int jj=sc; jj<ec; jj+=bs){
      for(int kk=0; kk<n; kk+=bs){

        int im=min(ii+bs,er);
        int jm=min(jj+bs,ec);
        int km=min(kk+bs,n);

        for(int i=ii; i<im; i++){
          for(int j=jj; j<jm; j++){
            int sum=0;
            for(int k=kk; k<km; k++){
              sum+=v[i][k]*v[k][j];
            }
            a[i][j]=sum;
          }
        }


      }
    }
  }
}

int main(){
  auto start = chrono::high_resolution_clock::now();

  vector<vector<int>> v(n, vector<int> (n));

  for(auto &x: v){
    for(auto &y: x){
      y=rand()%k; // so number remain in range 0 to k
    }
  }

  vector<vector<int>> a(n, vector<int> (n)); // a = v X v
  
  int mid=n/2;

  thread t1(solve, ref(a), ref(v), 0, mid, 0, mid);
  thread t2(solve, ref(a), ref(v), 0, mid, mid, n);
  thread t3(solve, ref(a), ref(v), mid, n, 0, mid);
  thread t4(solve, ref(a), ref(v), mid, n, mid, n);

  t1.join();
  t2.join();
  t3.join();
  t4.join();


  auto end = chrono::high_resolution_clock::now();

  auto ms = chrono::duration_cast<chrono::milliseconds>(end - start).count();

  float time_taken = ms;
  time_taken/=1000;

  cout<<"Time taken by multi-threaded tiling matrix multiplication: "<<time_taken<<" Sec \n";
  return 0;
}
```

Other code you can see on github :)