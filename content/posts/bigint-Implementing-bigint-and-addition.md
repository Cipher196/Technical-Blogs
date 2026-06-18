---
date: '2026-06-18T11:21:37+05:30'
draft: false
title: 'BigInt in C++: Representation and Addition'
---


## What is BigInt?

Have you used C++? There, we have to specify the size of the integer we are going to use. It can be 32-bit, 64-bit, or 128-bit. However, the problem is that the size is fixed, and we cannot go beyond that limit.

So, what is the solution?

We can use a vector in C++. If we use `vector<uint64_t>`, then when the integer starts to overflow in `v[0]`, we can store the extra overflow data in `v[1]`, and so on. Note that we are storing the digits in reverse order.

In mathematics, if `A` is our BigInt and `b` is our base `2^64` (the range of values that `uint64_t` can store is from `0` to `2^64 - 1`), then:

```
A = v[0] + v[1]*b + v[2]*(b^2) + ...
```

Now the question is: how do we implement this in C++?

## Implementing BigInt

The first problem we have to tackle is how to store negative integers. Since we are using unsigned integers (`uint64_t`) inside our vector, we cannot store the sign inside the vector itself. Therefore, we need to store it separately.

The solution is to create a class for our BigInt:

```cpp
class bigint {
	private:
		bool isPositive;
		vector<uint64_t> digits;
	// And other code goes here ...
};
```

Note that we use a `bool` to store the sign since there are only two possible values: `+` and `-`.

## Implementing Addition

I am using the book "Modern Computer Arithmetic" by Richard P. Brent and Paul Zimmermann to learn how computers handle BigInts, and I will implement their algorithms.

Input:

```
A = va[0] + va[1]*b + va[2]*(b^2) + ...
B = vb[0] + vb[1]*b + vb[2]*(b^2) + ...
```

Output:

```
C = vc[0] + vc[1]*b + vc[2]*(b^2) + ...
```

Algo:

```
d = 0
s = a[0] + b[0]
c[0] = s mod b
d = s / b 

for i: 1 -> n-1
	s = a[i] + b[i] + d
	c[i] = s mod b
	d = s / b
	
return c
```

Note that `d = s / b`, and `d` cannot be greater than `1`, as the maximum value that `s` can attain is:

```
2*(2^64 - 1) + 1 = 2^65 - 1
```

Therefore, the carry can only be `0` or `1`.

## Code

### my_std.h
```
// my_std.h
#pragma once // Prevents this file from being included multiple times

#include <iostream>
#include <vector>
#include <string>
#include <algorithm>

#include <cstdint>
#include <cstddef>



// Explicitly bring only what you need into the global space of whoever includes this
using std::cout;
using std::cin;
using std::endl;
using std::vector;
using std::string;
```
This is becouse i dont want to include everthing every time.

### bigint.hpp
```
#pragma once

#include "my_std.h"

class bigint {
private:
  bool isPositive = true;
  // we are user base = uint64_t ( 2^64 )
  vector<uint64_t> digits;
  // A = digit[0] + digit[1]*base^1 + ....

  void multiply_add(uint64_t scalar, uint64_t add_value) {
    unsigned __int128 carry = add_value;

    for (size_t i = 0; i < digits.size(); i++) {
      unsigned __int128 curr = (unsigned __int128)digits[i] * scalar + carry;
      digits[i] = (uint64_t)curr;
      carry = curr >> 64;
    }

    while (carry > 0) {
      digits.push_back((uint64_t)carry);
      carry >>= 64;
    }
  }

  uint64_t divide_scalar(uint64_t divisor) {
    unsigned __int128 carry = 0;

    for (long long i = digits.size() - 1; i >= 0; --i) {
      unsigned __int128 curr = (carry << 64) | digits[i];
      digits[i] = (uint64_t)(curr / divisor);
      carry = curr % divisor;
    }

    while (digits.size() > 1 && digits.back() == 0) {
      digits.pop_back();
    }

    return (uint64_t)carry;
  }

  void zero_check() {
    if (digits.empty() || (digits.size() == 1 && digits[0] == 0)) {
      digits = {0};
      isPositive = true;
    }
  }

public:
  bool ip() const { return isPositive; }

  void flipSign() { isPositive=!isPositive; }

  const vector<uint64_t>& get_digits() const { return digits;}
  
  vector<uint64_t>& get_digit() { return digits;}


  bigint(long long val = 0) {
    if (val < 0) {
      isPositive = false;
      val = -val;
    } else {
      isPositive = true;
    }

    digits.push_back((uint64_t)val);
    zero_check();
  }

  bigint(const std::string &s) { from_string(s); }

  void from_string(const string &s) {
    digits.clear();
    isPositive = true;
    if (s.empty())
      return;

    size_t start = 0;
    if (s[0] == '-') {
      isPositive = false;
      start = 1;
    } else if (s[0] == '+') {
      start = 1;
    }

    digits.push_back(0);

    for (size_t i = start; i < s.length(); i++) {
      if (s[i] < '0' || s[i] > '9')
        continue;
      uint64_t digit_val = s[i] - '0';
      multiply_add(10, digit_val);
    }
    zero_check();
  }

  string to_string() const {
    if (digits.empty() || (digits.size() == 1 && digits[0] == 0)) {
      return "0";
    }

    bigint temp = *this;
    string result = "";

    while (!(temp.digits.size() == 1 && temp.digits[0] == 0)) {
      uint64_t remainder = temp.divide_scalar(10);
      result += std::to_string(remainder);
    }

    if (!isPositive) {
      result += "-";
    }

    reverse(result.begin(), result.end());
    return result;
  }
};
```

### arithmetic.hpp
```
#pragma once

#include "bigint.hpp"

inline bigint bigint_add(const bigint &a, const bigint &b) {
  bigint c(0);
  unsigned __int128 s = a.get_digits()[0] + b.get_digits()[0];
  c.get_digit()[0] = (uint64_t)s;
  uint64_t d = (uint64_t)(s >> 64);

  for (size_t i = 0; i < b.get_digits().size(); i++) {
    s = a.get_digits()[i] + b.get_digits()[i] + d;
    d = (uint64_t)(s >> 64);
    c.get_digit().push_back((uint64_t)s);
  }

  for (size_t i = b.get_digits().size(); i < a.get_digits().size(); i++) {
    s = a.get_digits()[i] + d;
    d = (uint64_t)(s >> 64);
    c.get_digit().push_back((uint64_t)s);
  }

  if (d)
    c.get_digit().push_back(d);
  return c;
}

inline bigint add(const bigint &a, const bigint &b) {
  bigint ans;
  if (a.ip() && b.ip()) {
    if (a.get_digits().size() >= b.get_digits().size()) {
      ans = bigint_add(a, b);
    } else {
      ans = bigint_add(b, a);
    }
    return ans;
  } else if (!a.ip() && !b.ip()) {
    if (a.get_digits().size() >= b.get_digits().size()) {
      ans = bigint_add(a, b);
    } else {
      ans = bigint_add(b, a);
    }
    ans.flipSign();
    return ans;
  } else {
    // TODO:  Implement substract functionality
    return ans;
  }
}
```
