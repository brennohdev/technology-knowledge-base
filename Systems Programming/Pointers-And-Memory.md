---
tags: [systems-programming, c, memory, pointers]
---

# Pointers and Memory

Every variable in a running program lives somewhere in memory, at some numeric address. A **pointer** is just a variable whose value *is* an address, instead of, say, an integer or a character.

## Addresses and dereferencing

```c
#include <stdio.h>

int main(void) {
    int age = 25;
    int *ptr = &age;   // ptr now holds the ADDRESS of age

    printf("value of age:      %d\n", age);
    printf("address of age:    %p\n", (void *)&age);
    printf("value of ptr:      %p\n", (void *)ptr);
    printf("value pointed to:  %d\n", *ptr); // dereference: "go to that address and read"

    return 0;
}
```

- `&age` — the **address-of** operator, "give me where `age` lives".
- `int *ptr` — declares `ptr` as a pointer to an `int`.
- `*ptr` — the **dereference** operator, "go to the address `ptr` holds and give me the value there". Also works for *writing*: `*ptr = 30;` changes `age` itself, because `ptr` points at `age`'s address.

This is the core idea: a pointer doesn't hold a copy of the value, it holds a *reference* to where the value lives. Two pointers to the same address are, effectively, two ways of reaching the same piece of memory.

## Pointer arithmetic and arrays

In C, an array and a pointer are closely related. The array name decays to a pointer to its first element:

```c
#include <stdio.h>

int main(void) {
    int numbers[4] = {10, 20, 30, 40};
    int *p = numbers; // same as &numbers[0]

    for (int i = 0; i < 4; i++) {
        printf("numbers[%d] = %d  (via pointer: %d)\n", i, numbers[i], *(p + i));
    }

    return 0;
}
```

`*(p + i)` and `numbers[i]` do exactly the same thing. `p + i` doesn't move `i` bytes, it moves `i * sizeof(int)` bytes, the compiler does that scaling automatically based on the pointer's type.

## Dynamic memory: `malloc` and `free`

Local variables (like `age` and `numbers` above) live on the [[The Stack and the Heap|stack]] and disappear automatically when the function returns. Sometimes you need memory that outlives the function, or whose size you only know at runtime, that's what the **heap** is for, and you manage it explicitly:

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int n = 5;
    int *arr = malloc(n * sizeof(int)); // ask the heap for room for 5 ints

    if (arr == NULL) {
        // malloc can fail (out of memory) — always check
        return 1;
    }

    for (int i = 0; i < n; i++) {
        arr[i] = i * i;
    }

    for (int i = 0; i < n; i++) {
        printf("%d\n", arr[i]);
    }

    free(arr); // give the memory back — C does NOT do this for you
    arr = NULL; // avoid leaving a dangling pointer around

    return 0;
}
```

Unlike Node (V8's garbage collector reclaims memory automatically), in C **you** are responsible for calling `free()` on everything you `malloc()`. Nothing does it for you.

## Common bugs this causes

- **Memory leak** — you `malloc` and never `free`. The memory stays reserved for the life of the process even though nothing uses it anymore.
- **Dangling pointer** — you `free(ptr)` but keep using `ptr` afterwards. The memory might get reused for something else, so you're reading/writing garbage (or corrupting unrelated data).
- **Double free** — calling `free()` twice on the same pointer. Undefined behavior, often crashes.
- **Buffer overflow** — writing past the end of an allocated block (e.g. writing `arr[5]` on a 5-element array indexed 0-4). Silently corrupts adjacent memory instead of raising a clean error.

None of these exist as a *runtime* concern in Node, the garbage collector and array bounds checking handle all of this for you. Seeing them explicitly in C is what makes clear what "managed memory" is actually managing.

## See also

- [[The Stack and the Heap]]
- [[Systems Programming]]

Write by **Samuel**