---
tags: [systems-programming, c, compilation, toolchain]
---

# Compilation Process

Coming from Node, "running code" means `node server.js`, the JS source is interpreted/JIT-compiled on the fly by V8. C works completely differently: source code goes through several distinct stages *before* you get an executable, and understanding those stages explains a lot of otherwise-confusing compiler behavior (missing symbols, header errors, "works on my machine" binary issues).

## The four stages

Given a single file `main.c`:

```c
#include <stdio.h>

#define GREETING "Hello"

int main(void) {
    printf("%s, world!\n", GREETING);
    return 0;
}
```

Running `gcc main.c -o main` actually does four separate things under the hood:

### 1. Preprocessing

Handles everything starting with `#`: `#include` gets textually replaced by the referenced file's contents, `#define` macros get substituted, `#ifdef` blocks get resolved. Output is still C source, just expanded.

```bash
gcc -E main.c -o main.i
```

`main.i` would show `printf("%s, world!\n", "Hello");` with the full expanded contents of `stdio.h` pasted in above it, often thousands of lines from one `#include`.

### 2. Compilation

Translates the preprocessed C source into **assembly language** for the target architecture (e.g. x86-64). This is where syntax errors, type errors, and most of what people just call "compiler errors" actually get caught.

```bash
gcc -S main.i -o main.s
```

`main.s` is human-readable-ish assembly, CPU instructions in text form, not yet machine code.

### 3. Assembly

The assembler turns that assembly text into actual **machine code**, an **object file** (`.o`), binary instructions the CPU can execute. But it's not a runnable program yet, it's missing pieces (like the real implementation of `printf`, which lives in the C standard library).

```bash
gcc -c main.s -o main.o
```

### 4. Linking

The **linker** takes one or more object files and combines them with the libraries they depend on (like `libc`, which provides `printf`), resolving all the references between them, into a single executable.

```bash
gcc main.o -o main
```

Run all four as one command and this is exactly what `gcc main.c -o main` does behind the scenes.

## Why linking matters: static vs. dynamic

When linking, `printf`'s actual code can be pulled in two ways:

- **Static linking** — a copy of the library code gets embedded directly into your executable. The binary is bigger and self-contained, no external dependency needed to run it.
- **Dynamic linking** — the executable just records "I need `libc`", and the actual library code gets loaded from a shared `.so` file (Linux) at the moment the program runs. Smaller binary, but the shared library has to be present on whatever machine runs it.

This is the direct ancestor of problems that show up in higher-level ecosystems too, missing shared libraries at runtime, version mismatches between what a binary expects and what's installed, the same category of problem as a Node app failing because `node_modules` wasn't installed, just one layer lower and far less forgiving about it.

## Multiple files: why linking is a separate step

The whole reason compilation and linking are split into separate stages is so a project can be split into multiple `.c` files, each compiled independently into its own `.o`, and only combined at the end:

```bash
gcc -c file1.c -o file1.o
gcc -c file2.c -o file2.o
gcc file1.o file2.o -o program
```

That's what makes incremental builds possible: change `file2.c`, only recompile `file2.o`, then re-link, instead of recompiling the entire project every time. This is the direct predecessor of what a `Makefile` (or, at a much higher level, a JS bundler's incremental build) is automating.

## See also

- [[Pointers and Memory]]
- [[The Stack and the Heap]]
- [[Systems Programming]]

Write by **Samuel**