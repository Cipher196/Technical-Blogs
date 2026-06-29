---
date: '2026-06-29T19:37:28+05:30'
draft: false
title: 'Toom Cook'
math: true
---
## Idea
Schoolbook multiplication take $\mathcal{O}(n^2)$ and Karatsuba decrease it to $\mathcal{O}(n^{\log_23})$.

In karatsuba we divide our bigint into 2 parts, Now One may ask what if we divide bigint into 3 parts? Will it give better complexity? Answer is Yes.

In n-way Toom Cook we divide our bigint in n parts. Hence karatsuba is just 2-way Toom Cook. 

## Maths
**Algorithm 1.4** ToomCook3  
**Input:** Two integers $0 \le A, B < \beta^n$  
**Output:** $A \cdot B := c_0 + c_1\beta^k + c_2\beta^{2k} + c_3\beta^{3k} + c_4\beta^{4k}$ with $k = \lceil n/3 \rceil$  
**Require:** A threshold $n_1 \ge 3$  

1. **if** $n < n_1$ **then return** $\text{KaratsubaMultiply}(A, B)$
2. Write $A = a_0 + a_1x + a_2x^2$, $B = b_0 + b_1x + b_2x^2$ with $x = \beta^k$.
3. $v_0 \gets \text{ToomCook3}(a_0, b_0)$
4. $v_1 \gets \text{ToomCook3}(a_{02} + a_1, b_{02} + b_1)$ **where** $a_{02} \gets a_0 + a_2, b_{02} \gets b_0 + b_2$
5. $v_{-1} \gets \text{ToomCook3}(a_{02} - a_1, b_{02} - b_1)$
6. $v_2 \gets \text{ToomCook3}(a_0 + 2a_1 + 4a_2, b_0 + 2b_1 + 4b_2)$
7. $v_\infty \gets \text{ToomCook3}(a_2, b_2)$
8. $t_1 \gets (3v_0 + 2v_{-1} + v_2)/6 - 2v_\infty, \quad t_2 \gets (v_1 + v_{-1})/2$
9. $c_0 \gets v_0, \quad c_1 \gets v_1 - t_1, \quad c_2 \gets t_2 - v_0 - v_\infty, \quad c_3 \gets t_1 - t_2, \quad c_4 \gets v_\infty$.

## Results

### Without -O1 optimization

#### n=1000

Time taken by naive multiply : 0.113 Sec

Time taken by karatsuba multiply : 0.084 Sec

Time taken by toom cook multiply : 0.064 Sec

#### n=5000

Time taken by naive multiply : 3.417 Sec

Time taken by karatsuba multiply : 1.688 Sec

Time taken by toom cook multiply : 0.969 Sec

#### n=10000

Time taken by naive multiply : 13.234 Sec

Time taken by karatsuba multiply : 5.348 Sec

Time taken by toom cook multiply : 2.453 Sec

### With -O1 optimization

#### n=1000

Time taken by naive multiply : 0.016 Sec

Time taken by karatsuba multiply : 0.012 Sec

Time taken by toom cook multiply : 0.01 Sec

#### n=5000

Time taken by naive multiply : 0.417 Sec

Time taken by karatsuba multiply : 0.249 Sec

Time taken by toom cook multiply : 0.134 Sec

#### n=10000

Time taken by naive multiply : 2.003 Sec

Time taken by karatsuba multiply : 0.749 Sec

Time taken by toom cook multiply : 0.336 Sec

## Whats Next
Now Next step is FFT. But I will only learn it and will not implement it Because implementing toom cook took my nerve down. Hence I will only learn it from CLRS.

## Code
```
#include "karatsuba.hpp"

inline int toom_threshold = 30;

inline bigint scalar_divide(const bigint &a, unsigned __int128 b) {
  bigint c = a;
  vector<uint64_t> &v = c.get_digit();
  unsigned __int128 k = 0;
  for (size_t i = v.size() - 1; i >= 0; i--) {
    k = (k << 64) + v[i];
    v[i] = (uint64_t)k / b;
    k = k % b;
    if (!i)
      break;
  }
  if (!a.ip())
    c.flipSign();

  while (c.get_digits().size() > 1 && c.get_digits().back() == 0) {
    c.get_digit().pop_back();
  }

  return c;
}

inline bigint place_in_front(const bigint &c,
                             const bigint &v) { // assuming v is positive
  bigint k = v;
  for (size_t i = 0; i < c.get_digits().size(); i++) {
    k.get_digit().push_back(c.get_digits()[i]);
  }
  if (!c.ip())
    k.flipSign();
  return k;
}

inline bigint multiply_toom_cook(const bigint &a, const bigint &b);

inline bigint multiply_toom_cook_core(const bigint &a, const bigint &b) {
  size_t l = std::max(a.get_digits().size(), b.get_digits().size());
  if (l < toom_threshold) {
    return multiply_karatsuba(a, b);
  }

  size_t k = l / 3;

  bigint a0 = mod_with_basePower_k(a, k);
  bigint aa = div_with_basePower_k(a, k);
  bigint a1 = mod_with_basePower_k(aa, k);
  bigint a2 = div_with_basePower_k(aa, k);

  bigint b0 = mod_with_basePower_k(b, k);
  bigint bb = div_with_basePower_k(b, k);
  bigint b1 = mod_with_basePower_k(bb, k);
  bigint b2 = div_with_basePower_k(bb, k);

  bigint a02 = add(a0, a2);
  bigint b02 = add(b0, b2);

  bigint v0 = multiply_toom_cook(a0, b0);
  bigint v1 = multiply_toom_cook(add(a02, a1), add(b02, b1));
  bigint v11 = multiply_toom_cook(subtract(a02, a1), subtract(b02, b1));
  bigint v2 = multiply_toom_cook(
      add(add(a0, scalar_multiply(a1, 2)), scalar_multiply(a2, 4)),
      add(add(b0, scalar_multiply(b1, 2)), scalar_multiply(b2, 4)));
  bigint vinf = multiply_toom_cook(a2, b2);

  bigint t11 = add(add(scalar_multiply(v0, 3), scalar_multiply(v11, 2)), v2);
  bigint t22 = add(v1, v11);

  bigint t1 = scalar_divide(t11, 6);
  bigint t2 = scalar_divide(t22, 2);

  bigint coeff0 = v0;
  bigint coeff1 = subtract(v1, t1);
  bigint coeff2 = subtract(t2, add(v0, vinf));
  bigint coeff3 = subtract(t1, t2);
  bigint coeff4 = vinf;

  bigint c = coeff0;
  c = add(c, coeff1.shift_left(k));
  c = add(c, coeff2.shift_left(2 * k));
  c = add(c, coeff3.shift_left(3 * k));
  c = add(c, coeff4.shift_left(4 * k));

  return c;
}

inline bigint multiply_toom_cook(const bigint &a, const bigint &b) {
  bigint abs_a = a;
  if (!abs_a.ip())
    abs_a.flipSign();

  bigint abs_b = b;
  if (!abs_b.ip())
    abs_b.flipSign();

  bigint result = multiply_toom_cook_core(abs_a, abs_b);

  if (a.ip() ^ b.ip())
    result.flipSign();
  return result;
}

```