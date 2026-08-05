# Architecture

## Context and scope

Detect and prevent suspicious Terraform policy bypass attempts before Azure changes occur.

## Components

CI emits a canonical Terraform plan JSON that policy tests evaluate. Denials block deployment; attempted overrides or anomalous high-risk changes send sanitized events to Log Analytics where Sentinel analytics and playbooks alert security owners.

The checked-in vertical slice owns request validation and deterministic plan
generation. A future Azure adapter owns authentication, retries, provider calls,
and reconciliation. The evidence plane in `infra/main.bicep` supplies a private,
identity-first baseline for logs and immutable-ready storage integration.

## Contract

Inputs are tenant-scoped JSON requests. Approved output includes project,
action, target, correlation ID, and an idempotency key. Any unknown action,
public access, secret material, stale evidence, or unapproved protected target
is rejected before an adapter can run. This separation makes denial behavior
testable without cloud credentials and prevents local demos from mutating Azure.

## Decisions

1. **Fail closed:** missing or unknown fields never downgrade into approval.
2. **Secretless:** only managed or workload identity reaches an adapter.
3. **Private by default:** both application requests and IaC reject public paths.
4. **Human authority:** production and break-glass changes require approval.
5. **Deterministic evidence:** canonical JSON supports replay and audit comparison.
