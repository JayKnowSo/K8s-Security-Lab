# ADR-002: Falco Runtime Detection Engine Selection

## Status
Accepted

## Context
Falco requires a kernel instrumentation engine to capture syscall events.
Three options exist: kernel module (legacy), eBPF probe (transitional),
and Modern BPF (current standard). Deployment target is WSL2/kind —
a constrained environment without guaranteed kernel headers.

## Decision
Selected Modern BPF engine via Falco 0.43.0+ with falcoctl artifact sidecar.

## Rationale
- Modern BPF uses BTF metadata embedded in the kernel — no kernel headers required
- CO-RE (Compile Once Run Everywhere) ensures portability across kernel versions
- falcoctl sidecar enables real-time rule updates without pod restarts
- Legacy kernel module is deprecated and unsupported in modern Falco releases
- eBPF transitional probe still requires kernel headers — incompatible with WSL2

## Proof Points
- `syscall source opened` confirmed in pod logs
- `schema validation: ok` confirmed on rule load
- 2/2 containers running in monitoring namespace after 4+ hours

## Alternatives Rejected
- **Kernel module** — deprecated, requires kernel headers, incompatible with WSL2
- **eBPF transitional probe** — requires kernel headers, not CO-RE compatible

## Consequences
- Falco rules are portable across any BTF-enabled kernel without recompilation
- falcoctl sidecar must be kept running for live rule synchronization
- Custom rules targeting Aegis API traffic require tuning to eliminate false positives
