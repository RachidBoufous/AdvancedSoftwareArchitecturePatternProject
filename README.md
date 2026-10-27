# Event-Driven Order Processing System

A software architecture analysis and design for an event-driven microservices system that processes customer orders asynchronously using Apache Kafka. Developed for the **Advanced Software Architecture Patterns** course at the University of Debrecen.

> **Note:** This repository contains the architectural analysis and documentation only. Implementation will follow in a later phase.

## Overview

When a customer places an order, the system must create the order, process payment, update inventory, and send a notification. Instead of chaining these steps with synchronous calls — where a slow or unavailable service stalls the entire workflow — each service communicates through Kafka events. The Order Service publishes an `OrderCreatedEvent`, and downstream services react independently.

```
Client: POST /orders
        │
        ▼
  ┌─────────────┐    order-created     ┌─────────────────┐
  │ Order       │ ──────────────────►  │ Payment         │
  │ Service     │                      │ Service         │
  └─────────────┘                      └────────┬────────┘
        ▲                                       │
        │ (status updates)          payment-approved / rejected
        │                                       │
        │                                       ▼
        │                              ┌─────────────────┐
        │                              │ Inventory       │
        │                              │ Service         │
        │                              └────────┬────────┘
        │                                       │
        │                              inventory-updated / failed
        │                                       │
        │                                       ▼
        │                              ┌─────────────────┐
        └──────────────────────────────│ Notification    │
                                       │ Service         │
                                       └─────────────────┘
```

## Planned Architecture

| Pattern | Description |
|---------|-------------|
| Event-Driven Microservices | Four services (Order, Payment, Inventory, Notification) communicating via async events |
| Message Broker | Apache Kafka with topics keyed by `orderId` for ordering guarantees |
| Data Persistence | PostgreSQL with a separate database per service to preserve bounded contexts |
| Transactional Outbox | Business change + outbox row in one DB transaction; a relay publishes to Kafka |
| Idempotent Consumers | `processed_events` table deduplicates redelivered events by `eventId` |
| Choreographed Saga | Compensation events handle payment rejection and out-of-stock scenarios |
| Dead-Letter Topics | Failed events move to `<topic>.DLT` after 3 retries with exponential backoff |
| Deployment | Kubernetes on a single VM (Deployments, StatefulSets, ConfigMaps, Secrets) |

## Planned Tech Stack

- **Language:** Java 17+
- **Framework:** Spring Boot, Spring Kafka
- **Messaging:** Apache Kafka
- **Database:** PostgreSQL
- **Orchestration:** Kubernetes
- **Build:** Maven

## Order Lifecycle

An order moves through the following states:

```
PENDING → PAYMENT_APPROVED → CONFIRMED
                            ↘ FAILED
       → CANCELLATION_PENDING → CANCELLED
```

## Repository Structure

```
└── srs-and-adr/                # Project documentation
    ├── SRS.md                   # Software Requirements Specification (IEEE template)
    └── docs/adr/                # Architecture Decision Records
        ├── README.md            # ADR index and conventions
        ├── template.md          # Blank ADR template
        ├── 0001-use-event-driven-microservices-architecture.md
        ├── 0002-use-apache-kafka-as-event-broker.md
        ├── 0003-deploy-on-kubernetes-on-a-virtual-machine.md
        ├── 0004-use-postgresql-with-database-per-service.md
        ├── 0005-use-transactional-outbox-for-event-publication.md
        ├── 0006-make-event-consumers-idempotent.md
        ├── 0007-use-choreographed-saga-for-order-compensation.md
        └── 0008-use-bounded-retries-and-dead-letter-topics.md
```

## Documentation

- [Software Requirements Specification (SRS)](srs-and-adr/SRS.md) — full system requirements following the IEEE/Wiegers template
- [Architecture Decision Records](srs-and-adr/docs/adr/) — eight ADRs documenting every major design choice

## Team

| Name | Role |
|------|------|
| Elfadel Mohamed | Software Architect |
| Rachid Boufous | Software Architect |
| Chedy Medo | Software Architect |

## License

This project was developed as a university assignment and is not licensed for production use.
