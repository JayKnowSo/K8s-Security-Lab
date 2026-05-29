# ADR-005: CI Action and Binary SHA Pinning

## Status
Accepted

## Context
On March 19, 2026, a threat actor compromised Aqua Security's GitHub credentials
and force-pushed malicious code to 75 of 76 version tags on
`aquasecurity/trivy-action`. Pipelines using floating tags like `@master` or
`@v0.34.x` silently pulled and executed the backdoored code, exfiltrating secrets
via outbound C2 connections across 12,000+ public repositories.

This pipeline previously referenced:
- `actions/checkout@v4` — mutable tag, reassignable at any time
- `aquasecurity/trivy-action@master` — floating master branch, directly vulnerable
  to the March 2026 compromise
- `kubeconform` installed via `releases/latest` — unpinned binary, no integrity
  guarantee

All three references were vulnerable to the same class of attack: a compromised
upstream repo silently serving malicious code to every pipeline that trusts a
mutable reference.

## Decision
Pin every GitHub Actions reference and binary download to an immutable version:

- `actions/checkout` → SHA `11bd71901bbe5b1630ceea73d27597364c9af683` (v4.2.2)
- `aquasecurity/trivy-action` → SHA `57a97c7e7821a5776cebc9bb87c984fa69cba8f1` (v0.35.0)
  - v0.35.0 is the confirmed safe release protected by GitHub immutable releases,
    published before the March 2026 compromise window opened
- `kubeconform` → pinned to `v0.6.7` explicit release download URL

SHA pins point to a single immutable commit. Even if the upstream repo is
compromised and tags are force-pushed, a pinned pipeline continues executing
the exact code that was audited — not whatever the tag resolves to today.

## Consequences

**Positive:**
- Pipeline is immune to tag-based supply chain attacks (MITRE T1195.001)
- Every CI dependency is auditable and traceable to a specific commit
- Demonstrates active threat intelligence — responding to a real attack in the
  toolchain, not just following theoretical best practice

**Negative:**
- SHA pins require deliberate manual updates when upgrading action versions —
  dependabot or renovate must be configured to automate this or it will drift
- Inline version comments (# v4.2.2) can create false confidence if a SHA is
  updated without updating the comment — the comment becomes a lie that misleads
  future engineers auditing the pipeline
- Engineers must verify new SHAs against upstream release signatures before
  merging any dependency update

## References
- Aqua Security advisory: https://github.com/aquasecurity/trivy/security/advisories/GHSA-69fq-xp46-6x23
- Boost Security Labs analysis: https://labs.boostsecurity.io/articles/20-days-later-trivy-compromise-act-ii/
- Snyk writeup: https://snyk.io/articles/trivy-github-actions-supply-chain-compromise/
- MITRE ATT&CK T1195.001: https://attack.mitre.org/techniques/T1195/001/
