---
tags: [systems-programming, c, memory, stack, heap]
---

# The Stack and the Heap

A running program's memory is organized into a few regions. The two that matter day-to-day are the **stack** and the **heap**, they hold different kinds of data, have different lifetimes, and behave very differently in terms of speed and flexibility.

## The stack

Every time a function is called, the program pushes a **stack frame** onto the stack: space for that function's local variables, its parameters, and the address to return to once it finishes. When the function returns, that frame is popped off, its memory is instantly reclaimed, no cleanup needed.

```c
#include <stdio.h>

int square(int x) {
    int result = x * x; // 'result' and 'x' live in square's stack frame
    return result;
}

int main(void) {
    int value = 5;       // main's stack frame
    int output = square(value); // pushes a new frame for square()
    printf("%d\n", output);     // square's frame is already gone by now
    return 0;
}
```

Properties of the stack:

- **Fast** — allocating a stack frame is just moving a pointer (the "stack pointer"), no bookkeeping needed.
- **Automatic lifetime** — variables disappear the moment their function returns. You never call anything like `free()` for a plain local variable.
- **Bounded and fixed-size** — the stack has a limited size set by the OS/runtime. Recurse too deeply (or allocate a huge local array) and you get a **stack overflow**:

```c
// DON'T actually run this — illustrative only
int infinite_recursion(int n) {
    return infinite_recursion(n + 1); // never returns, keeps pushing frames
}
```

Each call adds another frame with nowhere to unwind to, eventually the stack runs out of space and the program crashes.

## The heap

The heap is a much larger, more flexible region for memory you request explicitly at runtime, with `malloc` (see [[Pointers and Memory]]) and give back explicitly with `free`.

```c
#include <stdlib.h>

int *make_array(int size) {
    int *arr = malloc(size * sizeof(int)); // heap memory
    // ... fill it in ...
    return arr; // still valid after make_array() returns!
}
```

Notice the difference from the stack example above: `arr` points to heap memory, so it's still valid after `make_array()` returns, unlike a plain local variable, which would be garbage the moment its function ends. That's the entire reason the heap exists: to hold data whose lifetime needs to outlive the function that created it, or whose size isn't known until runtime.

Properties of the heap:

- **Slower** — the allocator has to find a free block of the right size and track it; there's real bookkeeping involved.
- **Manual lifetime** — nothing frees heap memory automatically in C. You track it and call `free()` yourself (this is exactly where leaks and dangling pointers, discussed in [[Pointers and Memory]], come from).
- **Flexible size** — can be allocated on demand, at whatever size is needed, and resized with `realloc`.

## Stack vs. heap, side by side

| | Stack | Heap |
|---|---|---|
| Allocation | Automatic (function call) | Manual (`malloc`) |
| Deallocation | Automatic (function return) | Manual (`free`) |
| Speed | Very fast | Slower |
| Size | Small, fixed limit | Large, limited by system memory |
| Lifetime | Tied to the function call | Until explicitly freed |
| Typical failure mode | Stack overflow (too much recursion/local data) | Leaks, dangling pointers, fragmentation |

## The Node/V8 connection

This is where the "why does Node hide all this" question from [[Systems Programming]] gets its answer: V8 (Node's JS engine) manages a heap for you automatically, objects get allocated there, and V8's garbage collector figures out when nothing references them anymore and reclaims that memory, the equivalent of calling `free()`, but automatic and non-deterministic in timing. Primitive local values in JS get similar stack-like treatment under the hood. The concepts are the same, the difference is just who's responsible for managing it.

## See also

- [[Pointers and Memory]]
- [[Systems Programming]]

Write by **Samuel**