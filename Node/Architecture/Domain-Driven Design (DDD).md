---
tags: [architecture, ddd, domain-driven-design]
---

# Domain-Driven Design (DDD)

An approach to software design that puts the **business domain** at the center of the model, and pushes for a shared **ubiquitous language** between developers and domain experts, the same terms used in conversation should show up as names in the code.

DDD splits into two halves:

- **Strategic design**, how to divide a large domain into manageable pieces. Core concept: [[Bounded Context]].
- **Tactical design**, the building blocks used to model a bounded context's domain in code. Covered below.

## Tactical building blocks

### Entity
An object with a persistent **identity** that matters more than its attributes, two entities with identical data are still different if their IDs differ. Identity survives state changes.

### Value Object (VO)
An object defined entirely by its **attributes**, with no identity of its own. Two Value Objects with the same values are interchangeable. Immutable by convention, any "change" produces a new instance.

```ts
class Money {
  private constructor(
    public readonly amount: number,
    public readonly currency: string,
  ) {}

  static create(amount: number, currency: string): Money {
    return new Money(amount, currency);
  }

  add(other: Money): Money {
    if (other.currency !== this.currency) throw new Error('Currency mismatch');
    return Money.create(this.amount + other.amount, this.currency);
  }
}
```

### Aggregate / Aggregate Root
A cluster of entities and value objects treated as a single **consistency boundary**, the only way in or out is through the Aggregate Root, which enforces every invariant of the objects it owns.

**A common modeling mistake:** nesting everything under one giant aggregate "because they're related". The real question is: *what needs to be transactionally consistent together, right now?* Two objects that change independently, at different times, by different actors, are usually two separate aggregates, even if one references the other by ID.

```ts
abstract class AggregateRoot<Id> {
  private readonly domainEvents: DomainEvent[] = [];
  protected constructor(public readonly id: Id) {}

  protected addDomainEvent(event: DomainEvent): void {
    this.domainEvents.push(event);
  }

  pullDomainEvents(): DomainEvent[] {
    const events = [...this.domainEvents];
    this.domainEvents.length = 0;
    return events;
  }
}
```

### Domain Event
Something meaningful that happened, expressed in the past tense (`OrderPlaced`, not `PlaceOrder`). Raised by an aggregate during a use case, and typically persisted for audit/integration purposes, see [[Transactional Outbox Pattern]].

### Domain Service
Business logic that doesn't naturally belong to a single entity or value object, usually because it involves several of them, or is a pure calculation with no state of its own.

```ts
class ShippingCostCalculator {
  calculate(weightInKg: number, destination: Address): Money { /* pure rule */ }
}
```

### Repository (as a domain-defined port)
An interface, defined in the domain/application layer, describing how an aggregate is persisted and retrieved, without exposing any database detail. The concrete implementation (Prisma, TypeORM, in-memory) lives in infrastructure. See [[Dependency Injection and Ports Adapters]].

## See also

- [[Bounded Context]]
- [[Clean Architecture]]
- [[Dependency Injection and Ports Adapters]]
- [[Transactional Outbox Pattern]]

Write by **Samuel**
