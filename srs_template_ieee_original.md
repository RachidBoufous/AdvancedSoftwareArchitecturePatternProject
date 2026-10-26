# Software Requirements Specification

## for

# Event-Driven Order Processing System

Version 1.0

Prepared by Rachid Boufous, 

University of Debrecen

Faculty: Faculty of Informatics

Date: 26 October 2026

Copyright © 1999 by Karl E. Wiegers. Permission is granted to use, modify, and distribute this document.

## Table of Contents

- Table of Contents
- Revision History
- 1. Introduction
  - 1.1 Purpose
  - 1.2 Document Conventions
  - 1.3 Intended Audience and Reading Suggestions
  - 1.4 Product Scope
  - 1.5 References
- 2. Overall Description
  - 2.1 Product Perspective
  - 2.2 Product Functions
  - 2.3 User Classes and Characteristics
  - 2.4 Operating Environment
  - 2.5 Design and Implementation Constraints
  - 2.6 User Documentation
  - 2.7 Assumptions and Dependencies
- 3. External Interface Requirements
  - 3.1 User Interfaces
  - 3.2 Hardware Interfaces
  - 3.3 Software Interfaces
  - 3.4 Communications Interfaces
- 4. System Features
  - 4.1 System Feature 1
  - 4.2 System Feature 2 (and so on)
- 5. Other Nonfunctional Requirements
  - 5.1 Performance Requirements
  - 5.2 Safety Requirements
  - 5.3 Security Requirements
  - 5.4 Software Quality Attributes
  - 5.5 Business Rules
- 6. Other Requirements
- Appendix A: Glossary
- Appendix B: Analysis Models
- Appendix C: To Be Determined List

## Revision History

| Name | Date | Reason For Changes | Version |
|------|------|--------------------|---------|
| Rachid Boufous and Team | 26/10/2026 | Initial creation of the Software Requirements Specification | 1.0 (Draft) |

# 1. Introduction

## 1.1 Purpose

The purpose of this Software Requirements Specification (SRS) document is to define the functional and non-functional requirements of the Event-Driven Order Processing System, an application designed to demonstrate asynchronous order processing using Microservices and Event-Driven Architecture.

The system consists of four main microservices: Order Service, Payment Service, Inventory Service, and Notification Service. These services communicate asynchronously through Apache Kafka to process customer orders without relying on direct synchronous communication between services.

This document specifies the expected system behavior, external interfaces, functional requirements, performance expectations, quality attributes, and technical constraints. It also establishes the requirements for containerization and deployment using Docker and Kubernetes on a virtual machine.

The SRS serves as a reference for the development team throughout the design, implementation, testing, and deployment phases of the project. It also provides the course instructor with a basis for evaluating whether the implemented system satisfies the defined requirements.

This specification covers the complete order-processing workflow, from order creation to payment processing, inventory updates, and customer notification. The system is intended as an educational prototype, with payment processing and customer notifications simulated rather than integrated with real-world payment or messaging providers.

## 1.2 Document Conventions

This document follows the structure of the IEEE Software Requirements Specification (SRS) template provided for the Advanced Software Architecture Patterns course. Markdown formatting is used to organize sections, subsections, tables, and requirement descriptions.

The following conventions are applied throughout the document:

- **Functional Requirements (FR):** Identified using the format `FR-XX` (e.g., `FR-01`, `FR-02`). These describe the functions and behaviors the system must provide.
- **Non-Functional Requirements (NFR):** Identified using the format `NFR-XX` (e.g., `NFR-01`, `NFR-02`). These describe system quality attributes, such as performance, reliability, security, and scalability.
- **Business Rules (BR):** Identified using the format `BR-XX` (e.g., `BR-01`). These define conditions and constraints governing the order-processing workflow.
- **MUST:** Indicates a mandatory requirement that must be satisfied by the completed system.
- **SHOULD:** Indicates a recommended requirement that is desirable but not mandatory.
- **TBD (To Be Determined):** Indicates information or decisions that have not yet been finalized.

Requirements are assigned one of the following priority levels:

| Priority | Description |
|----------|-------------|
| High | Essential functionality or constraints required for the system to operate correctly and satisfy the project's main objectives. |
| Medium | Important functionality that improves system reliability, usability, or maintainability but is not essential to the basic workflow. |
| Low | Optional enhancements that may be implemented if time and resources permit. |

Each requirement is assigned a unique identifier to support traceability during implementation and testing. Requirement identifiers remain unchanged when the document is revised.

Technical terms, such as *microservices*, *event-driven architecture*, and *Apache Kafka*, are used consistently throughout the document. Abbreviations and specialized terminology are defined in Appendix A: Glossary.

Any sections that are not applicable to the project are retained and explicitly marked **Not Applicable (N/A)**, while unresolved requirements are marked **TBD**.

## 1.3 Intended Audience and Reading Suggestions

