---
layout: mathjax
title: "Assignment 2: Image Processing"
---

**Due**:

* Milestone 1 due **Friday, Feb 13th** by 11 pm
* Milestone 2 due **Friday, Feb 20th** by 11 pm
* Milestone 3 due **Friday, Feb 27th** by 11 pm

This is a **pair** assignment, so you may work with one partner.

*Update 2/3* — fixed some details so they are relevant to the
this semester's image transformations

<div class='admonition danger'>
  <div class='title'>Warning!</div>
  <div class='content' markdown='1'>
Assembly language programming is challenging! Make sure
you start each milestone as soon as possible, work steadily, and
ask questions early.  Also, writing unit tests and using `gdb` to
examine the detailed behavior of code under test will be critical
to successful implementation of the assembly language functions.
  </div>
</div>

## Overview

In this assignment, you will implement transformations on image files,
using both C (in Milestone 1) and x86-64 assembly language (in Milestones 2 and 3.)

### Milestones

In Milestone 1, you are required to implement all of the image transformations
in C. You are also required to implement comprehensive unit tests for all of the
helper functions you use to implement the transformations. The intent is that
your assembly language code in Milestones 2 and 3 will implement the same
helper functions, and your unit tests will help you gain confidence in their
correctness.

In Milestone 2, you are required to implement the
[`squash`](#the-squash-transformation) and
[`color_rot`](#the-color_rot-transformation)
transformations in assembly language. We expect you to have comprehensive unit tests
for the assembly language implementations of your helper functions. (In theory you can
just use the ones you implemented in Milestone 1.) Note that we will not officially
grade the quality and comprehensiveness of your unit tests until Milestone 3.

In Milestone 3, you will implement the
[`blur`](#the-blur-transformation) blur transformation and
[`expand`](#the-expand-transformation)
transformations.

Note that in each milestone, we expect all of the tests executed
by your unit test program to pass. For Milestone 2 in particular, you can
comment out calls to test functions that aren't related to
helper functions needed for Milestone 2. For example, for Milestone 2 your `imgproc_test.c`'s
`main` function might have code similar to the following:

```c
TEST( test_get_r );
TEST( test_get_g );
TEST( test_get_b );
TEST( test_get_a );
TEST( test_make_pixel );
//TEST( test_get_neighbor );
```

The tests for `get_r`, `get_g`, `get_b`, `get_a`, and `make_pixel`
are enabled because they are all test functions involved
in the implementations of the transformations
required for MS2. The test for `get_neighbor` 
is commented out because it is used in the `blur`
transformation, which is not part of the requirements for MS2.

### Non-functional requirements

In Milestones 2 and 3, you will be writing assembly language functions.
You **must** write these "by hand", and your assembly code must have
very detailed code comments explaining the purpose of each assembly language
instruction.

<div class='admonition tip'>
  <div class='title'>Assembly comments</div>
  <div class='content' markdown='1'>
A good rule of thumb is that *every* assembly language instruction
should have a comment describing what it is intended to do.
  </div>
</div>

It is **not** allowed to generate assembly
code using a C compiler and submit this as your own code. We will assign
a grade of 0 to any submissions containing compiler-generated code
where hand-written assembly language is expected.

Your submission for each milestone should include a `README.txt` file
describing how you and your partner divided the work, and letting
us know about any interesting implementation details. If there is
functionality you weren't able to get working completely, this is
a good place to mention that.

We expect you to follow the [style guidelines](style.html).
However, the expectations for function length will be relaxed considerably
for your assembly language code. It is not unusual for an assembly language
function to have 100 or more lines of code. In the reference solution,
the longest function was about 149 lines, although there was extensive use of
comments and whitespace to improve readability.

Of course, you should strive to make your assembly language functions
as simple and readable as possible.

We expect your code to be free of memory errors. You should use
`valgrind` to test your code to make sure there are no uses
of uninitialized variables, out of bounds memory reads or writes,
etc.  This applies to both your C code and your assembly code.

<div class='admonition info'>
  <div class='title'>Note</div>
  <div class='content' markdown='1'>
If any test assertions fail when you run the unit test program,
that may lead to a memory leak due to one or more test objects
not being freed. This situation does not count as a memory leak
in your code. (However, you should definitely fix the bug that
is causing the test assertion to fail.)
  </div>
</div>

### Grading breakdown

Milestone 1: 30%

* Implementation of C image transformation functions: 12.5%
* Unit testing of helper functions: 12.5%
* Design/coding style of C functions: 5%

Milestone 2: 35%

* Functional correctness of `imgproc_squash` and `imgproc_color_rot`: 30%
* Design/coding style of assembly functions: 5%

Milestone 3: 35%

* Functional correctness of `imgproc_blur`: 10%
* Functional correctness of `imgproc_expand`: 10%
* Unit testing of helper functions: 10%
* Design/coding style of assembly functions: 5%

## Getting started

To get started, download [csf\_assign02.zip](csf_assign02.zip) and
unzip it.

You will implement the functions in `c_imgproc_fns.c` (Milestone 1)
and `asm_imgproc_fns.S` (Milestones 2 and 3.) You will also add
prototypes for helper functions to `imgproc.h` and implement additional
unit tests in `imgproc_tests.c`.

## Image processing

This section describes the image format and the image transformations
you will implement.

### struct Image

An instance of the `struct Image` data type represents a grid of
pixels, where each pixel has 8-bit red, green, and blue color
component values, as well as an 8-bit alpha value.

The `struct Image` type is defined as follows (in `image.h`):

```c
struct Image {
  int32_t width;
  int32_t height;
  uint32_t *data;
};
```

The `width` and `height` fields define the width and height of an image,
in pixels. The `data` field is a pointer to a dynamically-allocated array
of `uint32_t` values, each one representing one pixel. The pixels are stored
in row-major order, starting with the top row of pixels.

A color is represented by a `uint32_t` value as follows:

* Bits 24-31 are the 8 bit red component value, ranging from 0–255
* Bits 16-23 are the 8 bit green component value, ranging from 0–255
* Bits 8–15 are the 8 bit blue component value, ranging from 0–255
* Bits 0–7 are the 8 bit alpha value, ranging from 0–255

This pixel data format is known as "RGBA".

The alpha value of a color represents its opacity, with 255 meaning
"fully opaque" and 0 meaning "fully transparent".

### Image transformation functions

You will implement the following image transformation functions in both
C and assembly language:

```c
void imgproc_squash( struct Image *input_img,
                     struct Image *output_img,
                     int32_t xfac, int32_t yfac );
void imgproc_color_rot( struct Image *input_img,
                        struct Image *output_img );
void imgproc_blur( struct Image *input_img,
                   struct Image *output_img,
                   int32_t blur_dist );
void imgproc_expand( struct Image *input_img,
                     struct Image *output_img );
```

These functions are declared in `imgproc.h`, and each one has a detailed API
comment describing its function, the meaning of the parameters, and the
meaning of the return value (for the non-`void` functions.)

### The `squash` transformation

The `squash` transformation transforms the input image by shrinking it
both horizontally and vertically by (potentially different) integer factors.

In the following explanation, we'll refer to the horizontal factor as
$$a$$ and the vertical factor as $$b$$. (In the code, they are referred to
by the names `xfac` and `yfac`.)

Each pixel of the output image is determined by sampling a pixel from
the input image. Specifically, the output pixel at row $$i$$ and column $$j$$
copies the input pixel at row $$bi$$ and column $$aj$$. This
is equivalent to keeping only the pixels whose row index is divisible
by $$b$$ and whose column index is divisible by $$a$$.
For example, consider the image below where each letter corresponds to a pixel:

```
XAAAYBBB
AAAABBBB
ZCCCWDDD
CCCCDDDD
```

If the user specifies $$a = 4$$ and $$b = 2$$, only pixels in rows 0 and
2 with columns 0 and 4 are sampled. The resultant image is:

```
XY
ZW
```

If the width and height of the original image are $$w$$ and $$h$$,
respectively, the output image will have width equal to $$\lfloor w / a \rfloor$$
and height equal to $$\lfloor h / b \rfloor$$.

Example (note that the transformed image has been enlarged, click to see it 
at its correct size):

Original image | Transformed image<br> (xfac 8, yfac 4)
:------------: | :---------------:
<a href="img/ingo.png"><img style="width: 20em;" alt="original cat image" src="img/ingo.png"></a> | <a href="img/ingo_squash_8_4.png"><img style="width: 10em;" alt="squashed cat image" src="img/ingo_squash_8_4.png"></a>

### The `color_rot` transformation

The `color_rot` transformation transforms the input image by shifting
around the color components' values in each pixel.

Each pixel of the output image should have its color components determined by 
taking the previous color component's value and applying it to that pixel. The 
old pixel's red component value will be used for the new pixel's green component 
value, the old pixel's green component value will be used new pixel's blue 
component value and the old pixel's blue component value will be used new 
pixel's red component value. The alpha value each output pixel should be 
identical to the corresponding input pixel.

For example, if a pixel had the hex value `0xAABBCCDD`, the transformed pixel 
would become `0xCCAABBDD` in the same location. (Recall that the red color channel
occupies the most-significant 8 bits of a pixel, and the alpha channel
occupies the least-significant 8 bits of a pixel.)

Example:

Original image | Transformed image<br>
:------------: | :---------------:
<a href="img/ingo.png"><img style="width: 20em;" alt="original cat image" src="img/ingo.png"></a > | <a href="img/ingo_color_rotated.png"><img style="width: 20em;" alt="color rotated cat image " src="img/ingo_color_rotated.png"></a>




### The `blur` transformation

The `blur` transformation transforms the input image using a blur effect.

Each pixel of the output image should have its color components
determined by taking the average of the color components of pixels
within `blur_dist` number of pixels horizontally and vertically from
the pixel's location in the original image. For example, if
`blur_dist` is 0, then only the original pixel is considered, and the
the output image should be identical to the input image. If `blur_dist`
is 1, then the original pixel and the 8 pixels immediately surrounding
it would be considered, etc. Pixels positions not within the bounds of
the image should be ignored: i.e., their color components aren't
considered in the computation of the result pixels.

The alpha value each output pixel should be identical to the
corresponding input pixel.

Averages should be computed using purely integer arithmetic with
no rounding.

Example:

Original image | Transformed image<br>(blur distance 5)
:------------: | :---------------:
<a href="img/ingo.png"><img style="width: 20em;" alt="original cat image" src="img/ingo.png"></a > | <a href="img/ingo_blur_5.png"><img style="width: 20em;" alt="blurred cat image " src="img/ingo_blur_5.png"></a>

### The `expand` transformation

The `expand` transformation doubles the width and height of the image.

Let's say that there are $$n$$ rows and $$m$$ columns of pixels in the
input image, so there are $$2n$$ rows and $$2m$$ columns in the output
image.  The pixel color and alpha value of the output pixel at row $$i$$ and column
$$j$$ should be computed as follows.

If both $$i$$ and $$j$$ are even, then the color and alpha value of the output
pixel are exactly the same as the input pixel at row $$i/2$$ and column $$j/2$$.

If $$i$$ is even but $$j$$ is odd, then the color components and alpha value
of the output pixel are computed as the average of those in the input pixels
in row $$i/2$$ at columns $$\lfloor j/2 \rfloor$$ and $$\lfloor j/2 \rfloor + 1$$.

If $$i$$ is odd and $$j$$ is even, then the color components and alpha value
of the output pixel are computed as the average of those in the input pixels
in column $$j/2$$ at rows $$\lfloor i/2 \rfloor$$ and  $$\lfloor i/2 \rfloor + 1$$.

If both $$i$$ and $$j$$ are odd then the color components and alpha value
of the output pixel are computed as the average of the input pixels

1. At row $$\lfloor i/2 \rfloor$$ and column $$\lfloor j/2 \rfloor$$
2. At row $$\lfloor i/2 \rfloor$$ and column $$\lfloor j/2 \rfloor + 1$$
3. At row $$\lfloor i/2 \rfloor + 1$$ and column $$\lfloor j/2 \rfloor$$
4. At row $$\lfloor i/2 \rfloor + 1$$ and column $$\lfloor j/2 \rfloor + 1$$

Note that in the cases where either $$i$$ or $$j$$ is odd, it is not
necessarily the case that either row $$\lfloor i/2 \rfloor + 1$$ or
column $$\lfloor j/2 \rfloor + 1$$ are in bounds in the input image.
Only input pixels that are properly in bounds should be incorporated into
the averages used to determine the color components and alpha value
of the output pixel.

Averages should be computed using purely integer arithmetic with
no rounding.

Example (note that the transformed image is twice the size of the
original in each dimension, click to see it full size):

Original image | Transformed image
:------------: | :---------------:
<a href="img/ingo.png"><img style="width: 20em;" alt="original cat image" src="img/ingo.png"></a > | <a href="img/ingo_expand.png"><img style="width: 20em;" alt="expanded cat image " src="img/ingo_expand.png"></a>

## `c_imgproc` and `asm_imgproc` programs

The `c_imgproc` and `asm_imgproc` programs apply one of the image transformations
to an input image (or, in the case of the `composite` transformation, two input images),
and write the result to an output image file.  The `c_imgproc_main.c` source file
implements both of these programs. The only difference between `c_imgproc` and
`asm_imgproc` is whether the image transformations are implemented in C
(`c_imgproc_fns.c`) or x86-64 assembly language (`asm_imgproc_fns.S`.)

To run these programs:

<div class='highlighter-rouge'><pre><code>./c_imgproc <i>transformation</i> <i>input.png</i> <i>output.png</i> [<i>args...</i>]
./asm_imgproc <i>transformation</i> <i>input.png</i> <i>output.png</i> [<i>args...</i>]
</code></pre></div>

In these commands:

* <code class='highlighter-rouge'><i>transformation</i></code> is the name of the
  image transformation to perform
* <code class='highlighter-rouge'><i>input.png</i></code> is the name of the input
  image file
* <code class='highlighter-rouge'><i>output.png</i></code> is the name of the output
  image file to write
* <code class='highlighter-rouge'>[<i>args...</i>]</code> represents additional arguments
  needed by the transformation; for example, the `blur` transformation needs an
  integer blur distance as an additional argument

For example, to run the `blur` transformation with a blur distance of 3
using the `c_imgproc` program:

```text
mkdir -p actual
./c_imgproc blur input/ingo.png actual/c_ingo_blur_3.png 3
```

The commands in this example would apply the `blur` transformation with a blur
distance of 3 on the input image `input/ingo.png` to produce the output image
file `actual/c_ingo_blur_3.png`.

## Unit tests, helper functions

The source file `imgproc_tests.c` is a unit test program that you should use to
test the functions in `c_imgproc_fns.c` and `asm_imgproc_fns.S`.

The starter code has some very basic tests for the API functions implementing the
various image transformations. However, you will need to write unit tests for
your *helper functions*. The idea is that your assembly language code will implement
exactly the same helper functions as your C code, and having a comprehensive set
of unit tests for these helper functions will allow you to get your assembly code
working incrementally by implementing the helper functions one at a time.

<div class='admonition info'>
  <div class='title'>Tip</div>
  <div class='content' markdown='1'>
Having a good set of unit tests for your helper functions is essential for being
able to make steady progress towards getting your assembly code to work in
Milestones 2 and 3.
  </div>
</div>

You are free to implement whatever helper functions make sense. Some of
the helper functions defined in the reference implementation are:

```c
uint32_t get_r( uint32_t pixel );
uint32_t get_g( uint32_t pixel );
uint32_t get_b( uint32_t pixel );
uint32_t get_a( uint32_t pixel );
uint32_t make_pixel( uint32_t r, uint32_t g, uint32_t b, uint32_t a );
int32_t compute_index( struct Image *img, int32_t row, int32_t col );
int get_neighbor( struct Image *img, int32_t row, int32_t col, uint32_t *out );
uint32_t blur_pixel( struct Image *img, int32_t row, int32_t col, int32_t blur_dist );
```

## Image tests

The provided script `run_all.sh` runs your `c_imgproc` or `asm_imgproc` program
on some example input images and checks whether a correct output image
is produced. To run it:

```bash
# test the C implementations of the image transformations
./run_all.sh c
# test the assembly implementations of the image transformations
./run_all.sh asm
```

## Hints and tips

### x86-64 tips

Here are some x86-64 assembly language tips and tricks in no particular order.

Callee-saved registers are your best option to serve as local variables
in your assembly language functions. The callee-saved registers are
`%r12`, `%r13`, `%r14`, `%r15`, `%rbx`, and `%rbp`. If you are going
to store data in a callee-saved register, make sure that you use `pushq`
to save its value at the beginning of the function, and `popq` to restore
its value at the end of the function. (The `popq` instructions must be in
the opposite order as the `pushq` instructions.)

If you run out of callee-saved registers, then you can use memory in the
stack frame to store local variables. We *highly* recommend using an ABI-compliant
stack frame to reserve memory for local variables, since that will allow
`gdb` to properly recognize the functions on the call stack. To do so,
your function's prologue code should look like this:

<div class='highlighter-rouge'><pre><code>pushq %rbp
movq %rsp, %rbp
subq $N, %rsp
<i>...push callee-saved registers...</i>
</code></pre></div>

This prologue will reserve *N* bytes of memory in the stack frame that
you can use for local variables. Note that all local variables in memory
should be accessed at *negative* offsets from `%rbp`. For example, if you
reserved 16 bytes for local variables, and you need space for four 4-byte
variables, you can refer to them as `-4(%rbp)`, `-8(%rbp)`, `-12(%rbp)`,
and `-16(%rbp)`.

Don't forget that the amount by which the stack pointer is
changed needs to be an odd multiple of 8 (so 8, or 24, or 40, etc.),
and that each `pushq` subtracts 8 from `%rsp`.
Also, don't  forget to pop back the original values of any saved
callee-saved registers, including `%rbp`. In general, your function
epilogue code should look like

<div class='highlighter-rouge'><pre><code><i>...pop callee-saved registers...</i>
addq $N, %rsp
popq %rbp
</code></pre></div>

Don't forget that you need to prefix constant values with `$`.  For example,
if you want to set register `%r10` to 16, the instruction is

```
movq $16, %r10
```

and not

```
movq 16, %r10
```

When calling a function, the stack pointer (`%rsp`) must contain an address
which is a multiple of 16.  However, because the `callq` instruction
pushes an 8 byte return address on the stack, on entry to a function,
the stack pointer will be "off" by 8 bytes.  You can subtract 8 from
`%rsp` when a function begins and add 8 bytes to `%rsp` before returning
to compensate.  Pushing an odd number of callee-saved registers also works,
and has the benefit that you can then use the callee-saved registers freely
in your function. Sometimes you may need to subtract 8 from `%rsp` even
if your function doesn't allocate storate for any local variables in memory,
just to ensure that `%rsp` is aligned correctly.

We *strongly* recommend that you have a comment in each function explaining
how it uses callee-saved registers and (if relevant) stack memory, since these are
the equivalent of local variables in assembly code. For example,
here is a comment taken from the implementation of the `imgproc_squash`
function in the reference solution:

<a name='register-memory-comment'>

```c
/*
 * Register use:
 *   %r12 - saved pointer to input Image
 *   %r13 - saved pointer to output Image
 *   %r14d - row loop counter (i)
 *   %r15d - column loop counter (j)
 *   %ebx - saved pixel value
 *
 * Memory use:
 *   -8(%rbp) - saved xfac
 *   -4(%rbp) - saved yfac
 */
```

Recall that your assembly language code must have detailed comments
explaining each line of assembly code. The following example
function illustrates the level of commenting that we expect to see:

```
/*
 * Determine the length of specified character string.
 *
 * Parameters:
 *   %rdi - pointer to a NUL-terminated character string
 *
 * Returns:
 *    number of characters in the string
 */
	.globl str_len
str_len:
	/* prologue to create ABI-compliant stack frame */
	pushq %rbp
	movq %rsp, %rbp

.Lstr_len_loop:
	cmpb $0, (%rdi)               /* found NUL terminator? */
	jz .Lstr_len_done             /* if so, done */
	inc %r10                      /* increment count */
	inc %rdi                      /* advance to next character */
	jmp .Lstr_len_loop            /* continue loop */

.Lstr_len_done:
	movq %r10, %rax               /* return count */

	/* epilogue */
	popq %rbp

	ret
```

As illustrated in the example function, labels for control flow
should be *local labels*, with names beginning with "`.L`".
If you don't use local labels within functions, debugging with
`gdb` will be difficult because `gdb` will think that each control
flow label is the beginning of a function.

### Debugging tips

You primary means of determining whether or not your code works correctly
is running the unit test programs (`c_imgproc_tests` and `asm_imgproc_tests`.)

If a unit test fails, you should use `gdb` to debug the code to determine
why it is not working.

Setting a breakpoint on the specific test function that is failing is
one way to start. For example, if the `test_get_neighbor` test function
is failing, in `gdb` set a breakpoint on the `get_neighbor` function, then run the
program so that it only runs that test function:

```
break test_get_neighbor
run test_get_neighbor
```

You will gain control of the program at the beginning of the test
function, at which point you can step through the code, inspect
variables, registers, and memory, etc.

Another good option for setting a breakpoint is the `tctest_fail`
function, because this is the function called when a test assertion
fails. For example, assuming `test_get_neighbor` has an assertion failure:

```
break tctest_fail
run test_get_neighbor
```

When the `tctest_fail` breakpoint is reached, use the `up` command (as many
times as needed) to enter the stack frame for the failing assertion.
This can allow you to check variables at the location of the assertion.

Don't forget that you can inspect register values in `gdb` by prefixing the
register name with the "`$`" character. For example:

```
print $ebx
```

would show you the contents of the `%ebx` register. Using `print/x` allows
you to see integer values in hexadecimal (very useful for checking color values.)

Casting a register to a pointer allows you to interpret memory as values
belonging to C data types. For example, let's say `%r10` points to a
`struct Image` instance. You could check the value of the element
at index 18 of the `data` array using the command

```
print/x ((struct Image *)$r10)->data[18]
```

If you are storing local variables in stack memory, and using `%rsp` to
access them, it is easy to see their values. In particular, if all of the
local variables are the same size and type (e.g., they are all
4-byte integers), then you can think of them as an array.
For example, in [the comment above about local variable allocation](#register-memory-comment),
there are 7 local variables allocated in stack memory, six of which
are 4 byte integer values, and one of which is an 8 byte integer value.
We can see the six four-byte values at once with the `gdb` comamnd

```
print (unsigned [6]) *((unsigned *)($rbp - 24))
```

Here we are pretending that these variables belong to the `unsigned` type,
which is the same as the `uint32_t` type.  The `(unsigned [6])` at the
beginning of the expression tells `gdb` that we are interpreting the
memory as an array of 6 `unsigned` elements. We use the expression
`$rbp - 24` to compute the address of the beginning of the
area containing the 4-byte variables,
because it is 24 bytes in size, and `%rbp` points to the "top"
of the area. Note that the `print` command will show the values of the local
variables starting with the local variable with the "lowest" address, i.e.,
the one referred to as `-24(%rbp)`.

We can see the 8 byte value in memory (at offset -32 from `%rbp`)
using the `gdb` command

```
print (unsigned long) *((unsigned long)($rbp - 32))
```

## Submitting

Before you upload your submission, make sure that your `README.txt`
contains the name of each team member and a brief summary of each team member's
contributions to the submission. If there is anything you would like us to
know about your submission, you can also add it to `README.txt`.

To prepare a zipfile for submission, run the command

```
make solution.zip
```

Please do not submit object files, executables, PNG image files,
or other files not mentioned above. (If you use `make solution.zip`
as recommended above, only the necessary files will be included in
the zipfile.)

Upload the zipfile to [Gradescope](https://www.gradescope.com) as
**Assignment 2 MS1**, **Assignment 2 MS2**, or **Assignment 2 MS3**, depending
on which milestone you are submitting.
