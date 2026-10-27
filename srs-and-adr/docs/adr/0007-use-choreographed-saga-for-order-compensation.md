# ADR-0007: Use a Choreographed Saga for Order Compensation

- **Status:** Proposed
- **Date:** 2026-10-27
- **Deciders:** Elfadel Mohamed, Rachid Boufous, Chedy
- **Related SRS items:** 4.2, 4.3, 4.5, FR-21, FR-37, FR-42, FR-61, BR-26 – BR-29, TBD-03, TBD-05, TBD-06, TBD-07

## Context

An order spans several services, each with its own database (ADR-0004), so there is no single transaction to roll back. Two failure cases need handling:

- **Payment rejected:** the order must end as `FAILED` and never reach inventory (BR-08).
- **Insufficient stock after payment approval:** the payment has already been approved, so it must be reversed (refunded in the simulation) and the order cancelled (BR-27).

The Order Service must also learn each outcome so it can update the order status (SRS 4.5).

## Considered Options

1. **Choreographed Saga**: each service reacts to events and publishes its own outcome events
2. **Orchestrated Saga**: a central orchestrator sends commands to each service
3. **No compensation**: mark failed orders only and resolve them manually

## Decision

We propose a **choreographed Saga** with these additional events:

| Event                  | Producer          | Consumers                      | Effect                                                                    |
| ---------------------- | ----------------- | ------------------------------ | ------------------------------------------------------------------------- |
| `PaymentRejectedEvent` | Payment Service   | Order Service                  | Order → `FAILED`                                                          |
| `InventoryFailedEvent` | Inventory Service | Payment Service, Order Service | Order → `CANCELLATION_PENDING`; Payment Service issues a simulated refund |
| `PaymentRefundedEvent` | Payment Service   | Order Service                  | Order → `CANCELLED`                                                       |

The Order Service also consumes `PaymentApprovedEvent` and `InventoryUpdatedEvent` to move the order to `PAYMENT_APPROVED` and `CONFIRMED`. Status transitions follow the state model in SRS Appendix B.3, and the Order Service ignores any event that would move an order backwards from a final state (FR-66, BR-24).

With only four services and a linear flow, choreography matches the existing event-driven design and avoids adding a fifth component. Option 3 does not satisfy BR-27.

## Consequences

**Positive**

- Each failure path has a defined, observable final state.
- No central coordinator to build, deploy, or keep available.
- Demonstrates the Saga pattern, a core topic of the course.

**Negative**

- The overall flow is spread across several services, which makes it harder to follow; the state diagram in the SRS becomes essential documentation.
- More topics and event types to maintain.
- Compensation events also need idempotent handling so a refund happens only once (BR-28, ADR-0006).

**Follow-up**

- Resolves TBD-03, TBD-05, and TBD-06 once accepted.
- Add failure test scenarios for both compensation paths (NFR-68).
