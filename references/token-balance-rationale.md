# Token Balance Rationale

What was kept, what was added, and why.

## Design position

Forge sits between the two skills it builds on:

- TerraShark: ~600 token activation, lean diagnostic workflow, no deep HCL patterns
- terraform-skill: ~4,400 token activation, deep HCL patterns, no diagnostic workflow
- Terraform Forge: ~900–1,100 token activation, diagnostic workflow preserved, production patterns merged in

The goal is best output quality per token spent, not minimum tokens at all costs.

## What was added and why

Content was merged from terraform-skill only when it met both conditions:
1. Not already covered in TerraShark's references
2. Directly reduces a known LLM hallucination or improves output correctness

| Content | Added to | Reason |
|---|---|---|
| Set-type block handling | `testing-matrix.md` | LLMs routinely index sets with `[0]` — causes test failures |
| Terratest Go patterns + cost estimates | `testing-matrix.md` | Native tests insufficient for cross-module workflows |
| Mock provider examples (1.7+) | `testing-matrix.md` | Reduces test cost; LLMs don't emit these unprompted |
| `try()`, `optional()`, cross-var validation, provider functions | `coding-standards.md` | LLMs emit legacy patterns when modern equivalents exist |
| Version constraint syntax + strategy table | `coding-standards.md` | LLMs float versions or use exact pins; both are wrong |
| File organization table | `coding-standards.md` | LLMs dump everything in `main.tf` |
| Locals for dependency/deletion ordering | `identity-churn.md` | Non-obvious pattern; LLMs miss deletion order bugs |
| State bucket hardening + DynamoDB locking | `security-and-governance.md` | LLMs emit S3 backends without encryption or locking |
| Infracost GitHub Actions integration | `ci-delivery-patterns.md` | Template completeness |
| TTL cleanup pattern | `ci-delivery-patterns.md` | LLMs create test infra with no cleanup strategy |
| 0.12/0.13 → 1.x migration checklist | `migration-playbooks.md` | Covers legacy codebases; LLMs miss `try()` substitution |
| Declarative `import` blocks with `for_each` | `migration-playbooks.md` | LLMs default to CLI import even when 1.5+ is available |
| Pre-commit hooks config | `coding-standards.md` | Completeness; commonly requested |

## What was not merged

- Broad tutorial prose already known to LLMs (basic HCL syntax, init/plan/apply explanations)
- terraform-skill's `quick-reference.md` — covered by `quick-ops.md` and inline decision matrices
- Provider deep dives beyond AWS patterns already present

## Activation cost tradeoff

The ~300–500 token increase over TerraShark is the cost of having Terratest patterns, set-type block handling, and modern HCL features available without needing a second skill. For most real Terraform tasks this pays for itself in the first avoided hallucination.

## When to add content

Add when:
- a real LLM mistake pattern is identified in practice
- removing it would measurably drop output correctness
- the content does not duplicate something already present

Do not add:
- tutorial prose that restates Terraform docs
- examples added for completeness with no failure-mode backing
- framework overviews with no actionable gates