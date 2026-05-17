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
K8s Security Lab
Show Image
Defense-in-depth Kubernetes security controls across the full workload lifecycle — admission, network, identity, and runtime.

Threat Coverage
MITRE TechniqueIDControlCommand and Scripting InterpreterT1059Falco custom rule — shell spawn detectionEscape to HostT1611Falco custom rule — sensitive path write + PSA restrictedLateral Movement via ServicesT1021NetworkPolicy default-denyPrivilege EscalationT1068PSA restricted — drop: ALL capabilitiesValid Accounts / Over-PrivilegeT1078RBAC scoped ServiceAccount

Security Controls
LayerControlImplementationAdmissionPod Security Admissionrestricted enforcement — blocks root, hostPath, privilege escalationIdentityRBACScoped ServiceAccount + Role + RoleBinding per namespaceNetworkNetworkPolicyDefault deny-all; traffic permitted by explicit policy onlyRuntimeFalco 0.43.0Modern BPF (CO-RE) — custom rules for T1059 + T1611VulnerabilityTrivy OperatorContinuous CVE scanning on running workloadsCI — SchemakubeconformManifest validation against upstream Kubernetes API schemasCI — MisconfigTrivyCRITICAL/HIGH misconfiguration gate on every push

Architecture
kind-security-lab
├── production    # PSA restricted — workload admission enforced at API server
└── monitoring    # Falco Modern BPF — syscall-level runtime detection

Repository
.
├── compliant-pod.yaml
├── namespaces.yaml
├── networkpolicy.yaml
├── rbac.yaml
├── falco/
│   └── rules/
│       └── custom-rules.yaml
├── scripts/
│   └── get_helm.sh
├── docs/
│   └── adr/
│       ├── ADR-001-local-cluster-tooling.md
│       ├── ADR-002-falco-ebpf-engine.md
│       ├── ADR-003-pod-security-admission.md
│       └── ADR-004-networkpolicy-default-deny.md
└── .github/
    └── workflows/
        └── ci.yaml

Stack
kind v0.22.0 · Falco 0.43.0 (Modern BPF / CO-RE) · Trivy Operator · kubeconform · kubectl v1.34.1

Decisions

ADR-001 — Local Cluster Tooling: kind over minikube/k3s
ADR-002 — Falco Engine: Modern BPF over kernel module
ADR-003 — Pod Security Admission: restricted over baseline
ADR-004 — NetworkPolicy: default-deny with explicit allow