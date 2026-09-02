---
tags: [systems-programming, c, memory, low-level]
---

# Systems Programming

Most day-to-day backend work (spinning up a Node server, calling an ORM, deploying to Render) happens several layers above what's actually going on in memory and in the CPU. This folder is about that lower layer: what a variable actually is in memory, what a pointer really points to, what "the stack" and "the heap" mean physically, and what happens between writing `.c` code and having a running program.

**All examples in this folder are in C.** C is used on purpose here — it doesn't hide the memory model behind a garbage collector or a managed runtime the way Node/JavaScript does, so it's the most direct way to see what's actually happening. Every code block is explicitly labeled `c`.

## Why bother, coming from Node/TypeScript

Node hides all of this: V8 manages memory for you, there's no manual `malloc`/`free`, no explicit pointers. That's convenient, but it also means things like "why is this object shared instead of copied", "why did my process just run out of memory", or "why is passing a huge array around expensive" are easier to reason about once you've seen the layer underneath. That's the motivation for this folder, not "become a C developer", but "understand what the runtime is doing for you".

## Notes in this folder

- [[Pointers and Memory]] — what a pointer actually is, addresses, dereferencing, pointer arithmetic, `malloc`/`free`, common bugs (dangling pointers, leaks, double free).
- [[The Stack and the Heap]] — where local variables live vs. where dynamically allocated memory lives, stack frames, function calls, why the stack is fast and bounded and the heap is flexible and slower.
- [[Compilation Process]] — what actually happens between writing `.c` source and running a binary: preprocessing, compilation, assembly, linking.

More notes will be added here as we go, memory layout, static vs. dynamic linking, processes and how a running program relates to the OS.

## See also

- [[Node.js]] — useful contrast: Node's event loop and garbage collector are doing, automatically, a version of what we manage by hand in these notes.

Write by **Samuel**
