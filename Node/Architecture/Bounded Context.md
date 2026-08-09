---
tags: [architecture, ddd, bounded-context]
---

# Bounded Context

A **Bounded Context** is an explicit boundary within which a specific domain model is valid and internally consistent. The same word can, and often should, mean something different in a different context; DDD treats that as normal, not as a naming conflict to resolve.

## A concrete example

In an e-commerce system, "Customer" isn't one universal concept:

- In the **Sales** context, a Customer has a cart, a payment method, an order history.
- In the **Support** context, a Customer has open tickets, a satisfaction score, a support tier.
- In the **Shipping** context, a Customer is mostly a delivery address and contact info.

Trying to build one single `Customer` entity that satisfies all three ends up either bloated with fields only one context cares about, or coupled in ways that make each team's changes risk breaking the others. Modeling three separate `Customer` concepts, one per bounded context, connected only by a shared ID, usually produces a healthier system.

## How this maps to code

A common, practical translation: **one module per bounded context** (in [[NestJS]] terms, one `@Module` per context), with a strict rule, a bounded context **never** imports another context's domain entities directly. Communication between contexts happens through:

- **Domain events**, published by one context and consumed asynchronously by another (see [[Transactional Outbox Pattern]]).
- **Explicit query ports**, when one context needs to read (not mutate) data owned by another, synchronously.
- Never: reaching directly into another module's `domain/` folder to reuse an entity or repository.

```
src/modules/
├── inventory/        ← one bounded context
│   └── domain/        ← never imported by other modules
├── donation/          ← another bounded context
│   └── domain/
```

## Context Mapping

Strategic DDD also names the *relationships* between bounded contexts, useful vocabulary for architecture discussions:

- **Shared Kernel**, two contexts deliberately share a small piece of model (e.g. a `Money` value object). Changes to it require both teams to agree.
- **Customer-Supplier**, one context's output feeds another's input; the downstream context depends on what the upstream one produces.
- **Anticorruption Layer (ACL)**, a translation layer that protects a context's clean model from a messy or legacy external system, converting external concepts into the context's own ubiquitous language at the boundary.
- **Open Host Service**, a context publishes a well-defined, stable API/protocol for others to consume, rather than negotiating a custom integration per consumer.

## Why this matters in practice

Getting bounded context boundaries wrong is expensive to fix later, it usually means one module's business rules leak into another's, and mutations meant to be atomic and independent end up entangled. A good sign a boundary is right: two contexts can each be understood, tested, and deployed without needing to reason about the other's internal state.

## See also

- [[Domain-Driven Design (DDD)]]
- [[Clean Architecture]]
- [[Transactional Outbox Pattern]]

Write by **Samuel**
