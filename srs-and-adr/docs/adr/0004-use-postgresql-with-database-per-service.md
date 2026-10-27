# ADR-0004: Use PostgreSQL with a Database per Service

- **Status:** Proposed
- **Date:** 2026-10-27
- **Deciders:** Elfadel Mohamed, Rachid Boufous, Chedy
- **Related SRS items:** 2.5.5, 6.1, 6.4, FR-06, FR-30, FR-99, TBD-01, TBD-02

## Context

The reference implementation stores orders in an in-memory H2 database, so all data is lost whenever the Order Service pod restarts. That conflicts with FR-99 and SRS 6.4. Several new requirements also need persistent state in other services: stock levels (FR-30), processed-event records for idempotency (ADR-0006), and an outbox table (ADR-0005).

## Considered Options

1. **Keep H2 in-memory**
2. **H2 in file mode** on a persistent volume
3. **PostgreSQL, one shared database for all services**
4. **PostgreSQL, a separate database (schema and user) per service**

## Decision

We propose PostgreSQL running as a single StatefulSet, with a **separate logical database and user per service** (`orders`, `payments`, `inventory`, `notifications`). No service may read or write another service's database.

Option 1 fails the restart requirement. Option 2 is not well suited to concurrent access or production-like behavior. Option 3 would couple services through shared tables, undermining ADR-0001. One PostgreSQL instance with separate databases keeps resource use low on the VM while keeping data ownership separate.

## Consequences

**Positive**

- Data survives pod restarts (FR-99).
- Transactions and row locking support the outbox, idempotency, and safe stock updates (FR-33, FR-39).
- Each service owns its data, so schemas can change independently.

**Negative**

- One PostgreSQL instance is a shared failure point for all services.
- Cross-service queries are not possible; data needed elsewhere must travel in events.
- Adds a stateful component that needs a persistent volume and backups.

**Follow-up**

- Replace H2 in the reference services and add schema migrations (e.g., Flyway).
- Store credentials in Kubernetes Secrets.
