# ADR-006: Falco Runtime Security Deployment via Helm Chart

## Status
Accepted

## Context
Deploying Falco to Kubernetes requires coordinating multiple interdependent
resources: a DaemonSet, ServiceAccount, ClusterRole, ClusterRoleBinding,
driver init containers, and ConfigMaps for detection rules.

Three deployment approaches were evaluated:

- **Raw YAML manifests** — manually authored Kubernetes resources, the same
  pattern used for Trivy Operator deployment in this lab
- **Official falcosecurity Helm chart** — versioned, templated deployment
  with values-driven configuration (current stable: chart 8.0.5)
- **Falco Operator** — CRD-based FalcoRule custom resources managed by a
  reconciliation controller

The Falco project now documents the Operator as the recommended Kubernetes
deployment method, citing its declarative, Kubernetes-native experience and
multi-instance management capabilities. The Helm chart remains fully supported.

Helm is the correct choice for this lab for three reasons. First, `helm template`
renders every Kubernetes resource the chart will deploy — DaemonSet spec,
RBAC bindings, ConfigMaps — as inspectable YAML before any resource
touches the cluster. A security lab must understand exactly what is running;
the Operator's CRD reconciliation loop inserts an abstraction layer between
declared configuration and actual running state that defeats this requirement.
Second, the Operator adds a controller pod, CRD installation, and independent
operator RBAC on top of Falco itself — each an additional component requiring
its own hardening. This lab manages a single Falco instance with one custom
rule set, which is precisely the use case where the Operator's advantages
do not apply. Third, Helm chart OCI artifact signing extends the supply chain
integrity posture established in ADR-005 directly to the runtime security layer.

Raw manifests were ruled out because Falco's resource surface is significantly
larger than Trivy Operator's single ClusterRole deployment. Driver selection
changes which init containers run, and custom rules must load in the correct
ConfigMap order. A misaligned mount path breaks rule loading silently — no
runtime error fires, threats go undetected — which is unacceptable in a
detection-focused lab.

## Decision
Deploy Falco via the official `falcosecurity/falco` Helm chart, pinned to
chart version `8.0.5`, using a committed `falco/values.yaml`:

- Chart version pinned to `8.0.5` — same immutability principle as ADR-005;
  `--version 8.0.5` passed explicitly to `helm upgrade --install` so the
  cluster never silently pulls a newer chart on re-deploy
- Modern eBPF probe selected as the driver — the default since Falco 0.38.0,
  embedded in the Falco binary via CO-RE (Compile Once - Run Everywhere);
  no kernel module loaded, no kernel headers required on cluster nodes,
  no separate probe installation step
- Custom detection rules (container-escape.yaml, MITRE T1059) injected via
  `customRules` block in values.yaml — rules version-controlled alongside
  chart version in a single auditable file
- Deployment command:
  `helm upgrade --install falco falcosecurity/falco \`
  `  --version 8.0.5 --namespace falco --create-namespace -f falco/values.yaml`
  manages all resources atomically; `helm rollback falco` restores the
  previous working detection state if a rule update breaks runtime behavior

## Consequences

**Positive:**
- `helm template` renders the full resource manifest before deployment —
  every ClusterRole, DaemonSet spec, and ConfigMap mount is inspectable
  and auditable before reaching the cluster
- All Falco resources managed atomically — no risk of ConfigMap/DaemonSet
  drift causing silent detection gaps where threats fire no alert
- `helm rollback` restores the previous working detection configuration
  immediately if a rule update breaks runtime behavior
- Modern eBPF probe eliminates kernel module loading, removing the need for
  kernel header packages on cluster nodes and avoiding the kernel panic risk
  inherent in a faulty kernel module
- Custom rules committed in values.yaml are diff-able and permanently tied
  to a specific chart version — full audit trail across every rule change
- Chart OCI artifact signing extends supply chain integrity from ADR-005's
  SHA pinning posture to the runtime security tooling itself

**Negative:**
- Helm 3.x is a required deployment dependency — adds a tool with its own
  version and upgrade lifecycle to maintain in the lab environment
- Pinning to `8.0.5` requires deliberate audits before upgrading — the same
  comment-drift risk identified in ADR-005 applies here; a stale version
  comment in values.yaml misleads future engineers reviewing the deployment
- Choosing Helm over the now-recommended Operator means forgoing future
  Operator ecosystem improvements (multi-instance management, native
  Kubernetes reconciliation) if this lab evolves beyond its current scope

## References
- falcosecurity/charts (official Helm chart): https://github.com/falcosecurity/charts/tree/master/charts/falco
- Falco Helm deployment on Kubernetes: https://falco.org/docs/setup/kubernetes/
- Falco kernel event sources and Modern eBPF probe: https://falco.org/docs/concepts/event-sources/kernel/
- MITRE ATT&CK T1059 (Command and Scripting Interpreter): https://attack.mitre.org/techniques/T1059/