This Software Requirements Specification is intended for the following audiences:
- Course Instructor: To review the project's scope, functional and non-functional requirements, architectural constraints, and expected system behavior, and to evaluate whether the completed implementation satisfies the specified requirements.
- Development Team: To understand the required system functionality, service responsibilities, communication mechanisms, and technical constraints during design and implementation.
- Testers: To identify expected system behaviors and acceptance criteria, design test cases, and verify that functional and non-functional requirements are satisfied.
- System Administrators and Operators: To understand the system's operating environment, deployment constraints, external interfaces, and reliability requirements when deploying and managing the application using Docker and Kubernetes.
The document is organized into six main chapters and three appendices:
- Chapter 1 – Introduction: Describes the document's purpose, conventions, intended audience, project scope, and references.
- Chapter 2 – Overall Description: Provides an overview of the system, its major functions, users, operating environment, constraints, and dependencies.
- Chapter 3 – External Interface Requirements: Defines the interfaces through which the system interacts with users, hardware, external software, and communication infrastructure.
- Chapter 4 – System Features: Specifies the system's functional requirements, including order creation, payment processing, inventory updates, and customer notifications.
- Chapter 5 – Other Nonfunctional Requirements: Defines performance, security, reliability, scalability, maintainability, and other quality requirements.
- Chapter 6 – Other Requirements: Covers additional requirements not addressed in the preceding chapters.
- Appendices: Provide definitions of technical terminology, supporting analysis models, and a list of unresolved decisions.
Readers unfamiliar with the project are encouraged to begin with Chapters 1 and 2 to understand its objectives and overall functionality.

Developers and testers should then focus on Chapters 3, 4, and 5, while system administrators and operators should pay particular attention to the operating environment, implementation constraints, communication interfaces, and deployment-related requirements.

## 1.4 Product Scope


The Event-Driven Order Processing System is a distributed application designed to simulate and manage the main stages of an online order-processing workflow. The system handles order creation, payment processing, inventory updates, and customer notifications through independently deployable microservices.

Problem Statement

In a traditional synchronous order-processing system, different operations may be connected through a sequence of direct service-to-service requests. When a customer places an order, the system may need to wait for payment processing, inventory updates, and notification services to complete before returning a response.

This approach can introduce several architectural problems:

Blocking dependencies: Slow downstream operations can increase the overall response time.

Tight coupling: Services depend directly on the availability and interfaces of other services.

Limited fault isolation: A failure in one service can disrupt the entire order-processing workflow.

Difficult scalability: Individual processing stages may be difficult to scale independently.

Reduced modifiability: Adding new functionality may require changes to existing service interactions.

Proposed Solution

The proposed system addresses these problems by combining Microservices Architecture with Event-Driven Architecture (EDA).

The application is divided into four main microservices:

Order Service: Receives customer orders, stores order information, and publishes an order-created event.

Payment Service: Consumes order-created events, simulates payment processing, and publishes payment-result events.

Inventory Service: Processes successful payment events, performs inventory operations, and publishes inventory-update events.

Notification Service: Consumes inventory-update events and generates customer confirmation notifications.

Instead of directly invoking each other for business operations, the services communicate asynchronously through Apache Kafka, which acts as the event broker.

This allows the Order Service to acknowledge an accepted order without waiting for all downstream operations to complete. Other services process the relevant events independently as they become available.

Project Objectives

The main objectives of the system are:

Responsiveness: Reduce blocking dependencies by allowing order requests to be acknowledged before downstream processing finishes.

Fault Isolation: Limit the impact of individual service failures on the rest of the order-processing system.

Scalability: Enable individual microservices to be scaled independently according to their processing workload.

Modifiability: Allow new event consumers or processing capabilities to be introduced with minimal changes to existing services.

Reliability: Support consistent order processing and recovery from temporary service failures.

Deployability: Package the application using Docker containers and deploy the microservices on a virtual machine using Kubernetes.

Project Boundaries

The project focuses on demonstrating the architectural behavior of a distributed, event-driven order-processing system.

In Scope:

Creation and basic validation of customer orders.

Asynchronous communication using Apache Kafka.

Simulated payment processing.

Inventory processing and updates.

Simulated customer notifications.

Basic order status tracking.

Containerization of application services using Docker.

Deployment and management using Kubernetes.

Testing of service communication, failure handling, and selected architectural quality attributes.

Out of Scope:

Integration with real banks or payment gateways.

Processing actual financial transactions.

Integration with commercial shipping or delivery providers.

Implementation of a complete e-commerce storefront.

Real-world customer billing and invoicing.

Production-scale deployment across multiple cloud regions.

The system is intended as an educational prototype for the Advanced Software Architecture Patterns course at the University of Debrecen. Its primary purpose is to demonstrate and evaluate architectural patterns rather than provide a production-ready e-commerce platform.

## 1.5 References

The following documents and resources were used as references for preparing this Software Requirements Specification.

[REF-01] Event-Driven Order System — Reference Implementation

Author: Flavio Carvalho (FL4V10ARC)

Description: Open-source reference implementation of an event-driven order-processing system using Spring Boot microservices and Apache Kafka.

