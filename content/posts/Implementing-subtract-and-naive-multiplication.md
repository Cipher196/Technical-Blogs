---
date: '2026-06-21T18:58:35+05:30'
draft: false
title: 'Implementing Subtract and Naive Multiplication'
---

## Implementing Subtraction

When I was reading the book, it said:

> "The subtraction code is very similar. Step 3 simply becomes s ← ai − bi + d,
where d ∈ {−1, 0} is the borrow of the subtraction"

But the implementation became too tough for me to handle.

## Running into Problems

When I started implementing addition, I already needed four functions just to handle the basic logic, and I completely skipped the case where one number is positive and the other is negative.

When I tried to tie everything together with a few more subtraction functions, it just got messy. There were too many functions and too many edge cases to check and handle. I completely lost my way.

So, I got some help from AI. To be honest, I just copy-pasted the code because I didn't want to spend too much time rewriting addition and subtraction logic from scratch.

## Implementing Naive Multiplication

The next step is Multiplication, which is a much more interesting challenge. First, we should implement the method we all learned back in school. The book calls it naive multiplication (some refer to it as schoolbook multiplication).

#### Input
```text
A = va[0] + va[1]*b + va[2]*(b^2) + ...
B = vb[0] + vb[1]*b + vb[2]*(b^2) + ...
```

#### Output
```
C = vc[0] + vc[1]*b + vc[2]*(b^2) + ...
```

### Algorithm
```
C = A*vb[0]

for j: 1 -> n-1
	C = C + (b^j)*(A*vb[j]) // b is base
	
return C
```

I Successfully implemented this with help of some functions. Like scalar_multiplication for `A*vb[j]` type operation and shift for `A*(b^j)` type operation.

## What next
Next, I am going forward with implementing Karatsuba’s algorithm and learning multithreading in cpp for makeing this more fast.

## Code

#### my_std.cpp
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

#### bigint.cpp
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

  void flipSign() {
    if (!(digits.size() == 1 && digits[0] == 0)) {
      isPositive = !isPositive;
    }
  }

  const vector<uint64_t> &get_digits() const { return digits; }

  vector<uint64_t> &get_digit() { return digits; }

  void at(size_t i, uint64_t val) { digits[i] = val; }

  bigint(long long val = 0) {
    uint64_t absVal;
    if (val < 0) {
      isPositive = false;
      absVal = static_cast<uint64_t>(-(val + 1)) + 1;
    } else {
      isPositive = true;
      absVal = (uint64_t)val;
    }

    digits.push_back(absVal);
    zero_check();
  }

  bigint(const std::string &s) { from_string(s); }

  bigint(vector<uint64_t> v, bool sign) {
    digits = v;
    isPositive = sign;
  }

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

