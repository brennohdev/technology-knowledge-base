---
tags: [nodejs, nestjs, fastify, performance]
---

# NestJS with Fastify

By default, [[NestJS]] runs on top of **Express**. It also supports **Fastify** as an alternative HTTP platform, by swapping only the *adapter*, the Controller/Provider/Module structure doesn't change at all.

## Why switch to Fastify

Fastify is significantly faster and lighter on memory than Express, with a JSON Schema-based validation/serialization scheme built natively for performance. It makes a real difference in high-concurrency scenarios, real-time dashboards, APIs with many simultaneous clients, high-volume endpoints.

## How to configure it

```ts
import { NestFactory } from '@nestjs/core';
import { FastifyAdapter, NestFastifyApplication } from '@nestjs/platform-fastify';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create<NestFastifyApplication>(
    AppModule,
    new FastifyAdapter(),
  );
  await app.listen(3000, '0.0.0.0');
}

bootstrap();
```

Required package: `@nestjs/platform-fastify` (replaces `@nestjs/platform-express`).

## Differences to keep in mind

- **Express middlewares aren't 100% compatible**, many packages from the Express ecosystem (e.g. some older middlewares) don't work directly on Fastify. Fastify-native equivalents exist for most common cases (`@fastify/helmet`, `@fastify/cors`, `@fastify/multipart`).
- **Registering Fastify plugins** is done via `app.register(...)`, not `app.use(...)`:

```ts
import fastifyHelmet from '@fastify/helmet';

await app.register(fastifyHelmet);
```

- **Request/Response object typing** changes (they're Fastify's types, not Express's), relevant if any code accesses `req`/`res` directly instead of using NestJS decorators (`@Body()`, `@Param()`, etc., which abstract this away).

## When it's worth it

For most internal CRUD applications, the performance difference between Express and Fastify isn't the real bottleneck. It's worth switching when: the API serves high-volume external clients (e.g. partners consuming via REST), low latency matters for real-time dashboards, or the team is already comfortable with the Fastify plugin ecosystem.

## See also

- [[NestJS]]
- [[Node]]

Write by **Samuel**