Source: GitHub — Event-Driven Order System

Accessed: 26 October 2026

[REF-02] Advanced Software Architecture Project Proposal

Author: Project Team

Document: Advanced Software Architecture Project Proposal

Description: Academic project proposal describing the problem of synchronous order processing, the proposed Event-Driven Microservices Architecture, its expected quality attributes, and architectural trade-offs.

Source: Project documentation provided for the Advanced Software Architecture Patterns course, University of Debrecen.

[REF-03] Ordering System — Event-Driven Architecture

Author: Project Team

Document: Ordering System: Event-Driven Microservices with Apache Kafka

Description: Project presentation introducing the order-processing workflow, architectural problems, proposed solution, benefits, and trade-offs.

Source: Project presentation prepared for the Advanced Software Architecture Patterns course, University of Debrecen.

[REF-04] IEEE Software Requirements Specification Template

Original Author: Karl E. Wiegers

Document: Software Requirements Specification Template

Description: IEEE-style SRS template used to structure and document the system's functional and non-functional requirements.

Source: SRS template provided by the course instructor.

Original Copyright: © 1999 Karl E. Wiegers.

[REF-05] Architecture Decision Records

Author: Architecture Decision Record community

Document: Architecture Decision Record (ADR)

Description: Reference material for documenting important software architecture decisions, including their context, rationale, and consequences.

Source: GitHub — Architecture Decision Records

Accessed: 26 October 2026.

# 2. Overall Description

## 2.1 Product Perspective

2.1 Product Perspective
The Event-Driven Order Processing System is a distributed, event-driven application designed to demonstrate asynchronous communication and independent service deployment within an online order-processing workflow.

The project is based on the open-source Event-Driven Order System developed by Flavio Carvalho. The reference implementation provides a foundation consisting of four Spring Boot microservices communicating through Apache Kafka. The proposed project extends this foundation to support additional requirements, improved reliability, containerization, and deployment using Kubernetes.

The system is designed as a self-contained educational prototype rather than as a replacement for an existing commercial e-commerce platform.
System Architecture
The system follows a combination of Microservices Architecture and Event-Driven Architecture (EDA).

Its main components are:
1. Order Service: Provides the HTTP interface for creating orders, validates incoming requests, stores order information, and publishes events when orders are created.
2. Payment Service: Receives order-created events, performs simulated payment processing, and publishes payment-result events.
3. Inventory Service: Receives successful payment events, processes inventory updates, and publishes events indicating successful inventory operations.
4. Notification Service: Receives inventory-update events and generates simulated customer confirmations.
5. Apache Kafka: Acts as the event broker, enabling asynchronous communication between the microservices.
Each microservice has a separate business responsibility and communicates with other services through events rather than direct synchronous business requests.
System Interaction
The order-processing workflow begins when a client submits an order through the Order Service's HTTP API.

After validating and accepting the order, the Order Service stores the order information and publishes an OrderCreatedEvent to Apache Kafka.

The Payment Service consumes this event and performs simulated payment processing. When payment is approved, it publishes a PaymentApprovedEvent.

The Inventory Service consumes the payment-approved event, performs an inventory operation, and publishes an InventoryUpdatedEvent.

Finally, the Notification Service consumes the inventory-updated event and generates a simulated order confirmation.

Because these operations are performed asynchronously, the initial order request does not need to wait for the entire processing workflow to finish.
Deployment Perspective
The completed application is intended to run in a containerized environment.

Each microservice will be packaged as a Docker container image and deployed within a Kubernetes cluster hosted on a virtual machine.

Kubernetes will be responsible for managing application containers, service networking, deployment configuration, and container lifecycle operations.

Apache Kafka will provide the messaging infrastructure required for asynchronous communication between the services.

The deployment architecture is intended to demonstrate independent service management, fault isolation, and horizontal scaling within the resource limitations of the virtual machine.
External Dependencies
The system depends on the following external technologies and infrastructure:
- Apache Kafka: Event streaming and asynchronous message delivery.
- Java and Spring Boot: Application development and execution.
- Database Management System: Storage of order information and other persistent application data.
- Docker: Packaging and distribution of the application services.
- Kubernetes: Container orchestration and service deployment.
- Virtual Machine: Infrastructure hosting the Kubernetes cluster.
The system does not require integration with external banking services, commercial payment gateways, or real customer notification providers. Such operations are simulated for educational purposes.

## 2.2 Product Functions

2.2 Product Functions
The Event-Driven Order Processing System provides a set of interconnected functions that support asynchronous order processing through independent microservices.

The major functions of the system are described below.
1. Order Creation and Management
The system allows clients to submit orders through an HTTP API.

The Order Service validates incoming order information, creates an order record, assigns a unique identifier, and publishes an event indicating that a new order has been created.

The system acknowledges accepted orders without waiting for payment, inventory, and notification processing to complete.
2. Payment Processing
The Payment Service automatically processes newly created orders by consuming order-created events from Apache Kafka.

