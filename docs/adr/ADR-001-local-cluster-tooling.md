# ADR-001: Local Kubernetes Cluster Tooling

## Status
Accepted

## Context
A local Kubernetes cluster is required for development and security testing without incurring cloud infrastructure costs. Options evaluated: kind, minikube, k3s.

## Decision
Selected kind (Kubernetes IN Docker) as the local cluster tool.

## Rationale
- Runs entirely inside Docker — no VM overhead, no hypervisor dependency
- Consistent behavior with production Kubernetes clusters
- Industry standard for CI/CD pipeline testing
- Zero cost — no cloud spend required during development
- Native multi-node cluster support for future network policy testing

## Alternatives Rejected
- **minikube** — requires VM or Docker driver, heavier resource footprint
- **k3s** — lightweight but diverges from standard Kubernetes API behavior

## Consequences
All security controls built here are portable to production EKS, GKE, or AKS clusters without modification.
