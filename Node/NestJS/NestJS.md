---
tags: [nodejs, nestjs, backend, framework]
---

# NestJS

An opinionated Node.js framework, built in TypeScript, heavily inspired by Angular, it uses **decorators** and a native **Dependency Injection (DI)** container to organize the application. By default it runs on top of Express, but it also supports [[NestJS and Fastify|Fastify]] as an alternative adapter.

Unlike plain Express (minimalist, no opinion about structure), NestJS enforces a modular architecture, which helps teams stay consistent as the application grows, and fits naturally with [[Clean Architecture]] and [[Domain-Driven Design (DDD)]].

## Building blocks

### Module
The organizational unit, groups related Controllers and Providers. Each domain [[Bounded Context]] tends to become its own Module.

```ts
@Module({
  controllers: [UsersController],
  providers: [CreateUserUseCase, UsersRepository],
})
export class UsersModule {}
```

### Controller
The HTTP entry point, receives the request, validates/deserializes it, delegates to a provider (ideally a use case), returns the response. Should not contain business logic.

```ts
@Controller('users')
export class UsersController {
  constructor(private readonly createUser: CreateUserUseCase) {}

  @Post()
  async create(@Body() body: CreateUserDto) {
    return this.createUser.execute(body);
  }
}
```

### Provider
Any class that can be injected, services, use cases, repositories, factories. Marked with `@Injectable()`.

```ts
@Injectable()
export class CreateUserUseCase {
  constructor(private readonly usersRepository: UsersRepository) {}

  async execute(input: CreateUserInput): Promise<void> {
    // use case orchestration
  }
}
```

## Dependency Injection (DI)

The NestJS container resolves dependencies automatically from what's declared in the constructor, no manual instantiation needed. This is what makes [[Dependency Injection and Ports Adapters|Ports & Adapters]] clean to implement: code depends on an *interface* (port), and the module decides, in its configuration, which concrete implementation (adapter) to inject.

```ts
// Port (interface), lives in the domain/application layer
export interface IUsersRepository {
  findById(id: string): Promise<User | null>;
}

// Injection token (good practice: use a Symbol, not a string, to avoid collisions)
export const USERS_REPOSITORY = Symbol('IUsersRepository');

// Concrete adapter, lives in infrastructure
@Injectable()
export class UsersPrismaRepository implements IUsersRepository {
  async findById(id: string): Promise<User | null> { /* ... */ }
}

// Wiring in the module, only here does the interface know about the concrete implementation
@Module({
  providers: [
    { provide: USERS_REPOSITORY, useClass: UsersPrismaRepository },
  ],
})
export class UsersModule {}
```

## Why it fits well with DDD/Clean Architecture

- Each bounded context becomes an isolated Module.
- The DI container replaces the need for a manual "composition root".
- The domain layer can stay 100% free of NestJS decorators (no `@Injectable()` on entities), only the application/infrastructure layer needs to know about the framework. This can be **enforced** with tools like `dependency-cruiser`, not just documented as a convention.

## See also

- [[NestJS and Fastify]]
- [[Clean Architecture]]
- [[Bounded Context]]
- [[Dependency Injection and Ports Adapters]]

Write by **Samuel**
