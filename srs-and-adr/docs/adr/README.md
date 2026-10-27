# Architecture Decision Records

This folder contains the Architecture Decision Records (ADRs) for the **Event-Driven Order Processing System** (Advanced Software Architecture Patterns course, University of Debrecen).

An ADR captures one architecturally significant decision: the context that forced it, the decision itself, the options that were considered, and the consequences. The format follows the guidance at [architecture-decision-record](https://github.com/architecture-decision-record/architecture-decision-record), using Michael Nygard's template extended with a "Considered Options" section (see [`template.md`](template.md)).

**Team:** Elfadel Mohamed, Rachid Boufous, Chedy

## Conventions

- One decision per file, named `NNNN-imperative-verb-phrase.md` (e.g., `0002-use-apache-kafka-as-event-broker.md`).
- Numbers are never reused. A changed decision gets a new ADR that marks the old one as **Superseded**.
- Status values: **Proposed**, **Accepted**, **Rejected**, **Deprecated**, **Superseded**.
- Each ADR links to the SRS requirements (`FR-`, `NFR-`, `BR-`) and TBD items it addresses.

## Index

| ADR                                                                | Title                                                      | Status   | Related SRS items                  |
| ------------------------------------------------------------------ | ---------------------------------------------------------- | -------- | ---------------------------------- |
| [ADR-0001](0001-use-event-driven-microservices-architecture.md)    | Use event-driven microservices architecture                | Accepted | 2.5.1, NFR-41 – NFR-55             |
| [ADR-0002](0002-use-apache-kafka-as-event-broker.md)               | Use Apache Kafka as the event broker                       | Accepted | 3.3.2, 3.4.2, TBD-12               |
| [ADR-0003](0003-deploy-on-kubernetes-on-a-virtual-machine.md)      | Deploy on Kubernetes on a virtual machine                  | Accepted | 4.7, FR-84 – FR-100, TBD-13        |
| [ADR-0004](0004-use-postgresql-with-database-per-service.md)       | Use PostgreSQL with a database per service                 | Proposed | 6.1, TBD-01, TBD-02                |
| [ADR-0005](0005-use-transactional-outbox-for-event-publication.md) | Use the Transactional Outbox pattern for event publication | Proposed | FR-12, FR-76, FR-77, TBD-08        |
| [ADR-0006](0006-make-event-consumers-idempotent.md)                | Make event consumers idempotent                            | Proposed | FR-23, FR-38, FR-50, FR-73, TBD-09 |
| [ADR-0007](0007-use-choreographed-saga-for-order-compensation.md)  | Use a choreographed Saga for order compensation            | Proposed | 4.5, BR-26 – BR-29, TBD-07         |
| [ADR-0008](0008-use-bounded-retries-and-dead-letter-topics.md)     | Use bounded retries and dead-letter topics                 | Proposed | FR-80 – FR-82, TBD-10, TBD-11      |

When a Proposed ADR is accepted, update its status here and in the file, and set the matching TBD item in SRS Appendix C to **Resolved**.
