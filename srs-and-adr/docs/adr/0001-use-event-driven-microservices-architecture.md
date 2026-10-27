# ADR-0001: Use Event-Driven Microservices Architecture

- **Status:** Accepted
- **Date:** 2026-10-26
- **Deciders:** Elfadel Mohamed, Rachid Boufous, Chedy
- **Related SRS items:** 1.4, 2.5.1, NFR-41 – NFR-55

## Context

In a synchronous order flow, the client waits while payment, inventory, and notification run one after another. A slow or failed step blocks or breaks the whole request, services are tightly coupled through direct calls, and individual stages cannot be scaled on their own (SRS 1.4, Problem Statement).

The project's main quality goals are responsiveness, fault isolation, scalability, and modifiability (SRS 5.4.9). The course also requires demonstrating an architectural pattern and its trade-offs.

## Considered Options

1. **Modular monolith** with synchronous in-process calls.
2. **Microservices with synchronous REST calls** between services (orchestrated by the Order Service).
3. **Microservices communicating through asynchronous events** (Microservices + Event-Driven Architecture).

## Decision

We will split the system into four independently deployable services (Order, Payment, Inventory, Notification) that communicate only through asynchronous events for the order-processing workflow. The Order Service is the only component exposed over HTTP.

Option 1 does not demonstrate fault isolation or independent scaling. Option 2 keeps the blocking dependencies and temporal coupling that motivated the project.

## Consequences

**Positive**

- The client gets an acknowledgement without waiting for downstream processing (NFR-41).
- A failed downstream service does not stop order acceptance (NFR-44, NFR-45).
- Each service can be scaled and redeployed on its own (NFR-48, NFR-49).
- New consumers can subscribe to existing events without changing producers (NFR-53).

**Negative**

- The system is eventually consistent: an order's status may lag behind the actual processing (SRS 4.5, Scenario 5).
- Events can be redelivered, so consumers must be idempotent (see ADR-0006).
- There is no distributed transaction across services; failures need compensation (see ADR-0007).
- Debugging spans several services and logs, which requires correlation IDs.

**Follow-up**

- Choose the event broker (ADR-0002).
- Define event schemas with event ID, order ID, type, timestamp, and correlation ID (TBD-04).
