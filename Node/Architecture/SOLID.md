---
tags: [architecture, solid, oop, design-principles]
---

# SOLID

Five object-oriented design principles, popularized by Robert C. Martin ("Uncle Bob"). They're guidelines for writing code that's easier to change, test, and extend, not rules to apply mechanically everywhere.

## S, Single Responsibility Principle

A class/module should have **one reason to change**. Not "one method", one *responsibility*.

```ts
// Bad: mixes business rule with persistence and notification
class Order {
  checkout() {
    // calculates total
    // saves to the database
    // sends a confirmation email
  }
}

// Better: each responsibility in its own place
class Order {
  checkout(): OrderTotal { /* only calculates */ }
}
class OrdersRepository {
  save(order: Order): Promise<void> { /* only persists */ }
}
class OrderConfirmationNotifier {
  notify(order: Order): Promise<void> { /* only notifies */ }
}
```

## O, Open/Closed Principle

Open for extension, closed for modification. Add new behavior without touching code that already works, usually via abstraction (interface/polymorphism) instead of `if/switch` chains that grow forever.

```ts
// Bad: every new payment method means editing this method
function calculateFee(paymentMethod: string): number {
  if (paymentMethod === 'CREDIT_CARD') return 2.5;
  if (paymentMethod === 'PIX') return 0;
  // adding "BOLETO" means touching this function again
}

// Better: extend by adding a new class, not by editing existing ones
interface PaymentMethod {
  calculateFee(): number;
}
class CreditCard implements PaymentMethod {
  calculateFee() { return 2.5; }
}
class Pix implements PaymentMethod {
  calculateFee() { return 0; }
}
```

## L, Liskov Substitution Principle

A subtype must be substitutable for its base type without breaking the program's correctness. If `Square extends Rectangle` but changing `width` unexpectedly also changes `height`, code that expects `Rectangle` behavior breaks, that's a Liskov violation.

## I, Interface Segregation Principle

Prefer several small, specific interfaces over one large, general-purpose one. A class shouldn't be forced to implement methods it doesn't need.

```ts
// Bad: a read-only repository is forced to implement write methods
interface Repository<T> {
  findById(id: string): Promise<T | null>;
  save(entity: T): Promise<void>;
  delete(id: string): Promise<void>;
}

// Better: segregated by actual need
interface Reader<T> {
  findById(id: string): Promise<T | null>;
}
interface Writer<T> {
  save(entity: T): Promise<void>;
}
```

## D, Dependency Inversion Principle

High-level modules shouldn't depend on low-level modules, both should depend on abstractions. This is the principle behind [[Dependency Injection and Ports Adapters|Ports & Adapters]]: a use case depends on a repository *interface*, never on a concrete `PrismaRepository` class.

```ts
// The use case depends on the abstraction (port), not on Prisma directly
class CreateOrderUseCase {
  constructor(private readonly ordersRepository: IOrdersRepository) {}
}
```

## See also

- [[Clean Architecture]]
- [[Dependency Injection and Ports Adapters]]
- [[Domain-Driven Design (DDD)]]

Write by **Samuel**