Payment processing is simulated for educational purposes and does not involve real financial transactions.

The system is intended to support successful and unsuccessful payment outcomes. Successful payments trigger the next stage of the order-processing workflow.
3. Inventory Processing
The Inventory Service processes inventory operations after receiving events indicating successful payment.

It is responsible for updating the inventory state associated with an order and publishing an event when the inventory operation completes successfully.

The proposed system also aims to support handling of inventory-related failures, such as insufficient stock.
4. Customer Notification
The Notification Service processes events generated after successful inventory operations.

It generates simulated customer confirmations indicating that an order has completed the required processing stages.

Notifications are handled asynchronously so that notification processing does not block order creation or other completed operations.
5. Order Status Tracking
The system is intended to provide a mechanism for retrieving the current processing status of an order.

This allows clients to distinguish between an order that has been accepted and an order that has completed the necessary payment and inventory operations.

Order status information reflects the asynchronous nature of the system, where different processing stages may complete at different times.
6. Event-Based Communication
The system uses Apache Kafka to exchange events between microservices.

Each service consumes events relevant to its responsibilities and publishes new events when appropriate.

This communication mechanism reduces direct dependencies between services and allows individual processing stages to operate independently.
7. Failure Handling and Recovery
The system is intended to support recovery from temporary service failures.

When a downstream service becomes unavailable, previously published events may remain available in Kafka for processing after the service recovers, subject to message retention and consumer configuration.

The proposed system will include mechanisms for handling processing failures and reducing the risk of duplicate operations.
8. Containerized Deployment and Management
The application will support containerized deployment using Docker and Kubernetes.

Each microservice will be packaged into a container image and deployed independently within a Kubernetes cluster hosted on a virtual machine.

The deployment will allow operators to manage service instances, inspect application logs, restart failed containers, and demonstrate independent service scaling.

These functions collectively support the project's objective of demonstrating responsiveness, fault isolation, scalability, and modifiability through Microservices and Event-Driven Architecture.

## 2.3 User Classes and Characteristics

The Event-Driven Order Processing System is designed primarily as an educational prototype for demonstrating distributed software architecture. Users interact with the system through its HTTP API, development tools, and deployment infrastructure rather than through a dedicated graphical user interface.

The following user classes have been identified:
1. API Client (Simulated Customer)
The API Client represents a customer or external application that interacts with the Order Service.

Its primary responsibility is to submit order requests and retrieve information about the processing status of submitted orders.

Characteristics:
- Interacts with the system through HTTP requests.
- Does not require knowledge of the internal microservices architecture.
- Expects valid order requests to be acknowledged without waiting for all downstream operations to complete.
- May retrieve order-processing information through a status endpoint.
- Does not interact directly with Apache Kafka or individual downstream microservices.
Priority: High
2. System Administrator / Operator
The System Administrator is responsible for deploying, configuring, monitoring, and maintaining the application infrastructure.

This user interacts with Docker, Kubernetes, and application monitoring or logging tools.

Characteristics:
- Possesses technical knowledge of containerization and distributed systems.
- Deploys and manages the application within the Kubernetes cluster.
- Monitors application health, service availability, and processing errors.
- Inspects service logs to diagnose problems.
- Restarts or scales individual services when necessary.
- Manages application configuration and infrastructure resources.
Priority: High
3. Developer / Tester
The Developer or Tester is responsible for implementing, maintaining, and verifying the system's functionality and architectural behavior.

This user interacts with the application source code, development environment, HTTP API, event broker, and testing tools.

Characteristics:
- Possesses knowledge of Java, Spring Boot, Apache Kafka, and microservices.
- Develops and maintains individual application services.
- Tests the complete order-processing workflow.
- Verifies event production and consumption between services.
- Simulates service failures and evaluates recovery behavior.
- Validates functional and non-functional requirements.
- Investigates system behavior using application logs and testing tools.
Priority: High
User Class Summary
User Class	Primary Interaction	Main Responsibilities	Priority
API Client (Simulated Customer)	HTTP API	Submit orders and retrieve order status	High
System Administrator / Operator	Docker, Kubernetes, system logs	Deploy, monitor, configure, and manage services	High
Developer / Tester	Source code, HTTP API, Kafka, testing tools	Implement, test, debug, and verify the system	HighThe system does not currently require separate administrative interfaces, customer authentication, or role-based access control for its educational demonstration.

Additional user classes may be introduced if the project scope is expanded in future versions.

## 2.4 Operating Environment

The Event-Driven Order Processing System is designed to operate in a containerized environment hosted on a virtual machine (VM). The application consists of independent Java-based microservices that communicate asynchronously using Apache Kafka.

The project supports two operating environments: a local development environment and a Kubernetes-based deployment environment.
2.4.1 Local Development Environment
The local development environment is used for implementing, testing, and debugging the microservices before deployment.

