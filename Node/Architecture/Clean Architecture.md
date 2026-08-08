---
tags: [architecture, clean-architecture, design-principles]
---

# Clean Architecture

An architectural style popularized by Robert C. Martin, organized around **layers** and one non-negotiable rule: **the Dependency Rule**, source code dependencies can only point *inward*. Nothing in an inner layer can know anything about an outer layer.

## The layers (from innermost to outermost)

```
┌─────────────────────────────────────────┐
│           Presentation                   │  ← HTTP controllers, DTOs, CLI
│  ┌─────────────────────────────────┐    │
│  │        Infrastructure            │    │  ← DB repositories, external APIs, message queues
│  │  ┌─────────────────────────┐    │    │
│  │  │      Application          │    │    │  ← Use cases, application ports
│  │  │  ┌───────────────────┐  │    │    │
│  │  │  │      Domain         │  │    │    │  ← Entities, Value Objects, business rules
│  │  │  └───────────────────┘  │    │    │
│  │  └─────────────────────────┘    │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
```

- **Domain**, entities, value objects, domain events, business rules. Pure code: no framework, no ORM, no HTTP knowledge. See [[Domain-Driven Design (DDD)]].
- **Application**, use cases that orchestrate the domain. Depends on domain-defined *ports* (interfaces), never on concrete implementations.
- **Infrastructure**, concrete adapters: database repositories, outbound HTTP clients, message queue producers. Implements the ports the application layer defined.
- **Presentation**, HTTP controllers, DTOs, request validation. Talks to use cases, never directly to the domain.

## Why the domain has to stay pure

If a `BloodComponent`, style entity imported `@nestjs/common` or a Prisma type directly, changing the ORM or the web framework would force changes to business rules that have nothing to do with either. Keeping the domain framework-agnostic means:

- Business rules are testable with plain unit tests, no framework bootstrap, no database, no HTTP server.
- Swapping infrastructure (a different ORM, a different message broker) never touches business logic.

## Enforcing it, not just documenting it

A common failure mode: the rule is written in a README, and under sprint pressure someone imports something they shouldn't "just this once". The fix is to make violations fail the build, with a tool like `dependency-cruiser`:

```js
// .dependency-cruiser.js (simplified example)
module.exports = {
  forbidden: [
    {
      name: 'domain-must-be-pure',
      severity: 'error',
      from: { path: '^src/modules/.+/domain' },
      to: { path: '^node_modules/(@nestjs|prisma|@prisma)' },
    },
  ],
};
```

## Relation to other concepts

Clean Architecture is the *structural* container; [[Domain-Driven Design (DDD)]] is what usually fills the domain layer with meaning (Entities, Aggregates, Value Objects); [[Dependency Injection and Ports Adapters|Ports & Adapters]] (Hexagonal Architecture) is the pattern that makes the Dependency Rule practical to implement.

## See also

- [[Domain-Driven Design (DDD)]]
- [[Dependency Injection and Ports Adapters]]
- [[SOLID]]
- [[Bounded Context]]

Write by **Samuel**