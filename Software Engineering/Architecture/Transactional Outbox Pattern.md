---
tags: [architecture, transactional-outbox, event-driven, ddd]
---

# Transactional Outbox Pattern

Solves a very specific, very common problem: **how do you atomically change an aggregate's state AND publish a domain event about it**, without a distributed transaction (2PC) spanning a database and a message broker?

## The problem

```ts
// Naive approach, two independent operations, not atomic
await ordersRepository.save(order);      // 1 writes to Postgres
await messageBroker.publish(orderEvent); // 2 publishes to SQS/Kafka
```

If the process crashes between 1 and 2, or 2 simply fails (network blip, broker down), the database says the order was placed, but nothing downstream ever finds out. The two systems (database, broker) don't share a transaction, so there's no way to guarantee both succeed or both fail together.

## The solution

Write the event to a table **in the same local database transaction** as the aggregate's state change. A separate, asynchronous worker later reads that table and publishes the events to the actual message broker (or writes them to an audit log).

```
1. BEGIN TRANSACTION
2.   UPDATE orders SET status = 'PLACED' WHERE id = ...
3.   INSERT INTO outbox_events (aggregate_id, event_name, payload, ...) VALUES (...)
4. COMMIT
   ──── same transaction, so both succeed or both roll back together ────

5. (async, separate process) worker polls outbox_events → publishes to the broker
   → marks the row as processed
```

Because step 3 is just one more `INSERT` inside a transaction the database was already doing, it adds negligible latency to the request, no synchronous network call to an external broker sits in the critical path.

## A minimal implementation shape

```ts
// Port, defined in the application layer
export interface IOutboxEventWriter {
  write(events: DomainEvent[]): Promise<void>;
}

// A use case, wiring both writes into one transaction
class PlaceOrderUseCase {
  async execute(input: PlaceOrderInput): Promise<void> {
    const order = Order.place(input);

    await this.transactionRunner.runInTransaction(async (scope) => {
      await this.ordersRepository.save(order, scope);
      await this.outboxEventWriter.write(order.pullDomainEvents(), scope);
    });
  }
}
```

The transactional guarantee only holds if `save()` and `write()` genuinely run inside the same database transaction, passing an opaque transaction scope through both calls (rather than each opening its own connection/transaction) is what makes that true, not just implied by calling them one after another.

## Trade-offs

- **At-least-once delivery.** If the worker crashes after publishing but before marking a row processed, that event gets published again on restart. Consumers of these events need to be **idempotent**, processing the same event twice must be safe.
- **Extra infrastructure.** Requires a worker process (or scheduled job) polling the outbox table, plus a strategy for what "processed" means and how old processed rows get cleaned up.
- **Not real-time.** There's a small delay between the transaction committing and the event actually reaching consumers, usually fine for audit trails and cross-[[Bounded Context]] integration, not fine for something requiring sub-millisecond propagation.

## Where it fits

This pattern is what makes reliable event publishing possible across [[Bounded Context]] boundaries in a [[Domain-Driven Design (DDD)]] system, each context can own its own outbox table and publish its own domain events without ever needing a distributed transaction with another context's database.

## See also

- [[Domain-Driven Design (DDD)]]
- [[Bounded Context]]
- [[Clean Architecture]]

Write by **Samuel**