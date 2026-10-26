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

&lt;List any other documents or Web addresses to which this SRS refers. These may include user interface style guides, contracts, standards, system requirements specifications, use case documents, or a vision and scope document. Provide enough information so that the reader could access a copy of each reference, including title, author, version number, date, and source or location.&gt;

# 2. Overall Description

## 2.1 Product Perspective

&lt;Describe the context and origin of the product being specified in this SRS. For example, state whether this product is a follow-on member of a product family, a replacement for certain existing systems, or a new, self-contained product. If the SRS defines a component of a larger system, relate the requirements of the larger system to the functionality of this software and identify interfaces between the two. A simple diagram that shows the major components of the overall system, subsystem interconnections, and external interfaces can be helpful.&gt;

## 2.2 Product Functions

&lt;Summarize the major functions the product must perform or must let the user perform. Details will be provided in Section 3, so only a high level summary (such as a bullet list) is needed here. Organize the functions to make them understandable to any reader of the SRS. A picture of the major groups of related requirements and how they relate, such as a top level data flow diagram or object class diagram, is often effective.&gt;

## 2.3 User Classes and Characteristics

&lt;Identify the various user classes that you anticipate will use this product. User classes may be differentiated based on frequency of use, subset of product functions used, technical expertise, security or privilege levels, educational level, or experience. Describe the pertinent characteristics of each user class. Certain requirements may pertain only to certain user classes. Distinguish the most important user classes for this product from those who are less important to satisfy.&gt;

## 2.4 Operating Environment

&lt;Describe the environment in which the software will operate, including the hardware platform, operating system and versions, and any other software components or applications with which it must peacefully coexist.&gt;

## 2.5 Design and Implementation Constraints

&lt;Describe any items or issues that will limit the options available to the developers. These might include: corporate or regulatory policies; hardware limitations (timing requirements, memory requirements); interfaces to other applications; specific technologies, tools, and databases to be used; parallel operations; language requirements; communications protocols; security considerations; design conventions or programming standards (for example, if the customer’s organization will be responsible for maintaining the delivered software).&gt;

## 2.6 User Documentation

&lt;List the user documentation components (such as user manuals, on-line help, and tutorials) that will be delivered along with the software. Identify any known user documentation delivery formats or standards.&gt;

## 2.7 Assumptions and Dependencies

&lt;List any assumed factors (as opposed to known facts) that could affect the requirements stated in the SRS. These could include third-party or commercial components that you plan to use, issues around the development or operating environment, or constraints. The project could be affected if these assumptions are incorrect, are not shared, or change. Also identify any dependencies the project has on external factors, such as software components that you intend to reuse from another project, unless they are already documented elsewhere (for example, in the vision and scope document or the project plan).&gt;

# 3. External Interface Requirements

## 3.1 User Interfaces

&lt;Describe the logical characteristics of each interface between the software product and the users. This may include sample screen images, any GUI standards or product family style guides that are to be followed, screen layout constraints, standard buttons and functions (e.g., help) that will appear on every screen, keyboard shortcuts, error message display standards, and so on. Define the software components for which a user interface is needed. Details of the user interface design should be documented in a separate user interface specification.&gt;

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
