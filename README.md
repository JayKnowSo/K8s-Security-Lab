# K8s Security Lab

![CI](https://github.com/JayKnowSo/K8s-Security-Lab/actions/workflows/ci.yaml/badge.svg)

A hardened Kubernetes environment demonstrating defense-in-depth security controls across the full workload lifecycle — from admission through runtime.

## Security Architecture

| Control | Implementation | Threat Mitigated |
|---|---|---|
| Pod Security Admission | restricted enforcement | Privileged containers, root execution, capability abuse |
| RBAC | Scoped ServiceAccount + Role + RoleBinding | Lateral movement via over-privileged identities |
| NetworkPolicy | Default deny-all, explicit allow | Unrestricted pod-to-pod communication |
| Falco | eBPF kernel-level runtime detection | Container escape, shell spawns (MITRE T1059) |
| Trivy Operator | Continuous CVE scanning | Known vulnerabilities in running workloads |

## Cluster Structure

    kind-security-lab
    production    # PSA restricted — all workloads enforced
    monitoring    # Falco runtime detection

## Stack

kind v0.22.0 · kubectl v1.34.1 · Falco · Trivy Operator

## ADRs

- [ADR-001 — Local Cluster Tooling](docs/adr/ADR-001-local-cluster-tooling.md)
