# ADR-001: Local Kubernetes Cluster Tooling

## Status
Accepted

## Context
Security controls must be validated against a conformant Kubernetes API before targeting production EKS. Deployment environment is WSL2 — no hypervisor, no cloud spend.

## Decision
kind (Kubernetes IN Docker) v0.22.0

## Rationale
- Runs inside existing Docker daemon — zero additional dependencies
- Passes Kubernetes conformance tests — behavior is identical to EKS/GKE/AKS
- Native multi-node support — required for NetworkPolicy and inter-pod traffic testing
- CI/CD compatible — spins up in GitHub Actions runners without elevated privileges

## Alternatives Rejected
- **minikube** — VM-based by default, hypervisor dependency conflicts with WSL2
- **k3s** — API deviations from upstream Kubernetes invalidate conformance assumptions

## Consequences
All manifests — PSA, RBAC, NetworkPolicy — apply to production clusters without modification.
