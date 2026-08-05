# Terraform-Enterprise-Sentinel-Guard

Detect and prevent suspicious Terraform policy bypass attempts before Azure changes occur.

## Example synopsis

A canonical Terraform plan request is gated for tenant, approved target, evidence freshness, and secretless execution before policy evaluation and Sentinel alert adapters process risky changes.

## Real-world scenario

An infrastructure team fears that a rushed override could disable policy or expose a critical resource. The guard blocks suspicious plans before apply and makes attempted bypasses visible to the SOC rather than leaving them only in CI logs.

## Architecture

CI emits a canonical Terraform plan JSON that policy tests evaluate. Denials block deployment; attempted overrides or anomalous high-risk changes send sanitized events to Log Analytics where Sentinel analytics and playbooks alert security owners.

Primary services: `Terraform`, `Azure Policy`, `GitHub Actions`, `Sentinel`, `Logic Apps`.

This repository implements the first production-oriented vertical slice: a
fail-closed, adapter-neutral control plane that validates tenant scope,
freshness, approvals, secretless identity, private access, and the exact
project action before producing a deterministic execution plan. Azure adapters
consume that plan; they are deliberately outside the local simulator so local
tests cannot claim a live cloud change occurred.

```mermaid
flowchart LR
  Request[Desired-state request] --> Validate[Fail-closed validation]
  Validate -->|denied| Evidence[Sanitized denial evidence]
  Validate -->|approved| Plan[Idempotent project plan]
  Plan --> Adapter[Azure adapter integration gate]
  Adapter --> Monitor[Private evidence and monitoring plane]
```

## Quickstart

Requirements: Python 3.11+ and Git. No Azure credentials are required.

```bash
./scripts/validate.sh
python3 src/control_plane.py --request examples/approved-request.json
```

The command emits canonical JSON with a stable idempotency key. The denied
fixture exits with status 2 and explains the failed invariants.

## Security boundaries

- Managed identity or workload identity only; embedded credentials are denied.
- Public network access and stale evidence are denied.
- Production and break-glass targets require explicit approval.
- The IaC entry point is opt-in and defaults to deploying nothing.
- Evidence output contains identifiers and decisions, never credential values.

## Verification and limitations

Local validation covers 13 tests, deterministic replay, JSON parsing, Python
compilation, ignore hygiene, and Bicep compilation when a compiler is present.
It does **not** prove Azure deployment, service licensing, quota, data-plane
permissions, provider/API availability, cloud failover, load, cost, or teardown.
See [[`docs/test-matrix.md`](docs/test-matrix.md)](docs/test-matrix.md) and [[`docs/runbook.md`](docs/runbook.md)](docs/runbook.md) before any integration trial.

## Community

See [[`CONTRIBUTING.md`](CONTRIBUTING.md)](CONTRIBUTING.md), [[`SECURITY.md`](SECURITY.md)](SECURITY.md), [[`SUPPORT.md`](SUPPORT.md)](SUPPORT.md), and [[`LICENSE`](LICENSE)](LICENSE). The reference
is intentionally conservative and uses synthetic identifiers only.

## Repository guide

- [Architecture](docs/architecture.md)
- [Threat model](docs/threat-model.md)
- [Operations runbook](docs/runbook.md)
- [Test matrix](docs/test-matrix.md)
- [Cost model](docs/cost-model.md)
- [Security policy](SECURITY.md)
- [Contributing guide](CONTRIBUTING.md)
- [Support policy](SUPPORT.md)
- [Changelog](CHANGELOG.md)
- [License](LICENSE)
