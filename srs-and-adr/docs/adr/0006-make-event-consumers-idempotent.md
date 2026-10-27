# ADR-0006: Make Event Consumers Idempotent

- **Status:** Proposed
- **Date:** 2026-10-27
- **Deciders:** Elfadel Mohamed, Rachid Boufous, Chedy
- **Related SRS items:** FR-23, FR-38, FR-50, FR-66, FR-73, NFR-18, NFR-56, BR-09, BR-15, BR-19, TBD-09

## Context

Kafka guarantees at-least-once delivery by default (ADR-0002), and the outbox relay may republish events (ADR-0005). A consumer can therefore receive the same logical event more than once, for example after a crash before its offset was committed or after a consumer-group rebalance. Without protection, a redelivered `PaymentApprovedEvent` would deduct stock twice (violating BR-15).

## Considered Options

1. **Rely on Kafka exactly-once semantics**
2. **Processed-event table**: record each handled event ID in the same transaction as the business change
3. **Natural idempotency** via unique business constraints (e.g., one payment row per order ID)

## Decision

We propose that every event carries a unique `eventId` (UUID), and that each consumer:

1. Starts a local database transaction.
2. Inserts the `eventId` into its `processed_events` table (primary key on `eventId`).
3. If the insert fails because the ID already exists, skips the event as a duplicate.
4. Otherwise performs the business change and writes any resulting event to its outbox, then commits.
5. Commits the Kafka offset only after the database transaction succeeds.

Where possible, we also add natural unique constraints (option 3) as a second safeguard, such as a unique `orderId` in the payments table.

Kafka exactly-once semantics only cover Kafka-to-Kafka processing and do not include the services' database writes, so option 1 is not sufficient on its own.

## Consequences

**Positive**

- Redelivered events have no additional business effect (FR-23, FR-38, FR-50).
- Works the same way in every service and is straightforward to test (NFR-69).

**Negative**

- Each service needs a database, including the Notification Service.
- The `processed_events` table grows and needs a retention policy.
- Adds one insert per event, which slightly lowers throughput.

**Follow-up**

- Add `eventId`, `correlationId`, `eventType`, `occurredAt`, and `schemaVersion` to all event schemas (TBD-04).
- Write tests that deliberately deliver the same event twice.
