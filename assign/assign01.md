---
layout: mathjax
title: "Assignment 1: Big Integers"
---

Milestone 1: due **Wednesday, Jan 28th** by 11 pm

Milestone 2: due **Wednesday, Feb 4th** by 11 pm

Assignment type: **Pair**, you may work with one partner

# Overview

In this assignment, you will implement a `BigInt` C++ data type which
allows arbitrary-precision integer arithmetic.

## Milestones, grading criteria

Milestone 1 (15% of the assignment grade):

* Implementation of functions (15%)
  - constructors
  - destructor
  - `get_bit_vector()` member function
  - `git_bits(unsigned)` member function
  - `is_negative()` member function
  - `operator-()` member function (unary minus)
  - `to_hex()` member function (conversion to base-16)

<div class='admonition danger'>
  <div class='title'>Important!</div>
  <div class='content' markdown='1'>
Milestone 1 is intended as a warm-up, since you might not have
written C++ code in a while. For that reason, it is a very
lightweight milestone. Milestone 2 will require significantly
more work.
  </div>
</div>

Milestone 2 (85% of the assignment grade):

* Implementation of functions (65%)
  - `is_bit_set(unsigned)` member function
  - `operator+(const BigInt &)` member function (addition)
  - `operator-(const BigInt &)` member function (two-operand subtraction)
  - `operator*(const BigInt &)` member function (multiplication—challenging!)
  - `compare(const BigInt &)` member function
  - `operator<<(unsigned)` member function (left shift)
  - `operator/(const BigInt &)` member function (division—challenging!)
  - `to_dec()` member function (conversion to base-10—challenging!)
* Comprehensiveness and quality of your unit tests (10%)
* Design and coding style (10%)

<div class='admonition danger'>
  <div class='title'>Important!</div>
  <div class='content' markdown='1'>
Multiplication (`operator*`) is somewhat challenging, and is worth
at most 1.5% of the assignment grade.  Division (`operator/`) and conversion to
base-10 (`to_dec()`) are even more challenging, and each is worth at most 0.75% of the
overall assignment grade. You should work on all of these only after implementing
and testing the other member functions.
  </div>
</div>

## Getting started

To get started, download [csf\_assign01.zip](csf_assign01.zip), which contains the
skeleton code for the assignment, and then unzip it.

Note that you can download the zipfile from the command line using the `curl` program:

```bash
curl -O https://jhucsf.github.io/spring2024/assign/csf_assign01.zip
```

