# ADR-003 — Pod Security Admission: restricted Enforcement

## Status
Accepted

## Context
Kubernetes Pod Security Admission (PSA) replaced PodSecurityPolicy in v1.25. Three policy levels exist: `privileged`, `baseline`, and `restricted`. The production namespace requires a policy level that prevents common container escape vectors without requiring a separate admission webhook.

## Decision
Enforce the `restricted` PSA level on the `production` namespace.

## Rationale
- **Blocks root execution** — requires `runAsNonRoot: true`, eliminating UID 0 privilege escalation paths
- **Drops all capabilities** — `allowPrivilegeEscalation: false` + `drop: ALL` removes kernel capability abuse vectors (MITRE T1611)
- **Enforces read-only root filesystem** — reduces attacker persistence after initial compromise
- **No webhook dependency** — PSA is built into the API server; zero additional attack surface from admission controllers
- **Production parity** — `restricted` maps directly to CIS Kubernetes Benchmark Level 2 controls

## Alternatives Rejected
- **baseline** — permits hostPath mounts and does not require non-root execution; insufficient for workloads handling sensitive data
- **OPA/Gatekeeper** — adds webhook complexity and a new failure domain; unjustified for a lab demonstrating native controls
- **privileged** — no restrictions; rejected outright

## Consequences
All workloads in the `production` namespace must declare explicit security contexts. Non-compliant pods are blocked at admission. Validated by `compliant-pod.yaml`.
