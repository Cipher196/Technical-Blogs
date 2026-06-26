---
date: '2026-06-26T11:03:02+05:30'
draft: false
title: 'Karatsuba'
---
## Idea
The main idea of this Algo is Divide and Concure. We divide big integer into half so that multiplication become easy. We choose a size `n` such that if size of integer is less than `n` then dividing into half become unnessary. Such `n` is known as Karatsuba threshold. 

Let A and B is bigint of size `n` then divide both into two part A0, A1, B0 and B1 each of size `k=n/2`.
and let s = b^k where b is base.

Then, 
```
 A*B = ( A1*b + A0 )*( B1*b + B0 ) = A1*B1*b^2 + ( A1*B0 + A0*B1 )*b + A0*B0
 
 and 
 
 (A1-A0)*(B1-B0)=A1*B1 - A1*B0 - A0*B1 + A0*B0
 
 => A1*B0 + A0*B1 = A1*B1 + A0*B0 - (A1-A0)*(B1-B0)
```


## Algorithm

```
if n < n0 then return BasecaseMultiply(A, B) // naive multiplication

k ← n/2

(A0, B0) := (A, B) mod β^k
(A1 , B1 ) := (A, B) div β^k

C0 ← KaratsubaMultiply(A0 , B0 )
C1 ← KaratsubaMultiply(A1 , B1 )
C2 ← KaratsubaMultiply(A0−A1, B0-B1)

return C := C0 + (C0 + C1 − C2)*β^k + C1*β^2k .
```

## Time Complecity

As Our task of multiply is broken into 3 small multiplication of half size and add which cost O(n).

So,
```
 T(n) = 3*T(n/2) + O(n)
 
 By Master Therom, first case 
 
 If f(n) = O( n^(logb(​a)−ϵ) ) for some constant ϵ>0, then: T(n)=Θ( n^logb(​a) ).
 
 So T(n) = n^( log2(3) ) = n^1.585
 
```

## Code

#### karatsuba.hpp

```
#include "BasecaseMultiply.hpp"
#include "addition_and_subtraction.hpp"
#include "bigint.hpp"

inline size_t karatsuba_threshold = 10;

inline bigint mod_with_basePower_k(const bigint &a, size_t k) {
  bigint c(a.get_digits()[0]);
  for (int i = 1; i < k; i++) {
    c.get_digit().push_back(a.get_digits()[k]);
  }
  return c;
}

inline bigint div_with_basePower_k(const bigint &a, size_t k) {
  bigint c(a.get_digits()[k]);
  for (int i = k + 1; i < a.get_digits().size(); i++) {
    c.get_digit().push_back(a.get_digits()[k]);
  }
  return c;
}

inline bigint multiply_karatsuba(const bigint &a, const bigint &b) {
  size_t l = std::max(a.get_digits().size(), b.get_digits().size());
  if (l < karatsuba_threshold)
    return multiply_naive(a, b);

  size_t k = l / 2;

  bigint a0 = mod_with_basePower_k(a, k);
  bigint b0 = mod_with_basePower_k(b, k);

  bigint a1 = div_with_basePower_k(a, k);
  bigint b1 = div_with_basePower_k(b, k);

  bigint c0 = multiply_karatsuba(a0, b0);
  bigint c1 = multiply_karatsuba(a1, b1);
  bigint c2 = multiply_karatsuba(subtract(a1, a0), subtract(b1, b0));

  bigint c3 = add(c0, c1);
  c3 = subtract(c3, c2);
  c3 = shift(c3, k);

  bigint c4 = shift(c1, 2 * k);
  c4 = add(c3, c4);
  c4 = add(c4, c0);

  return c4;
}
```

Ohter code is avilable on [git hub](https://github.com/Cipher196/cpp-deep-dive/tree/main/bigint)