---
name: terraform-forge
description: "Terraform/OpenTofu skill for Claude Code. Diagnoses security failure modes before writing code: secret exposure, blast radius, compliance gaps, CI drift. Use when generating, reviewing, refactoring, migrating, testing, or securing IaC."
version: 1.0.0
license: MIT
---

# Terraform Forge: Failure-Mode Workflow for Terraform/OpenTofu

Run this workflow top to bottom.

## 1) Capture execution context

Record before writing code:
- runtime (`terraform` or `tofu`) and exact version
- provider(s), target platform, and state backend
- execution path (local CLI, CI, HCP Terraform/TFE, Atlantis)
- environment criticality (dev/shared/prod)

If unknown, state assumptions explicitly.

## 2) Diagnose likely failure mode(s)

Select one or more based on user intent and risk:
- **identity churn**: resource addressing instability, refactor breakage
- **secret exposure**: secrets in state, logs, defaults, artifacts
- **blast radius**: oversized stacks, weak boundaries, unsafe applies
- **CI drift**: version mismatch, unreviewed applies, missing artifacts
- **compliance gaps**: missing policies/approvals/audit controls

## 3) Load only the relevant reference file(s)

Primary failure-mode references:
- `references/identity-churn.md`
- `references/secret-exposure.md`
- `references/blast-radius.md`
- `references/ci-drift.md`
- `references/compliance-gates.md`

Code and architecture references:
- `references/coding-standards.md`
- `references/module-architecture.md`
- `references/structure-and-state.md`

Delivery and testing references:
- `references/ci-delivery-patterns.md`
- `references/testing-matrix.md`
- `references/security-and-governance.md`
- `references/migration-playbooks.md`
- `references/quick-ops.md`

Pattern banks:
- `references/examples-good.md`
- `references/examples-bad.md`
- `references/examples-neutral.md`
- `references/do-dont-patterns.md`

Integration:
- `references/mcp-integration.md`

## 4) Propose fix path with explicit risk controls

For each fix, include:
- why this addresses the failure mode
- what could still go wrong
- guardrails (tests, approvals, rollback)

## 5) Generate implementation artifacts

When applicable, output:
- HCL changes (typed vars, stable keys, bounded versions)
- migration blocks (`moved`, import strategy)
- CI pipeline updates (plan/apply separation, artifacts, policy checks)
- compliance controls (approvals, policy rules, evidence paths)

## 6) Validate before finalize

Always provide command sequence tailored to runtime and risk tier.
Never recommend direct production apply without reviewed plan and approval.

## 7) Output contract

Return:
- assumptions and version floor
- selected failure mode(s)
- chosen remediation and tradeoffs
- validation/test plan
- rollback/recovery notes for destructive-impact changes
