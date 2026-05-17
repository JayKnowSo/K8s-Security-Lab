# ADR-004 — NetworkPolicy: Default-Deny with Explicit Allow

## Status
Accepted

## Context
By default, Kubernetes allows unrestricted pod-to-pod communication across all namespaces. Without NetworkPolicy, a compromised pod can reach any other pod in the cluster — enabling lateral movement (MITRE T1021). The lab requires a network segmentation model that reflects production zero-trust principles.

## Decision
Apply a default-deny-all NetworkPolicy to the `production` namespace, with explicit allow rules for required traffic paths only.

## Rationale
- **Eliminates implicit trust** — pods cannot communicate unless explicitly permitted; mirrors zero-trust network architecture
- **Limits blast radius** — a compromised workload cannot reach the monitoring namespace or other services without an explicit policy
- **Audit-friendly** — all permitted traffic paths are documented in `networkpolicy.yaml`; implicit allows leave no audit trail

## Alternatives Rejected
- **Allow-all (default)** — unacceptable; permits unrestricted lateral movement
- **Namespace-level isolation only** — insufficient; does not restrict pod-to-pod traffic within the same namespace
- **Service mesh (Istio/Linkerd)** — adds operational complexity out of scope for this control layer

## Consequences
Every new service requires an explicit NetworkPolicy allow rule before it can communicate. New workload onboarding must include a NetworkPolicy review.
