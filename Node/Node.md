---
tags: [nodejs, backend, javascript, typescript]
---

# Node.js

Node.js is a **server-side JavaScript runtime**, built on top of the V8 engine (the same one Chrome uses). It's not a new language or a framework — it's a runtime environment that lets JavaScript run outside the browser, with access to the filesystem, networking, processes, etc.

## Why Node exists

Before Node, JavaScript only ran in the browser. Node brought the same execution model, **single-threaded, event-driven, non-blocking I/O**, to the backend. That makes it particularly good for workloads with heavy concurrent I/O (HTTP APIs, WebSockets, streaming), and less ideal, without extra help, for CPU-bound work (image processing, heavy cryptography), which blocks the single main thread.

## Concurrency model: the Event Loop

Node doesn't use a thread per request (unlike many traditional runtimes). It runs on a single main thread and delegates I/O operations (disk reads, network calls, database queries) to the OS or an internal thread pool (via `libuv`), resuming execution through a **callback/Promise** once the operation finishes.

This means: synchronous, heavy code (a huge loop, an expensive computation) blocks the entire event loop, no other request gets served while it runs. That's why blocking operations (`fs.readFileSync`, loops without `await`) are avoided in production code.

## TypeScript as the de facto standard

Virtually every medium/large-scale modern Node project uses **TypeScript** on top of JavaScript, static typing, better IDE support, and, critically, the decorators and type metadata that frameworks like [[NestJS]] rely on for Dependency Injection depend on TypeScript (`experimentalDecorators`, `emitDecoratorMetadata` in `tsconfig.json`).

## Ecosystem

- **npm** (or `pnpm`/`yarn`), package manager, the largest open-source package registry in the world.
- **HTTP frameworks:** Express (the most traditional, minimalist), Fastify (performance-focused), [[NestJS]] (opinionated, built-in architecture).
- **Common ORMs:** Prisma, TypeORM, Knex.

## See also

- [[NestJS]]
- [[NestJS and Fastify]]
- [[Clean Architecture]]

Write by **Samuel**