The reference implementation uses the following technologies:
Component	Technology	Purpose
Programming Language	Java 21	Implementation of the microservices
Application Framework	Spring Boot 3.3.5	Development of standalone application services
Build Tool	Apache Maven	Dependency management and application builds
Message Broker	Apache Kafka	Asynchronous communication between services
Database	H2 Database	In-memory order storage in the reference implementation
Containerization	Docker	Packaging and running containerized components
Local Infrastructure	Docker Compose	Running Kafka infrastructure during development
Operating System	Windows or Linux	Local development and testingDevelopment tools such as an IDE, HTTP API testing client, and Git may be used to implement and verify the application.
2.4.2 Deployment Environment
The completed system will be deployed on a virtual machine using a Kubernetes-based container orchestration environment.

The target deployment will contain the four application microservices, Apache Kafka, and the database components required for the selected persistence strategy.
Component	Target Environment
Hosting Infrastructure	Virtual Machine (VM)
Operating System	Linux-based Kubernetes host
Container Images	Docker-built OCI-compatible images
Container Orchestration	Kubernetes
Application Runtime	Java 21
Application Framework	Spring Boot
Event Broker	Apache Kafka
Database	To Be Determined (TBD)
Application Networking	Kubernetes Services
External API Access	Kubernetes networking or ingress configuration (TBD)Each microservice will run as an independently managed Kubernetes workload.

Kubernetes will provide service discovery, container lifecycle management, restart capabilities, and configurable application scaling.
2.4.3 Hardware Requirements
The deployment environment must provide sufficient computing resources to execute the application microservices, Apache Kafka, Kubernetes components, and the selected database.

The exact minimum hardware requirements have not yet been established and will depend on the available virtual machine and the final deployment configuration.

The following specifications remain to be determined:
- Number of available virtual CPUs.
- Amount of system memory (RAM).
- Available disk storage.
- Network configuration and external connectivity.
- Persistent storage requirements.
The final resource requirements will be documented and validated during deployment testing.
2.4.4 Network and Communication Environment
The system requires network connectivity between its microservices and Apache Kafka.

External clients will communicate with the Order Service through an HTTP-based API.

Internal business communication between microservices will use Apache Kafka topics rather than direct synchronous service-to-service calls.

Within Kubernetes, services will use the cluster's networking and service-discovery mechanisms to communicate with the required infrastructure components.

The exact external access method and network security configuration will be determined during deployment design.
2.4.5 Environment Compatibility
The application will be packaged into container images to provide consistent runtime dependencies across development and deployment environments.

Environment-specific configuration, such as Kafka broker addresses, database connection settings, and service ports, will be supplied through external configuration rather than embedded directly in the application code.

The system must support deployment on the selected Kubernetes environment without requiring modifications to its core business logic.

## 2.5 Design and Implementation Constraints

The design and implementation of the Event-Driven Order Processing System are subject to several architectural, technological, and infrastructure constraints. These constraints ensure that the system satisfies the objectives of the Advanced Software Architecture Patterns course while remaining compatible with the selected reference implementation.

### 2.5.1 Architectural Constraints

The system must follow a combination of **Microservices Architecture** and **Event-Driven Architecture (EDA)**.

The following architectural constraints apply:

- The application must consist of independently deployable microservices with clearly defined responsibilities.
- The main business functions must be separated into Order, Payment, Inventory, and Notification services.
- Communication between microservices for order-processing operations must be asynchronous and event-driven.
- Apache Kafka must be used as the event broker for exchanging events between services.
- Microservices must not rely on direct synchronous service-to-service calls to execute the main order-processing workflow.
- Each microservice should remain independently maintainable and deployable without requiring modifications to unrelated services.
- The system must account for eventual consistency resulting from asynchronous processing.

### 2.5.2 Technology Constraints

The project is based on an existing Java and Spring Boot implementation.

The following technologies are selected for the application:

| Technology | Constraint |
|------------|------------|
| Java 21 | Primary programming language |
| Spring Boot | Framework for implementing the microservices |
| Apache Kafka | Required asynchronous communication infrastructure |
| Apache Maven | Application build and dependency management |
| Docker | Building and packaging container images |
| Kubernetes | Deployment and orchestration of application containers |
| Database System | Selected database must support the application's data storage requirements |

The database technology and persistence strategy will be finalized during implementation planning.

### 2.5.3 Communication Constraints

External clients will interact with the Order Service using an HTTP-based API with JSON-formatted requests and responses.

Internal communication between the microservices must use Apache Kafka events.

Each microservice must publish and consume events according to defined message structures to ensure compatibility between producers and consumers.

Communication failures must not require the entire system to stop processing unrelated operations.

The system must provide mechanisms for handling temporary service unavailability and unsuccessful event processing.

### 2.5.4 Deployment Constraints

The completed application must be deployable on a virtual machine using Kubernetes.

The following deployment constraints apply:

- Each application microservice must be packaged as a container image.
- Kubernetes configuration must define the workloads and networking resources required to run the application.
- Apache Kafka must be accessible to the microservices within the deployment environment.
- Application configuration must support differences between development and deployment environments.
- Deployment must operate within the available CPU, memory, storage, and networking resources of the virtual machine.
- The deployment must support independent management and restarting of individual microservices.