We strongly recommend using a [Git](https://git-scm.com/) repository to store
your code. Please do not use a public repository, however.

To compile the test program, run the commands

```
make depend
make
```

Note that `make depend` automatically determines header file dependencies.

To run the unit tests:

```
./bigint_tests
```

# Machine integers, arbitrary-precision integers

As you know, the "built-in" C integer data types (`int`, `uint64_t`, etc.) have
a finite number of bits, and so can represent only a finite set of possible values.
Therefore, they don't model mathematical integers with complete fidelity.
However, they can be building blocks for the creation of an arbitrary-precision
integer data type, where there is no fixed limit on the range of values that can
be represented (other than machine limitations such as the amount of memory that
can be addressed by the program.)

In this assignment, you will implement the `BigInt` C++ data type. It should have
two member variables:

* a `std::vector` containing `uint64_t` values representing the magnitude
  of the `BigInt` value
* a `bool` representing whether or not the `BigInt` value is negative

The vector of `uint64_t` elements is the "bit string". If you think about the
magnitude of a `BigInt` value as an arbitrary integer, the bit string specifies
that value in base 2. The first element contains the least-significant 64 bits
of the bit string, the second element contains the next 64 bits of the bit string,
etc. For example, consider the integer value $$2^{64} = 18,446,744,073,709,551,616$$.
This value is represented in base 2 as

> $$10000000000000000000000000000000000000000000000000000000000000000$$

which is one followed by 64 zeroes.  In hexadecimal (base 16), this value is
represented as

> $$10000000000000000$$

which is 1 followed by 16 zeroes.  Note that each hex "digit" represents
4 bits (base-2 "digits".)

Since a `uint64_t` value contains exactly 64 bits, the value $$2^{64}$$ could
be represented as two `uint64_t` values: the less-significant would have
the value 0 (representing the lowest 64 bits, all of which are 0), and
the more-significant would have the value 1 (meaning that there is a 1
in the $$2^{64}$$ place in the base-2 representation.

In general, because a `BigInt` object contains a vector of `uint64_t` values,
with no arbitrary limit on the number of elements the vector can hold,
it can contain as many `uint64_t` values as it needs in order to represent any integer
magnitude.

Since integers can be negative or non-negative, a `BigInt` will also have a `bool`
value which is set to true if the `BigInt` is negative, false if non-negative.
This makes `BigInt` a sign/magnitude integer representation.

<div class='admonition info'>
  <div class='title'>Important!</div>
  <div class='content' markdown='1'>
`BigInt` values do not use two's complement. Two `BigInt` values
with the same magnitude and opposing signs will differ only in the
`bool` value indicating their sign. The representations of their
magnitudes will be exactly the same.
  </div>
</div>

## Your tasks

You have two general tasks:

1. implement the member functions of `BigInt`
2. implement additional unit tests so that all member functions are
   tested thoroughly

The member functions have detailed documentation comments in `bigint.h` which
describe how they should behave. There are also provided unit tests in
`bigint_tests.cpp` which further clarify the intended behavior of the
member functions. Note that while the provided unit tests are useful,
and are a good starting point for validating the implementation of the
member functions, there are important cases that they don't test.
So, it will be critical for you to write additional tests.

As documented above, there are two milestones. Milestone 1 tests only
a limited subset of member functions, and the unit tests you write
aren't part of the grading criteria for Milestone 1. Milestone 2
requires you to implement the rest of the member functions, and also
requires you to have comprehensive unit tests for all functions (including
the ones tested in Milestone 1.)

## Restrictions

You must adhere to the following restrictions in completing the assignment.

**Only standard library classes/functions can be used.** You aren't
allowed to use any external libraries in your implementation.
However, you are free (and encouraged) to use any functionality in the
C++ (or C) standard library.

**Only 64-bit and smaller data types can be used.** You aren't allowed
to use any data type whose representation is larger than 64 bits.
(However, you won't need to, and it wouldn't make the job easier
in any case.)

**Original code only.** It should go without saying that all of the code
you submit must be your original work. Copying code from an external
source would be a violation of academic ethics.

## Recommendations and hints

This section has further recommendations and hints, in no particular order.

### Don't use floating point operations

You will not need to use floating point (`float` or `double`) operations.
If you have a problem that you think requires floating point, there is
definitely a way to solve the problem without floating point. For example,
if you need to compute an arbitrary power of 2, don't use the `pow` function.
Instead, left shift the value `1UL` the appropriate number of places.
E.g., `1UL << n` will be equal to $$2^{n}$$ as long as `n` is in the range
$$0 \ldots 63$$.

This assignment is about integers.

### Do use `auto`

We encourage you to use `auto` to infer types, since it makes code
cleaner and more readable. For example, to iterate through a
vector `v`:

```c++
for (auto i = v.begin(); i != v.end(); ++i) {
  // do something with *i
}
```

### Helper functions

We recommend adding private member functions as necessary to support the implementation
of the required public member functions. For example, the reference implementation
defines the following private member functions:

```c++
bool is_zero() const;
static BigInt add_magnitudes(const BigInt &lhs, const BigInt &rhs);
static BigInt subtract_magnitudes(const BigInt &lhs, const BigInt &rhs);
static int compare_magnitudes(const BigInt &lhs, const BigInt &rhs);
BigInt div_by_2() const;
```

You aren't required to implement these specific private member functions
(or any private member functions for that matter.)

### Addition of magnitudes

You should implement addition of magnitudes using the "grade school" algorithm.
Think about each element of the vector of `uint64_t` values in a `BigInt`
object as a "digit". The `uint64_t` values just happen to be digits in base
$$2^{64}$$.

Start by adding the "digits" in the rightmost column to compute the rightmost
digit of the result. Note that the computed "digit" is correct modulo $$2^{64}$$.
If the addition of the column values overflows, you will need to carry a 1 into
the next column (to the left.) This is exactly analogous to base-10 addition.
For example:

```
  16
+  7
----
  ??
```

When adding the digits in the right column ($$6 + 7$$), the sum is $$13$$,
which modulo the base (10) is 3. So, the rightmost digit of the sum is 3.
However, the addition overflows, since $$13$$ can't be represented as a
single base-10 digit. So, we will need to carry a 1 into the next
column.

To detect overflow when adding `uint64_t` "digits", you can use the standard
trick for detecting unsigned integer overflow:

```c
sum = a + b;
if (sum < a) {
  // overflow occurred
}
```

<div class='admonition danger'>
  <div class='title'>Warning</div>
  <div class='content' markdown='1'>
An additional way that column values could overflow is if a 1 is being
carried in from the previous column. As a base-10 analogy, adding
$$7 + 2$$ wouldn't normally cause an overflow. However, if a 1 is carried
in from the previous column, then $$7 + 2$$ *will* cause an overflow,
since $$7 + 2 + 1 = 10$$.  You will need to think carefully about how this
type of situation should be handled when dealing with `uint64_t` "digits".
  </div>
</div>

One concern when implementing addition is that the two `BigInt` values being
added might have different numbers of `uint64_t` values in their bit string
vectors. The `get_bits()` member function can be helpful for retrieving
part of a bit string without needing to worry about how many elements the
vector actually has.

### Addition with negative values

You can handle negative values as follows:

* If two nonnegative values are added, or if two negative values are added,
  then the result has a magnitude that is the sum of the magnitudes of
  the operands, and the sign will be the same as the sign of both operands.
* In a "mixed" addition (one operand is non-negative and one operand is
  negative), the magnitude of the result is the difference $$a-b$$, where $$a$$
  is the magnitude of the value with the larger magnitude, and $$b$$ is the
  magnitude of the value with the smaller magnitude. The sign of the result
  is the same as the sign of the operand with the larger magnitude.

You will probably find it useful to create helper functions to add magnitudes
and subtract magnitudes.

### Subtraction

The two-operand subtraction operator `operator-(const BigInt &)` can be
implemented by adding the negation of the right-hand operand to the
value of the left hand operand. In other words, you can compute

$$a - b$$

as

$$a + -b$$

### Subtraction of magnitudes

Subtraction of magnitudes should always involve subtracting a smaller magnitude
from a larger magnitude. As with adding magnitudes, you can use the "grade
school" algorithm: start with the "digits" in the rightmost column. Each time
a column difference requires subtracting a larger value from a smaller
value, you will need to borrow 1 from the next column.

Again, as a base-10 analogy, in the subtraction

```
  16
-  7
----
  ??
```

the rightmost column difference $$6-7$$ requires borrowing 1 from the next
column since $$7$$ is greater than $$6$$.

### Left shift

The left shift operator (`operator<<(unsigned)`) will require some careful thought.
Here are some ideas that could make it simpler to implement:

* If the number of bits shifted is a multiple of 64, that is an "easy" case,
  since each `uint64_t` value in the result's bit string will be identical
  to a corresponding `uint64_t` value in the original value's bit string.
* The number of bits shifted is really only "interesting" modulo 64.
  For example, shifting left by 1 bit, shifting left by 65 bits,
  shifting left by 129 bits, etc., are very similar cases. The only
  way in which they differ is how many `uint64_t` values worth of 0
  bits are incorporated into the least-significant part of the result's
  bit string.

### Conversion to hexadecimal (`to_hex()`)

Converting a `BigInt` value to a hexadecimal string is fairly straightforward
if you use a `std::stringstream` object to help with the formatting of the
`uint64_t` values as hexadecimal. The `std::hex`, `std::setfill`, and `std::setw`
manipulators will likely be helpful.

Don't forget that a negative value needs a leading `-` sign.
Also, make sure there are no unnecessary leading `0` digits. (Although,
the value equal to $$0$$ should be coverted to the string "`0`".)

### Multiplication

One way to implement multiplication is to break down one of the operands
into powers of 2, multiply the other operand by each of those powers of 2,
and add those partial products together.

For example, $$37 = 32 + 4 + 1 = 2^{5} + 2^{2} + 2^{0}$$. The product
$$37 \times m$$ could therefore be computed as

$$2^{5} \times m + 2^{2} \times m + 2^{0} \times m$$

Note that left-shifting a value by $$n$$ is the same as multiplying it
by $$2^{n}$$. So, if you have implemented the `is_bit_set()`, `operator<<`,
and `operator+` member functions, you should have everything you need to implement
multiplication.

You will need to think about how the sign of the result relates to the
signs of the operands.

### Division

A simple way to implement division is using binary search. In computing
$$q = n / m$$, where $$n$$ is the dividend and $$m$$ is the divisor,
we can note that the quotient $$q$$ will be in the range $$0 \ldots n$$, inclusive.
So, initially, $$0$$ is the lower bound of the range, and $$n$$
is the upper bound of the range. Repeatedly, we can choose a
possible value of $$q$$ midway between the lower and upper bounds.
Depending on whether or not $$q \times m$$ is greater than $$n$$,
we can revise either the lower or upper search bound. Eventually,
the range should collapse to a single value, which is the computed
quotient $$q$$.

Note that this is not a particularly fast way to implement division, but
it is adequate for this assignment.

When choosing a value midway between the current lower and upper bounds,
it's useful to have a "divide by 2" operation. Note that a right shift
by 1 bit is effectively a division by 2.

The intended semantics of division is that the computed quotient has the
largest magnitude such that multiplying it by the divisor yields a product
whose magnitude is not greater than the dividend's magnitude.

You will need to think about how the sign of the result relates to the
signs of the operands.

### Conversion to decimal (`to_dec()`)

One algorithm for converting an integer to decimal (base 10) is repeatedly
dividing by 10: the remainder of each division will yield one digit, in order
from least significant to most significant.

Note that you could make this process more efficient by generating more than
one digit at a time. For example, repeatedly dividing by 100 would yield
two base-10 digits at a time. You should think about how many base-10 digits
can be produced by each iteration of this process.

As with `to_hex()`, `std::stringstream` will be helpful. Also, don't forget
to prepend a leading `-` sign if the value is negative.

### Writing tests

Your unit tests should test each required member function thoroughly.
We recommend that you implement your tests by adding additional
test functions to `bigint_tests.cpp`, rather than adding new tests
to the provided test functions.

Your tests should try to create "interesting" scenarios for each tested
member function. This includes things like

* zero vs. non-zero values
* negative vs. non-negative values (and combinations thereof)
* smaller vs. larger values
* etc.

A good mindset for testing is that you are an adversary of your own
code, i.e., you are trying to make it break.

One approach that can be helpful is to write a program to generate
test cases. Both [Python](https://www.python.org/) and
[Ruby](https://www.ruby-lang.org/en/) support arbitrary-precision
integers as part of the core language. So, you can write a script
in either of those languages which computes arbitrarily large
integer values, does an operation on them, and then prints code
of a test case to check whether `BigInt` produces the same result
when doing the same computation. Some of the test cases in the
provided unit tests were generated by a Ruby script. For example,
here is an automatically-generated test for subtraction:

```c++
{
  BigInt left(
      {0x94e439a254295b2fUL, 0xc02d6dc0be0efef4UL, 0xe5156c9d912b61f2UL,
       0xb82729123ce1051eUL, 0x1d2c69a0ed4011c3UL, 0xf13f35779fd54911UL,
       0x15056f71d40516eaUL, 0xdb571f43f9416bdeUL, 0x7e21086e7df7095UL,
       0x797275a8e7538b0aUL, 0x18a6284e20e7893aUL});
  BigInt right(
      {0x9161ea05eb48510dUL, 0xb7a476402ef52acaUL, 0xdf96be7a926695adUL,
       0x53e8bc19a9c14029UL, 0xf87ee595e422d5f0UL, 0x72dd209be1d990cbUL,
       0xb991581205507625UL, 0x77bbceb930f0c50eUL, 0x862b240a5ee05327UL,
       0x44af5ae70f9c63b6UL, 0x30UL}, true);
  BigInt result = left - right;
  check_contents(result, {0x264623a83f71ac3cUL, 0x77d1e400ed0429bfUL,
                          0xc4ac2b182391f7a0UL, 0xc0fe52be6a24548UL,
                          0x15ab4f36d162e7b4UL, 0x641c561381aed9ddUL,
                          0xce96c783d9558d10UL, 0x5312edfd2a3230ecUL,
                          0x8e0d349146bfc3bdUL, 0xbe21d08ff6efeec0UL,
                          0x18a6284e20e7896aUL});
  ASSERT(!result.is_negative());
}
```

Note that automatically-generated test cases are a complement to
hand-written test cases, not a substitute for them.

## Submitting

Before submitting each milestone, you should edit `README.txt`
to include information about

* the contributions of each team member
* any interesting implementation details or things we should know
  about your submission

You can use the command `make solution.zip` to create a zipfile with
the required submission files.

Upload `solution.zip` to either **Assignment 1 MS1** or **Assignment 1 MS2**
on [Gradescope](https://www.gradescope.com/) as appropriate.
