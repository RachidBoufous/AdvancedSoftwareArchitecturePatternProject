# ADR-0002: Use Apache Kafka as the Event Broker

- **Status:** Accepted
- **Date:** 2026-10-26
- **Deciders:** Elfadel Mohamed, Rachid Boufous, Chedy
- **Related SRS items:** 3.3.2, 3.4.2, FR-72, NFR-46, NFR-50, TBD-12

## Context

ADR-0001 requires asynchronous communication between services. The broker must keep events available while a consumer is down so that it can catch up after recovery (NFR-46), allow several instances of a service to share the load (NFR-50), and allow new consumers to read existing event streams (NFR-53). The reference implementation [REF-01] already uses Kafka with Spring Boot.

## Considered Options

1. **Apache Kafka**
2. **RabbitMQ**
3. **Direct HTTP callbacks / webhooks** between services (no broker)

## Decision

We will use Apache Kafka, with one topic per event type:

| Topic               | Producer          | Consumer             |
| ------------------- | ----------------- | -------------------- |
| `order-created`     | Order Service     | Payment Service      |
| `payment-approved`  | Payment Service   | Inventory Service    |
| `inventory-updated` | Inventory Service | Notification Service |

Each consuming service uses its own consumer group. Events are keyed by order ID so that all events of one order land in the same partition and keep their order.

Kafka's retained, replayable log fits the recovery and modifiability goals better than RabbitMQ's delete-on-acknowledge queues. Keeping the reference implementation's broker also reduces project risk. Option 3 would reintroduce direct coupling.

## Consequences

**Positive**

- Events survive consumer outages within the retention period (FR-72).
- Consumer groups and partitions allow horizontal scaling of consumers.
- Any new service can subscribe to an existing topic without producer changes.

**Negative**

- Delivery is at-least-once by default, so duplicates must be handled (ADR-0006).
- Kafka is resource-heavy for a single VM and needs persistent storage in Kubernetes (TBD-16).
- Parallelism of a consumer group is capped by the number of partitions.

**Follow-up**

- Set partitions, replication, and retention per topic (TBD-12).
- Add topics for failure outcomes (`payment-rejected`, `inventory-failed`) when ADR-0007 is accepted.