#### addition_and_subtraction.cpp
```
// addition_and_subtraction.hpp
#pragma once

#include "bigint.hpp"

// Value-based magnitude comparison helper (|a| >= |b|)
inline bool abs_geq(const bigint &a, const bigint &b) {
  if (a.get_digits().size() != b.get_digits().size()) {
    return a.get_digits().size() > b.get_digits().size();
  }
  for (long long i = (long long)a.get_digits().size() - 1; i >= 0; --i) {
    if (a.get_digits()[i] != b.get_digits()[i]) {
      return a.get_digits()[i] > b.get_digits()[i];
    }
  }
  return true; 
}

// Ensures a zero result doesn't accidentally retain a negative sign
inline void sanitize_zero_sign(bigint &n) {
  if (n.get_digits().size() == 1 && n.get_digits()[0] == 0) {
    if (!n.ip()) n.flipSign();
  }
}

// Pure magnitude addition: Computes |a| + |b| (Assumes |a| >= |b|)
inline bigint bigint_add(const bigint &a, const bigint &b) {
  bigint c;
  c.get_digit().clear();
  
  unsigned __int128 carry = 0;
  size_t a_size = a.get_digits().size();
  size_t b_size = b.get_digits().size();

  for (size_t i = 0; i < a_size; i++) {
    unsigned __int128 sum = (unsigned __int128)a.get_digits()[i] + carry;
    if (i < b_size) {
      sum += b.get_digits()[i];
    }
    c.get_digit().push_back((uint64_t)sum);
    carry = sum >> 64; // Captures upper 64-bit carry cleanly
  }

  if (carry) {
    c.get_digit().push_back((uint64_t)carry);
  }
  return c;
}

// Pure magnitude subtraction: Computes |a| - |b| (Assumes |a| >= |b|)
inline bigint bigint_sub(const bigint &a, const bigint &b) {
  bigint c;
  c.get_digit().clear();

  uint64_t borrow = 0;
  size_t a_size = a.get_digits().size();
  size_t b_size = b.get_digits().size();

  for (size_t i = 0; i < a_size; i++) {
    uint64_t b_val = (i < b_size) ? b.get_digits()[i] : 0;
    unsigned __int128 s = (unsigned __int128)a.get_digits()[i] - b_val - borrow;
    c.get_digit().push_back((uint64_t)s);
    borrow = (s >> 64) ? 1 : 0; // If underflow occurs, bit 64+ clears to 1s
  }

  // Pop trailing zeroes to prevent leading zero string outputs like "0001"
  while (c.get_digits().size() > 1 && c.get_digits().back() == 0) {
    c.get_digit().pop_back();
  }
  return c;
}

// Global signed addition handler
inline bigint add(const bigint &a, const bigint &b) {
  bigint ans;
  if (a.ip() == b.ip()) {
    // Both positive or both negative
    ans = abs_geq(a, b) ? bigint_add(a, b) : bigint_add(b, a);
    if (!a.ip()) ans.flipSign();
  } else {
    // Mixed signs: addition turns into subtraction rules
    if (abs_geq(a, b)) {
      ans = bigint_sub(a, b);
      if (!a.ip()) ans.flipSign();
    } else {
      ans = bigint_sub(b, a);
      if (!b.ip()) ans.flipSign();
    }
  }
  sanitize_zero_sign(ans);
  return ans;
}

// Global signed subtraction handler
inline bigint subtract(const bigint &a, const bigint &b) {
  bigint ans;
  if (a.ip() != b.ip()) {
    // Mixed signs: A - (-B) -> A + B  or  (-A) - B -> -(A + B)
    ans = abs_geq(a, b) ? bigint_add(a, b) : bigint_add(b, a);
    if (!a.ip()) ans.flipSign();
  } else {
    // Same signs: A - B or (-A) - (-B) -> B - A
    if (abs_geq(a, b)) {
      ans = bigint_sub(a, b);
      if (!a.ip()) ans.flipSign();
    } else {
      ans = bigint_sub(b, a);
      if (a.ip()) ans.flipSign(); // e.g. 10 - 20 = -10
    }
  }
  sanitize_zero_sign(ans);
  return ans;
}

```

#### basecaseMultiplication.cpp
```
// multiplication.hpp
#pragma once

#include "addition_and_subtraction.hpp"

inline bigint shift(const bigint &a, size_t i) {
  bigint c(vector<uint64_t>(a.get_digits().size() + i, 0), true);
  for (int j = 0; j < a.get_digits().size(); j++) {
    c.at(i + j, a.get_digits()[j]);
  }
  return c;
}

inline bigint scalar_multiply(const bigint &a, unsigned __int128 b) {
  unsigned __int128 d = 0;
  bigint c(a.get_digits(), true);
  for (int i = 0; i < a.get_digits().size(); i++) {
    d += a.get_digits()[i] * b;
    c.at(i, (uint64_t)d);
    d = d >> 64;
  }
  if (d > 0)
    c.get_digit().push_back(d);
  return c;
}

inline bigint shift_add(const bigint &c, const bigint &a, size_t i,
                        uint64_t bi) {
  bigint b = scalar_multiply(a, bi);
  bigint d = shift(b,i);
  bigint e = add(c,d);
  return e;
}

inline bigint multiply_naive(const bigint &a, const bigint &b) {
  bigint c(0);
  c = scalar_multiply(a, b.get_digits()[0]);
  for (int i = 1; i < b.get_digits().size(); i++) {
    c = shift_add(c, a, i, b.get_digits()[i]);
  }
  if(a.ip()^b.ip()) c.flipSign();
  return c;
}

```

#### main.cpp
```
#include "BasecaseMultiply.hpp"


int main() {
  string a, b;
  cout<<"Please provide first number to multiply: ";
  cin>>a;
  cout<<"Please provide second number to multiply: ";
  cin>>b;

  bigint x(a);
  bigint y(b);

  bigint z=multiply_naive(x,y);

  string c=z.to_string();

  cout<<"Their multiply is "<<c;
  return 0;
}

```