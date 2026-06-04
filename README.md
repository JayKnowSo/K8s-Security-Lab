# K8s Security Lab

![CI](https://github.com/JayKnowSo/K8s-Security-Lab/actions/workflows/ci.yaml/badge.svg)

Runtime security for Kubernetes — syscall-level threat detection, hardened admission controls, continuous vulnerability scanning, and a supply-chain-hardened CI pipeline.

Two of the custom Falco rules in this lab were contributed upstream to the Falco OSS project:
- **[PR #363](https://github.com/falcosecurity/falco/pull/363)** — Container shell spawn detection (MITRE T1059)
- **[PR #367](https://github.com/falcosecurity/falco/pull/367)** — Impair Defenses detection: iptables flush + security daemon stop (MITRE T1562.001)

---

## Live Detection

Falco firing on a real shell spawn inside a container (MITRE T1059):

```json
{
  "rule": "Container Shell Spawn",
  "priority": "Warning",
  "output": "Shell spawned in container (user=root shell=sh cmdline=sh -c pg_isready)",
  "tags": ["T1059", "container", "mitre_execution", "shell"],
  "time": "2026-06-04T06:17:27Z"
}
```

---

## Threat Coverage

| MITRE Technique | ID | Control |
|---|---|---|
| Command and Scripting Interpreter | T1059 | Falco custom rule — shell spawn detection |
| Impair Defenses | T1562.001 | Falco custom rule — iptables flush + security daemon stop |
| Escape to Host | T1611 | Falco modern_ebpf + PSA restricted |
| Lateral Movement via Services | T1021 | NetworkPolicy default-deny |
| Privilege Escalation | T1068 | PSA restricted — drop: ALL capabilities |
| Valid Accounts / Over-Privilege | T1078 | RBAC scoped ServiceAccount |
| Exploit Public-Facing Application | T1190 | Trivy Operator continuous CVE scanning |
| Compromise Software Dependencies | T1195.001 | SHA-pinned CI actions + binary downloads |

---

## Security Controls

| Layer | Control | Implementation |
|---|---|---|
| Admission | Pod Security Admission | `restricted` enforcement — blocks root, hostPath, privilege escalation |
| Identity | RBAC | Scoped ServiceAccount + Role + RoleBinding per namespace |
| Network | NetworkPolicy | Default deny-all; traffic permitted by explicit policy only |
| Runtime | Falco (Helm) | `modern_ebpf` driver — custom rules for T1059 + T1562.001 |
| Vulnerability | Trivy Operator | Continuous CVE scanning on running workloads |
| CI — Schema | kubeconform | Manifest validation against upstream Kubernetes API schemas |
| CI — Misconfig | Trivy | CRITICAL/HIGH misconfiguration gate on every push |
| CI — Supply Chain | SHA pinning | All actions and binaries pinned to immutable SHAs — direct response to [March 2026 trivy-action compromise](https://attack.mitre.org/techniques/T1195/001/) |

---

## Architecture

```
kind-security-lab
├── production      # PSA restricted — workload admission enforced at API server
└── falco-system    # Falco modern_ebpf — syscall-level runtime detection
```

---

## Repository

```
.
├── compliant-pod.yaml              # Reference pod — non-root, read-only fs, drop ALL caps
├── namespaces.yaml                 # production + falco-system namespace definitions
├── networkpolicy.yaml              # Default deny-all + explicit allow
├── rbac.yaml                       # Least-privilege ServiceAccount, Role, RoleBinding
├── falco/
│   ├── falco-values.yaml           # Hardened Helm values — modern_ebpf, resource limits, custom rules
│   └── rules/
│       └── container-escape.yaml   # Standalone T1059 rule
├── trivy/
│   └── trivy-operator.yaml         # Trivy Operator — trivy-system namespace, least-privilege ClusterRole
├── docs/
│   └── adr/                        # 6 Architecture Decision Records
└── .github/
    └── workflows/
        └── ci.yaml                 # kubeconform + Trivy misconfig gate, SHA-pinned
```

---

## Deployment

**Prerequisites:** kind v0.22.0+, kubectl, helm ≥ 3.x, kernel ≥ 5.8

```bash
# Apply baseline manifests
kubectl apply -f namespaces.yaml
kubectl apply -f rbac.yaml
kubectl apply -f networkpolicy.yaml
kubectl apply -f compliant-pod.yaml

# Deploy Falco
helm repo add falcosecurity https://falcosecurity.github.io/charts && helm repo update
helm install falco falcosecurity/falco \
  --namespace falco-system \
  --create-namespace \
  --values falco/falco-values.yaml
kubectl rollout status daemonset/falco -n falco-system

# Deploy Trivy Operator
kubectl apply -f trivy/trivy-operator.yaml
```

### Test T1059 Detection

```bash
# Terminal 1 — watch for alerts
kubectl logs -l app.kubernetes.io/name=falco -n falco-system -f

# Terminal 2 — trigger detection
kubectl run test-shell --image=alpine --restart=Never -- sleep 3600
kubectl exec -it test-shell -- sh
# Falco fires: "Container Shell Spawn" WARNING, tags: [T1059]
```

---

## ADR Index

| ADR | Decision |
|---|---|
| [ADR-001](docs/adr/ADR-001-local-cluster-tooling.md) | Local cluster tooling: kind over minikube/k3s |
| [ADR-002](docs/adr/ADR-002-falco-ebpf-engine.md) | Falco engine: modern_ebpf over kernel module |
| [ADR-003](docs/adr/ADR-003-pod-security-admission.md) | Pod Security Admission: restricted over baseline |
| [ADR-004](docs/adr/ADR-004-networkpolicy-default-deny.md) | NetworkPolicy: default-deny with explicit allow |
| [ADR-005](docs/adr/ADR-005-ci-action-sha-pinning.md) | SHA pinning for all CI actions and binary downloads |
| [ADR-006](docs/adr/ADR-006-falco-helm-deployment.md) | Deploy Falco via Helm chart over raw manifests |

---

## Stack

kind v0.22.0 · Falco (modern_ebpf) · Trivy Operator · kubeconform · kubectl v1.34.1

---

## Author

Jemel Padilla — [GitHub](https://github.com/JayKnowso)