The exact Kubernetes distribution and virtual machine specifications will be confirmed according to the course requirements.

### 2.5.5 Data Management Constraints

The system must maintain the information required to create and process orders.

The following constraints apply:

- Order information must be stored in a database.
- Each order must have a unique identifier.
- Events exchanged between microservices must contain sufficient information to identify the associated order.
- The data management approach must account for asynchronous updates and eventual consistency.
- The system must prevent repeated processing of the same event from causing unintended duplicate business operations.
- Persistence requirements and database ownership will be defined before final implementation.

### 2.5.6 Security Constraints

The application must apply basic security practices appropriate for an educational distributed system.

The following constraints apply:

- Incoming API requests must be validated.
- Sensitive configuration values must not be hard-coded into application source code.
- Communication interfaces must be restricted to the components and users that require access.
- Internal infrastructure services should not be unnecessarily exposed to external networks.
- The system must not process real financial transactions or store actual payment-card information.

### 2.5.7 Project Scope and Resource Constraints

The project is developed within the time and resource limitations of an academic course.

Consequently:

- The implementation will focus on demonstrating the selected architectural patterns and their quality attributes.
- Payment processing and customer notifications will be simulated.
- The system will not require integration with commercial banking, payment, or delivery services.
- The application will be designed for demonstration and evaluation rather than production-scale commercial operation.
- Additional functionality may be limited to maintain a manageable implementation and deployment scope.

## 2.6 User Documentation

The Event-Driven Order Processing System will be accompanied by technical documentation to support its installation, configuration, operation, testing, and maintenance.

Since the application is primarily an API-based educational prototype, the documentation will focus on developers, testers, and system administrators rather than end users of a graphical interface.

The following documentation will be provided:

### 1. Project README

A `README.md` file will be maintained in the project's GitHub repository.

It will provide:

- An overview of the project and its objectives.
- A description of the system architecture and main microservices.
- The technologies and dependencies used.
- Instructions for obtaining and building the source code.
- Instructions for running the application locally.
- Example requests for testing the order-processing workflow.
- References to additional technical documentation.

### 2. API Documentation

API documentation will describe the HTTP endpoints exposed by the application.

It will include:

- Available API endpoints and their purposes.
- Supported HTTP methods.
- Request and response formats.
- Required and optional parameters.
- Example JSON requests and responses.
- Expected error responses.

The documentation will enable API clients and testers to interact with the system without requiring knowledge of its internal implementation.

### 3. Architecture Documentation

Architecture documentation will describe the organization and communication of the system's components.

It will include:

- An overview of the Microservices and Event-Driven architectural patterns.
- The responsibilities of each microservice.
- Architecture and communication diagrams.
- Apache Kafka topics and event flows.
- Important architectural decisions and their rationale.
- Architectural trade-offs and selected quality attributes.

### 4. Deployment and Configuration Guide

A deployment guide will describe how to deploy and operate the application using Docker and Kubernetes on a virtual machine.

It will include:

- Deployment prerequisites.
- Container image build instructions.
- Required application configuration.
- Kubernetes deployment instructions.
- Service networking and connectivity requirements.
- Procedures for starting, stopping, and restarting services.
- Instructions for accessing application logs.
- Basic troubleshooting procedures.

### 5. Testing and Verification Documentation

Testing documentation will describe the procedures used to verify the system's functional and non-functional requirements.

It will include:

- Test scenarios for the order-processing workflow.
- Expected results for successful and unsuccessful operations.
- Procedures for testing communication between microservices.
- Service failure and recovery test scenarios.
- Procedures for evaluating selected architectural quality attributes.
- A summary of test results and observed system behavior.

### Documentation Formats

The primary documentation format will be **Markdown (`.md`)**, stored alongside the application's source code in the GitHub repository.

The final project report may additionally be provided in PDF format for academic submission.

A separate graphical user manual is not required because the project does not include a dedicated graphical user interface.

## 2.7 Assumptions and Dependencies

The design, implementation, and deployment of the Event-Driven Order Processing System rely on several assumptions and external dependencies. Changes to these conditions may affect the system's functionality, performance, or deployment feasibility.

### 2.7.1 Assumptions

The following assumptions are considered during the development of the system:

**A-01: Virtual Machine Availability**

It is assumed that a virtual machine will be available for hosting the Kubernetes environment and the application components.

The virtual machine is expected to provide sufficient CPU, memory, storage, and networking resources to run the required services.

**A-02: Development Environment**

It is assumed that the development team has access to the tools required to build, test, and containerize the application, including Java, Maven, Docker, and Git.

**A-03: Apache Kafka Availability**

It is assumed that Apache Kafka will be operational during normal system execution and accessible to all microservices that publish or consume events.

Temporary broker failures may affect event delivery and processing until communication is restored.

**A-04: Simulated External Operations**

Payment processing and customer notifications are assumed to be simulated.

