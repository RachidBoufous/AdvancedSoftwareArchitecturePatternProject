# ADR-0005: Use the Transactional Outbox Pattern for Event Publication

- **Status:** Proposed
- **Date:** 2026-10-27
- **Deciders:** Elfadel Mohamed, Rachid Boufous, Chedy
- **Related SRS items:** FR-12, FR-76, FR-77, NFR-03, NFR-57, TBD-08

## Context

When an order is accepted, the Order Service must both save the order and publish `OrderCreatedEvent`. These are two separate systems (database and Kafka) with no shared transaction. This is the "dual write" problem:

- If the order is saved but publishing fails, the client is told the order was accepted, but nothing downstream ever happens (violates FR-12).
- If the event is published but saving fails, downstream services process an order that does not exist.

The same problem applies to every service that changes its own data and then publishes an event.

## Considered Options

1. **Publish directly after saving** (current reference implementation)
2. **Kafka transactions** combined with the database transaction
3. **Transactional Outbox** with a polling relay
4. **Transactional Outbox with Change Data Capture** (e.g., Debezium)

## Decision

We propose the **Transactional Outbox pattern with a polling relay**:

1. In one local database transaction, the service saves the business change and inserts the event into an `outbox` table.
2. A scheduled relay in the same service reads unpublished outbox rows, publishes them to Kafka, and marks them as published only after Kafka acknowledges them.

Option 1 is the problem itself. Option 2 cannot make the database and Kafka atomic together. Option 4 is robust but needs an extra Kafka Connect/Debezium deployment, which is too heavy for one VM.

## Consequences

**Positive**

- An acknowledged order always gets its event eventually, even after a Kafka outage (FR-12, FR-77).
- No event is ever published for data that was not committed.
- Simple to implement with Spring's `@Transactional` and `@Scheduled`.

**Negative**

- The relay may publish an event more than once (for example, if it crashes after publishing but before marking the row). Consumers must be idempotent (ADR-0006).
- Polling adds a small delay (the polling interval) to end-to-end latency (NFR-04).
- The outbox table must be cleaned up periodically.

**Follow-up**

- Choose the polling interval and cleanup policy.
- Apply the same pattern to the Payment and Inventory services when they become database-backed.
