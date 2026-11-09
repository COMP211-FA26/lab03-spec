# Lab 03: Generic Vectors, Strings, and Test-Driven Development

## Abstract

In this lab, you will implement two C abstractions, `Vec` and `Str`, while practicing **test-driven development (TDD)**. You will write unit tests to guide your implementation, improve the reliability of your code, and protect against regressions when making changes.

Most functions in this lab have relatively short implementations, but they require careful attention to memory management, pointer arithmetic, and edge cases.

The `Vec` abstraction provides a generic, dynamically sized array capable of storing elements of any type. The `Str` abstraction is built on top of `Vec` and provides a more convenient API for working with dynamically sized strings.

Throughout the lab, you will practice writing tests, implementing functions according to an API specification, and refactoring code while preserving its behavior.

**Getting Started**

Accept the assignment: [Lab 03 Classroom50](https://classroom50.org/COMP211-FA26/comp211-fa26/assignments/lab03/accept)

After accepting the assignment:

1. Open your GitHub Classroom repository.
2. Click the green **Code** button and copy the HTTPS URL.
3. Open a terminal in your `learncli` container.
4. Clone your repository using `git clone <URL>`, replacing `<URL>` with the copied URL.
5. Change your working directory to the cloned repository.

---

## Part 0. Project Familiarization

In this lab, you will focus primarily on implementing and testing the `Vec` and `Str` libraries. Initially, there is no meaningful functionality in `main.c`. Your work will focus on implementing an **application programming interface (API)** according to its specification.

Start by examining these three files:

1. `include/Vec.h`
2. `src/Vec.c`
3. `test/unit/VecImpl.cpp`

First, read through `Vec.h`. Pay attention to the function declarations and documentation. These describe the behavior your implementations must provide.

Next, examine the starter code in `src/Vec.c`. Implementations of `Vec_value()`, `Vec_drop()`, `Vec_length()`, and `Vec_ref()` are provided.

Notice that `Vec_ref()` checks whether the requested index is within bounds. A design decision of this library is to terminate the program when an out-of-bounds operation is attempted.

**Submission:** You are not implementing any functions in part 0. Answer the corresponding [Gradescope questions](https://www.gradescope.com/courses/1349648/assignments/9029249) to check your understanding of the `Vec` API.

---

## Part 1. Unit Tests

Open `test/unit/VecImpl.cpp`. This file contains example unit tests for `Vec_value()`, `Vec_drop()`, and `Vec_ref()`.

Answer the corresponding [Gradescope questions](https://www.gradescope.com/courses/1201157/assignments/7880548/) to check your understanding of these tests.

### Writing Your Own Tests

You will write unit tests for the following functions:

1. `Vec_get()`
2. `Vec_set()`
3. `Vec_equals()`
4. `Vec_splice()`

Add your tests to `VecImpl.cpp`.

**Start with simple tests before moving on to more complicated cases.** As you develop each function, add tests for additional behaviors and edge cases.

Because these are *implementation tests*, you are allowed to access the members of the `Vec` struct directly to inspect its internal state.

### Testing `Vec_splice()`

The `Vec_splice()` function is particularly well suited to test-driven development because it supports several different operations.

Write tests for a variety of cases, including inserting, removing, and replacing elements at different positions in the vector.

Consider what should happen when:

- The number of inserted elements is greater than, less than, or equal to the number of removed elements.
- Elements are inserted or removed at the beginning, middle, or end.
- The operation requires the vector to increase its capacity.

You should write tests for **at least eight substantively different use cases** of `Vec_splice()`.

Give each test a meaningful name that describes the behavior it checks.

### Running Your Tests

The repository includes a Makefile with commands for building and testing your code.

```bash
make             # Build the project
make unit-test   # Run all unit tests
make test        # Run unit and integration tests
make clean       # Remove generated build files
```

Run your tests frequently as you implement your functions.

Remember that tests for functions you have not implemented yet are expected to fail. As you complete each function, its corresponding tests should begin to pass.

---

## Part 2. `Vec` Implementation

Your job in `src/Vec.c` is to implement the following functions declared in `Vec.h`:

1. `Vec_get()`
2. `Vec_set()`
3. `Vec_equals()`
4. `Vec_splice()`

The expected behavior of each function is described in `Vec.h`. The starter code in `Vec.c` also provides implementation hints.

You will need to work with raw memory because `Vec` stores its elements in a `void*` buffer.

### Working with Memory

Three useful C standard library functions for this lab are `memcpy()`, `memmove()`, and `memcmp()`.

All three are declared in:

```c
#include <string.h>
```

### `memcpy()`

`memcpy()` copies raw bytes from one memory location to another.

Unlike assignment of individual variables or `strcpy()` for strings, `memcpy()` can copy data of any type, including integers, arrays, and structs.

**Function declaration:**

```c
void *memcpy(void *dest, const void *src, size_t n);
```

Parameters:

- `dest`: Destination memory address.
- `src`: Source memory address.
- `n`: Number of bytes to copy.

The function returns a pointer to `dest`.

**Example 1: Copying an integer**

```c
int a = 42;
int b = 0;

memcpy(&b, &a, sizeof(int));

printf("%d\n", b);  // Prints 42
```

This copies the bytes representing `a` into `b`.

**Example 2: Copying an array**

```c
int src[5] = {1, 2, 3, 4, 5};
int dest[5];

memcpy(dest, src, 5 * sizeof(int));
```

Afterward, `dest` contains the same values as `src`.

**Important:** `memcpy()` must not be used when the source and destination memory regions overlap. Use `memmove()` instead.

### `memmove()`

`memmove()` also copies raw bytes between memory locations, but unlike `memcpy()`, it safely handles overlapping memory regions.

**Function declaration:**

```c
void *memmove(void *dest, const void *src, size_t n);
```

The parameters have the same meanings as those of `memcpy()`.

**Example: Shifting array elements**

```c
int arr[] = {10, 20, 30, 40, 50};

memmove(&arr[2], &arr[1], 3 * sizeof(int));
```

Afterward, `arr` contains:

```text
{10, 20, 20, 30, 40}
```

The values originally at indices 1, 2, and 3 have been shifted one position to the right.

Notice that the source and destination regions overlap. Using `memcpy()` here would result in undefined behavior.

**Why is this useful for `Vec_splice()`?**

When inserting or removing elements, you may need to shift existing elements within the same buffer. Since these regions can overlap, `memmove()` is appropriate.

You can also use `memmove()` to copy between non-overlapping memory regions.

### `memcmp()`

`memcmp()` compares two memory regions byte by byte.

**Function declaration:**

```c
int memcmp(const void *ptr1, const void *ptr2, size_t n);
```

Parameters:

- `ptr1`: Pointer to the first memory region.
- `ptr2`: Pointer to the second memory region.
- `n`: Number of bytes to compare.

Return values:

- `0` if all compared bytes are identical.
- A nonzero value if the bytes differ.

**Example: Comparing arrays**

```c
int a[] = {10, 20, 30};
int b[] = {10, 20, 30};

if (memcmp(a, b, 3 * sizeof(int)) == 0) {
    printf("The arrays contain identical bytes.\n");
}
```

**Why is this useful for `Vec_equals()`?**

Because `Vec` can store elements of any type, `memcmp()` provides a way to compare the raw bytes stored in two vectors.

Keep in mind that byte-for-byte equality is not always the same as logical value equality. For example, structs containing pointers may have different pointer values even when the objects they reference contain identical data.

### Implementing and Testing

As you implement each function:

1. Write tests for the expected behavior.
2. Implement the function.
3. Run your tests.
4. Fix any problems your tests reveal.
5. Add more tests for edge cases.

Pay particular attention to bounds checking, capacity growth, and correctly updating the vector's length.

---

## Part 3. User Tests

Once you believe your `Vec` implementations are correct, open `test/unit/VecSpec.cpp`.

In this file, you will write tests from the **user's perspective**.

Unlike implementation tests, user tests must use only the public API functions declared in `Vec.h`.

For example, a user test may call `Vec_length()` to determine the number of elements, but it should not access `v.length` directly.

This distinction illustrates an important principle called the **abstraction barrier**.

Users of an abstraction should not need to know how it is implemented internally. They should be able to interact with it entirely through its public API.

Your user tests should verify the behavior promised by the API without depending on internal implementation details.

---

## Part 4. `Vec` Improvements

Your goal in this part is to improve your `Vec` implementation without changing its behavior.

This process is called **refactoring**.

### Refactor `Vec_set()`

First, rewrite `Vec_set()` so that it uses `Vec_splice()` instead of directly implementing the replacement and append operations.

Your new implementation must preserve all the original behavior of `Vec_set()`, including:

- Replacing an existing element.
- Appending a new element when the index equals the vector's length.
- Handling invalid indices according to the API specification.

After refactoring, run your existing tests.

All tests should continue to pass **without modifying the tests**. If any fail, investigate and fix the regression before proceeding.

Once your refactored implementation passes all tests, make a Git commit.

### Additional Improvements

Next, identify other opportunities to improve your code.

For example, you might:

- Improve variable names.
- Remove duplicated code.
- Extract repeated logic into static helper functions.
- Simplify complicated sections of your implementation.

After each change, rerun your tests to confirm that the behavior has not changed.

Make a Git commit after each successful refactoring.

If a change introduces a problem, you can use Git to restore a previous working version. Use `git status` to inspect your changes before deciding what to revert.

---

## Part 5. `Str` Implementation

The `Str` abstraction provides a dynamically allocated, growable string.

It is built on top of `Vec`, but unlike `Vec`, it specifically stores characters (`char`). This allows `Str` to provide a more convenient API for string operations.

Start by reading:

1. `include/Str.h`
2. `src/Str.c`
3. `test/unit/StrImpl.cpp`
4. `test/unit/StrSpec.cpp`

### Understanding the `Str` Invariant

An important **invariant** of `Str` is that its underlying vector always contains a null terminator (`'\0'`) as its final element.

An invariant is a property that must remain true throughout the lifetime of a data structure.

For example, a `Str` containing `"hello"` stores six characters:

```text
'h' 'e' 'l' 'l' 'o' '\0'
```

Although the underlying `Vec` contains six elements, `Str_length()` returns `5` because the null terminator is not included in the string's length.

Similarly, `Str_value(capacity)` takes the number of characters the string can initially store, excluding the null terminator. The underlying vector reserves one additional element for the terminator.

As you implement the `Str` functions, you must preserve this invariant when characters are inserted, removed, replaced, or appended.

### Respecting the Abstraction Barrier

Your `Str` implementation should use the public functions provided by `Vec` rather than accessing the vector's struct members directly.

For example, use `Vec_length()` rather than reading the `length` member directly.

There is one intentional exception in the starter code: `Str_value()` directly modifies the vector's members because `Vec_set()` has not yet been implemented.

**Your first task is to replace this temporary code with the provided call to `Vec_set()`.**

After making this change, `Str_value()` should work without directly accessing the underlying vector's members.

### Implementing the `Str` Functions

Implement the unfinished functions in `src/Str.c` according to the specifications in `Str.h`.

The starter code provides hints about which `Vec` functions may be useful.

Remember that the purpose of `Str` is to build a new abstraction on top of an existing one. You should reuse your `Vec` implementations rather than duplicating their functionality.

### Testing `Str`

Follow the same test-driven development process you used for `Vec`.

In `StrImpl.cpp`, write implementation tests that focus on behavior specific to strings, especially null termination and character operations.

You do not need to retest every detail of `Vec` that your earlier tests have already established.

In `StrSpec.cpp`, write user tests that verify the behavior of `Str` using only the public API declared in `Str.h`.

As before, user tests should not access the underlying struct members directly.

---

## Part 6. `main.c` Line Echo

Write a program in `src/main.c` that reads lines from standard input and prints each line back to standard output using your `Str` implementation.

Your program should:

1. Use a fixed-size `char` buffer and `fgets()` to read input.
2. Use `Str_append()` to accumulate characters into a `Str`.
3. Continue reading when a line is longer than the fixed-size buffer, appending additional chunks until the complete line has been read.
4. Print the completed line using `Str_cstr()`, preserving the newline character if one was present.
5. Repeat until end-of-file (EOF) is reached.
6. If the final line does not end with a newline, print it without adding one.
7. Free all dynamically allocated memory using `Str_drop()`.

Your program should handle lines of arbitrary length, including lines longer than the fixed-size input buffer.

### Testing Your Program

Test your program with several different inputs, including:

- A single short line.
- Multiple lines.
- An empty line.
- A line longer than your input buffer.
- A final line without a newline character.
- Empty input.

Use Valgrind to check for memory leaks and invalid memory accesses.

You can run your compiled program under Valgrind using:

```bash
make
valgrind --leak-check=full ./build/bin/thstr
```

Your program should not leak memory or perform invalid memory accesses.

---

## Grading

Your lab grade will be based on the following:

| Component | Weight |
|---|---:|
| Autograded correctness | 70% |
| Unit test coverage and quality | 30% |

### Autograded Correctness (70%)

Your implementations must behave according to the API specifications in `Vec.h` and `Str.h`.

### Unit Tests (30%)

Your tests will be evaluated based on:

- **Organization:** Are tests clearly named and organized?
- **Coverage:** Do tests exercise important behaviors and edge cases?
- **Implementation tests:** Do they verify relevant internal behavior?
- **User tests:** Do they verify the public API without directly accessing struct members?
- **Meaningful assertions:** Do the tests actually check that the functions behave correctly?

For each function you implement in `Vec.c` and `Str.c`, write at least **two implementation tests and two user tests**.

These tests should cover meaningfully different scenarios rather than repeating the same behavior with different input values.

For `Vec_splice()`, you must also satisfy the requirement of at least eight substantively different test cases described in Part 1.

Remember that the goal is not simply to write a large number of tests. Your tests should give you confidence that your implementation is correct and help detect regressions as your code changes.