The system will not depend on real banking systems, payment gateways, email providers, or SMS services.

**A-05: Reference Implementation**

It is assumed that the existing open-source Event-Driven Order System provides a suitable foundation for the project.

The development team may modify or extend the reference implementation to satisfy the requirements defined in this specification.

**A-06: Educational Workload**

The system is expected to operate under a limited workload appropriate for an academic demonstration.

Production-scale traffic and large numbers of simultaneous users are outside the intended scope.

**A-07: Containerized Deployment**

It is assumed that the selected virtual machine and Kubernetes environment will support the container images and infrastructure components required by the application.

### 2.7.2 External Dependencies

The system depends on the following technologies and infrastructure:

| ID | Dependency | Description |
|----|------------|-------------|
| D-01 | Java Runtime | Required to execute the Spring Boot microservices. |
| D-02 | Spring Boot | Provides the application framework for implementing the microservices. |
| D-03 | Apache Kafka | Provides asynchronous event communication between services. |
| D-04 | Database System | Stores order information and any additional persistent application state. |
| D-05 | Docker / OCI Container Images | Provides the application packaging format used for deployment. |
| D-06 | Kubernetes | Manages application workloads, networking, and container lifecycle operations. |
| D-07 | Virtual Machine | Provides the computing infrastructure for the deployed system. |
| D-08 | Network Connectivity | Enables communication between clients, application services, Kafka, and databases. |

### 2.7.3 Dependency-Related Risks

Failures or limitations affecting external dependencies may influence the behavior of the system.

The following risks have been identified:

- **Kafka unavailability:** Microservices may be unable to publish or consume events until broker connectivity is restored.
- **Database unavailability:** Services may be unable to create, retrieve, or update information that requires persistent storage.
- **Insufficient VM resources:** Limited CPU, memory, or storage may prevent components from starting correctly or reduce processing performance.
- **Kubernetes deployment failures:** Incorrect configuration or unavailable cluster resources may prevent application workloads from running.
- **Network failures:** Communication interruptions may delay event delivery or prevent clients from accessing the application's HTTP API.
- **Reference implementation limitations:** Existing functionality may require modifications to support persistent storage, error handling, and Kubernetes deployment.

These risks will be considered during implementation, integration testing, and deployment planning.

### 2.7.4 Unresolved Dependencies

The following decisions remain to be determined:

- Exact virtual machine hardware specifications.
- Final Kubernetes distribution and configuration.
- Database technology and persistence strategy.
- Kafka deployment configuration and persistence requirements.
- External API access and networking configuration.
- Detailed failure recovery and event retry mechanisms.

These unresolved items will be tracked in **Appendix C: To Be Determined List** and updated as implementation decisions are finalized.

# 3. External Interface Requirements

## 3.1 User Interfaces

The Event-Driven Order Processing System is designed primarily as an API-based application and does not require a dedicated graphical user interface (GUI).

Users interact with the application through an HTTP API exposed by the Order Service. The interface allows clients to submit orders and, in the proposed extended implementation, retrieve information about their processing status.

The system's API can be accessed using HTTP clients such as Postman, cURL, or other applications capable of sending HTTP requests.

### 3.1.1 Order Submission Interface

The Order Service provides an HTTP endpoint through which clients can submit new orders.

The interface accepts order information in JSON format, including:

- Product identifier.
- Order quantity.
- Product price.

When a valid order is accepted, the system returns an acknowledgement containing information that allows the client to identify the submitted order.

The acknowledgement does not require all downstream operations, such as payment processing, inventory updates, and customer notification, to be completed.

### 3.1.2 Order Status Interface

The proposed system will provide an HTTP interface for retrieving the processing status of an existing order.

Clients will be able to request information about an order using its unique identifier.

The returned information will indicate the latest known processing state, allowing clients to distinguish between orders that have been accepted, are being processed, or have completed processing.

The exact endpoint structure, response format, and supported status values will be defined in the functional and interface requirements.

### 3.1.3 Error Handling and Feedback

The HTTP interface must provide understandable responses when a request cannot be processed.

The system must:

- Reject requests containing invalid or missing required information.
- Return appropriate HTTP status codes.
- Provide error descriptions in a consistent JSON format.
- Distinguish between successful order acceptance and unsuccessful requests.
- Avoid exposing sensitive internal implementation details through error messages.

### 3.1.4 Administrative and Development Interfaces

Developers and system operators will interact with the application using development, testing, and infrastructure-management tools.

These interfaces may include:

- HTTP API testing tools for submitting requests and inspecting responses.
- Application logs for observing order-processing events.
- Docker tools for building and inspecting container images.
- Kubernetes command-line tools for monitoring and managing deployed services.

These tools are external utilities and are not custom graphical interfaces developed as part of the application.

### 3.1.5 Graphical User Interface

A dedicated graphical user interface is **Not Applicable (N/A)** to the current project scope.

The system does not require customer registration screens, shopping-cart interfaces, administrative dashboards, or other graphical e-commerce components.

