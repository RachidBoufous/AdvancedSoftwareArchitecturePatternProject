# ADR-0003: Deploy on Kubernetes on a Virtual Machine

- **Status:** Accepted
- **Date:** 2026-10-26
- **Deciders:** Elfadel Mohamed, Rachid Boufous, Chedy
- **Related SRS items:** 2.4.2, 2.5.4, 4.7, FR-84 – FR-100, TBD-13, TBD-14

## Context

The course requires the system to run in containers on Kubernetes hosted on a virtual machine. The deployment must also demonstrate the architecture's promises: independent deployment, automatic restart of failed services, and independent scaling (FR-85, FR-92, FR-95).

## Considered Options

1. **Docker Compose** on the VM
2. **Single-node Kubernetes** on the VM (e.g., k3s, minikube, or kubeadm)
3. **Managed cloud Kubernetes** (e.g., EKS, GKE, AKS)

## Decision

We will package each microservice as a Docker image and deploy everything on a single-node Kubernetes cluster on the VM:

- One `Deployment` and one `Service` per microservice.
- Kafka and the database run as `StatefulSet`s with persistent volumes.
- Configuration comes from `ConfigMap`s; credentials come from `Secret`s (NFR-28 – NFR-30).
- Readiness and liveness probes use Spring Boot Actuator health endpoints (FR-93).

Docker Compose remains the local development setup but does not meet the course requirement. Managed cloud Kubernetes adds cost and is outside the VM-based scope.

## Consequences

**Positive**

- Meets the mandatory deployment requirement.
- Kubernetes restarts crashed pods and allows changing replica counts per service.
- Manifests in the repository make the deployment reproducible (FR-96).

**Negative**

- Kubernetes, Kafka, and the database compete for the VM's limited CPU and memory (FR-97).
- A single node is a single point of failure; this demonstrates service-level, not node-level, fault tolerance.
- The team must learn and maintain Kubernetes manifests.

**Follow-up**

- Choose the Kubernetes distribution (TBD-13) once the VM specifications are known (TBD-14).
- Choose how the Order Service is exposed: NodePort or Ingress (TBD-15).
