# Threat model

| Threat | Control | Residual risk / validation |
|---|---|---|
| Cross-tenant action | Tenant ID is mandatory and bound into idempotency evidence | Adapter must verify token tenant in Azure |
| Credential disclosure | Secret fields and non-federated modes are denied | Hosted runner and Azure configuration require review |
| Public exposure | Request and Bicep baselines deny public network access | Private DNS and endpoint wiring are integration work |
| Replay or duplicate execution | Stable SHA-256 idempotency key | Adapter must persist keys atomically |
| Stale or forged signal | Fresh-evidence invariant and explicit action allow-list | Source signatures require service-specific adapter |
| Privileged target mutation | Production and break-glass approval gate | Approval authority must be independently governed |
| Evidence tampering | Canonical output plus private evidence store baseline | Immutability policy lock is an Azure integration gate |
| Supply-chain compromise | Minimal dependency-free runtime and read-only CI token | Hosted action provenance and SBOM require release review |

Project-specific acceptance focus: Use malicious and benign plans; test policy accuracy, sensitive-value redaction, override authorization, duplicate alerts, branch protection, tamper resistance, Sentinel correlation, plan/apply drift, and tool outages.