Such interfaces may be developed in future versions without changing the main event-driven processing architecture.

## 3.2 Hardware Interfaces

&lt;Describe the logical and physical characteristics of each interface between the software product and the hardware components of the system. This may include the supported device types, the nature of the data and control interactions between the software and the hardware, and communication protocols to be used.&gt;

## 3.3 Software Interfaces

&lt;Describe the connections between this product and other specific software components (name and version), including databases, operating systems, tools, libraries, and integrated commercial components. Identify the data items or messages coming into the system and going out and describe the purpose of each. Describe the services needed and the nature of communications. Refer to documents that describe detailed application programming interface protocols. Identify data that will be shared across software components. If the data sharing mechanism must be implemented in a specific way (for example, use of a global data area in a multitasking operating system), specify this as an implementation constraint.&gt;

## 3.4 Communications Interfaces

&lt;Describe the requirements associated with any communications functions required by this product, including e-mail, web browser, network server communications protocols, electronic forms, and so on. Define any pertinent message formatting. Identify any communication standards that will be used, such as FTP or HTTP. Specify any communication security or encryption issues, data transfer rates, and synchronization mechanisms.&gt;

# 4. System Features

&lt;This template illustrates organizing the functional requirements for the product by system features, the major services provided by the product. You may prefer to organize this section by use case, mode of operation, user class, object class, functional hierarchy, or combinations of these, whatever makes the most logical sense for your product.&gt;

## 4.1 System Feature 1

&lt;Don’t really say “System Feature 1.” State the feature name in just a few words.&gt;

### 4.1.1 Description and Priority

&lt;Provide a short description of the feature and indicate whether it is of High, Medium, or Low priority. You could also include specific priority component ratings, such as benefit, penalty, cost, and risk (each rated on a relative scale from a low of 1 to a high of 9).&gt;

### 4.1.2 Stimulus/Response Sequences

&lt;List the sequences of user actions and system responses that stimulate the behavior defined for this feature. These will correspond to the dialog elements associated with use cases.&gt;

### 4.1.3 Functional Requirements

&lt;Itemize the detailed functional requirements associated with this feature. These are the software capabilities that must be present in order for the user to carry out the services provided by the feature, or to execute the use case. Include how the product should respond to anticipated error conditions or invalid inputs. Requirements should be concise, complete, unambiguous, verifiable, and necessary. Use “TBD” as a placeholder to indicate when necessary information is not yet available.&gt;

&lt;Each requirement should be uniquely identified with a sequence number or a meaningful tag of some kind.&gt;

REQ-1:

REQ-2:

## 4.2 System Feature 2 (and so on)

# 5. Other Nonfunctional Requirements

## 5.1 Performance Requirements

&lt;If there are performance requirements for the product under various circumstances, state them here and explain their rationale, to help the developers understand the intent and make suitable design choices. Specify the timing relationships for real time systems. Make such requirements as specific as possible. You may need to state performance requirements for individual functional requirements or features.&gt;

## 5.2 Safety Requirements

&lt;Specify those requirements that are concerned with possible loss, damage, or harm that could result from the use of the product. Define any safeguards or actions that must be taken, as well as actions that must be prevented. Refer to any external policies or regulations that state safety issues that affect the product’s design or use. Define any safety certifications that must be satisfied.&gt;

## 5.3 Security Requirements

&lt;Specify any requirements regarding security or privacy issues surrounding use of the product or protection of the data used or created by the product. Define any user identity authentication requirements. Refer to any external policies or regulations containing security issues that affect the product. Define any security or privacy certifications that must be satisfied.&gt;

## 5.4 Software Quality Attributes

&lt;Specify any additional quality characteristics for the product that will be important to either the customers or the developers. Some to consider are: adaptability, availability, correctness, flexibility, interoperability, maintainability, portability, reliability, reusability, robustness, testability, and usability. Write these to be specific, quantitative, and verifiable when possible. At the least, clarify the relative preferences for various attributes, such as ease of use over ease of learning.&gt;

## 5.5 Business Rules

&lt;List any operating principles about the product, such as which individuals or roles can perform which functions under specific circumstances. These are not functional requirements in themselves, but they may imply certain functional requirements to enforce the rules.&gt;

# 6. Other Requirements

&lt;Define any other requirements not covered elsewhere in the SRS. This might include database requirements, internationalization requirements, legal requirements, reuse objectives for the project, and so on. Add any new sections that are pertinent to the project.&gt;

# Appendix A: Glossary

&lt;Define all the terms necessary to properly interpret the SRS, including acronyms and abbreviations. You may wish to build a separate glossary that spans multiple projects or the entire organization, and just include terms specific to a single project in each SRS.&gt;

# Appendix B: Analysis Models

&lt;Optionally, include any pertinent analysis models, such as data flow diagrams, class diagrams, state-transition diagrams, or entity-relationship diagrams.&gt;

# Appendix C: To Be Determined List

&lt;Collect a numbered list of the TBD (to be determined) references that remain in the SRS so they can be tracked to closure.&gt;
