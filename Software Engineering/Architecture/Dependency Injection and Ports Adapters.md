---
tags: [architecture, dependency-injection, hexagonal-architecture, ports-and-adapters]
---

# Dependency Injection & Ports and Adapters

Two closely related ideas that, together, are what actually makes the [[Clean Architecture]] Dependency Rule practical to implement in real code.

## Dependency Injection (DI)

Instead of a class creating its own dependencies (`new PrismaClient()` inside a use case), the dependency is **provided from the outside**, usually by a container that wires everything together at startup. The class only declares what it needs, typically via its constructor.

```ts
// Without DI: the use case is coupled to a concrete implementation
class CreateOrderUseCase {
  private repository = new OrdersPrismaRepository(); // hardcoded dependency
}

// With DI: the dependency is injected, the use case doesn't know or care what it actually is
class CreateOrderUseCase {
  constructor(private readonly repository: IOrdersRepository) {}
}
```

This alone doesn't fix coupling, that's what the next section is for. DI is just the *mechanism*; **depending on an abstraction** is the actual principle (see [[SOLID]], Dependency Inversion).

## Ports and Adapters (Hexagonal Architecture)

Coined by Alistair Cockburn. The application's core (domain + use cases) defines **ports**, interfaces describing what it needs from the outside world, in its own vocabulary, with no knowledge of any specific technology. **Adapters** are the concrete implementations of those ports, living in infrastructure.

```
                    ┌─────────────────────┐
   HTTP request ──▶ │                     │ ──▶ IOrdersRepository (port)
                    │   Application Core   │
   CLI command  ──▶ │  (domain + use cases)│ ──▶ IEmailSender (port)
                    │                     │
                    └─────────────────────┘
                              ▲
              adapters implement the ports:
     OrdersPrismaRepository, SendgridEmailSender, InMemoryOrdersRepository (for tests)
```

The core never imports an adapter. Adapters import the port interface and implement it. Wiring, deciding *which* adapter satisfies *which* port, happens at the composition boundary (in [[NestJS]], that's a `@Module`'s `providers` array).

```ts
// Port, defined by the application core
export interface IOrdersRepository {
  save(order: Order): Promise<void>;
}

// Adapter, defined by infrastructure, implements the port
@Injectable()
export class OrdersPrismaRepository implements IOrdersRepository {
  async save(order: Order): Promise<void> { /* Prisma-specific code */ }
}

// Wiring, the only place that knows both sides
@Module({
  providers: [
    { provide: ORDERS_REPOSITORY, useClass: OrdersPrismaRepository },
  ],
})
export class OrdersModule {}
```

## Why this pays off

- **Testability**, swap `OrdersPrismaRepository` for an in-memory fake in unit tests, with zero changes to the use case.
- **Replaceability**, switching from Prisma to a different ORM, or from SendGrid to a different email provider, only touches the adapter, never the business logic that depends on the port.
- **Enforceable boundary**, because the dependency direction is explicit (core → port ← adapter), tools like `dependency-cruiser` can verify at build time that no adapter-specific type ever leaks into the core.

## See also

- [[Clean Architecture]]
- [[SOLID]]
- [[Domain-Driven Design (DDD)]]
- [[NestJS]]

Write by **Samuel**