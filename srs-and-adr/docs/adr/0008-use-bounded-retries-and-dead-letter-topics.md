# ADR-0008: Use Bounded Retries and Dead-Letter Topics

- **Status:** Proposed
- **Date:** 2026-10-27
- **Deciders:** Elfadel Mohamed, Rachid Boufous, Chedy
- **Related SRS items:** 4.6, FR-24, FR-71, FR-79 – FR-83, TBD-10, TBD-11

## Context

Consumers can fail for two different reasons:

- **Temporary technical failures**, such as a database that is briefly unavailable. Retrying usually helps.
- **Permanent failures**, such as a malformed message ("poison pill"). Retrying never helps, and endless retries block every later event in the same partition.

Failed events must not be silently dropped (FR-24, FR-71) and must not block the rest of the system. Note that business rejections (payment rejected, insufficient stock) are not errors: they are handled as normal outcome events (ADR-0007, FR-75).

## Considered Options

1. **Retry forever**
2. **Log and skip** failed events
3. **Bounded retries with backoff, then move the event to a dead-letter topic**

## Decision

We propose using Spring Kafka's `DefaultErrorHandler` with a `DeadLetterPublishingRecoverer`:

- Retry a failed event **3 times with exponential backoff** (starting at 1 second, doubling each time). The values are configurable (FR-80).
- Do not retry exceptions marked as non-retryable, such as deserialization or validation errors.
- After the last attempt, publish the event to `<topic>.DLT` (e.g., `order-created.DLT`) with headers recording the original topic, the exception, and the timestamp, and then commit the offset.
- Log every failure with the event ID and order ID (FR-74).
- Replay events from a DLT manually once the cause is fixed (FR-82).

Option 1 blocks the partition indefinitely on a poison pill. Option 2 loses events and violates FR-71.

## Consequences

**Positive**

- Temporary failures recover automatically.
- Poison pills are isolated and no longer block other orders.
- Failed events remain inspectable and replayable (FR-79, FR-83).

**Negative**

- Retries delay the events behind the failed one in the same partition.
- DLT events need someone to review and replay them; there is no automatic resolution.
- Replayed events rely on idempotent consumers (ADR-0006).

**Follow-up**

- Create the DLT topics in the Kafka configuration (TBD-12).
- Document the replay procedure in the deployment guide.
