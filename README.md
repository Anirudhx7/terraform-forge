# Terraform Forge

[![Claude Skill](https://img.shields.io/badge/Claude-Skill-6272F5)](https://docs.claude.ai/docs/agent-skills)
[![Terraform](https://img.shields.io/badge/Terraform-1.1+-623CE4)](https://www.terraform.io/)
[![OpenTofu](https://img.shields.io/badge/OpenTofu-1.6+-FFD814)](https://opentofu.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A Terraform/OpenTofu skill for Claude Code. Diagnoses security failure modes before writing code: secret exposure, blast radius, compliance gaps, CI drift. Loads only what's needed, ships an output contract with every response.

---

[Quick Start](#-quick-start) • [How It Works](#-how-it-works) • [What's Included](#-whats-included) • [Repository Layout](#-repository-layout) • [Credits](#credits)

---

## ⚡ Quick Start

**One-line install (recommended)**
```bash
npx skills add anirudhx7/terraform-forge
```

**Manual install**

macOS / Linux:
```bash
git clone https://github.com/anirudhx7/terraform-forge.git ~/.claude/skills/terraform-forge
```

Windows (PowerShell):
```powershell
git clone https://github.com/anirudhx7/terraform-forge.git "$env:USERPROFILE\.claude\skills\terraform-forge"
```

Claude Code auto-discovers skills - no restart needed.

**Invoke explicitly:**
```
/terraform-forge Create a multi-region S3 module with replication
```

**Or just ask naturally** - Forge activates automatically for any Terraform/OpenTofu task:
```
Review my main.tf for secret exposure risks
```
```
Create a GitHub Actions pipeline with plan/apply separation and policy checks
```
```
Migrate this count-based module to for_each safely
```

---

## 🔎 How It Works

Every Terraform task runs through a **7-step failure-mode workflow**:

1. **Capture context** - runtime, providers, backend, environment criticality
2. **Diagnose failure modes** - identity churn, secret exposure, blast radius, CI drift, compliance gaps
3. **Load relevant references** - only the files needed for this specific task
4. **Propose with risk controls** - what could go wrong, guardrails, rollback path
5. **Generate artifacts** - HCL, migration blocks, CI configs, policy rules
6. **Validate** - command sequence tailored to runtime and risk tier
7. **Output contract** - assumptions, tradeoffs, validation plan, recovery notes

---

## 🐣 What's Included

**Diagnostic workflow**
- 7-step failure-mode sequence before any code is generated
- Output contract with assumptions, tradeoffs, and rollback notes
- LLM mistake checklists in every reference file
- Granular reference loading - only relevant files loaded per task

**AWS and HCL patterns**
- Secrets Manager, KMS, S3, IAM, VPC patterns
- Write-only arguments (1.11+) with migration steps
- State bucket hardening with DynamoDB locking
- Modern HCL features: `try()`, `optional()`, cross-variable validation, provider functions

**Testing**
- Native test guidance with set-type block handling
- Terratest Go patterns with staged structure and cost estimates
- Mock providers (1.7+)
- Plan vs apply mode decision matrix

**CI/CD and delivery**
- GitHub Actions, GitLab CI, Atlantis, Infracost templates
- Pipeline hardening checklist
- Automated TTL cleanup for ephemeral test environments

**Security and compliance**
- Trivy, Checkov, OPA/Conftest integration
- 6 compliance framework mappings: ISO 27001, SOC 2, FedRAMP, GDPR, PCI DSS, HIPAA
- Risk-classed approval model with evidence checklist

**Migration**
- `moved`, `import`, refactor, and upgrade playbooks
- 0.12/0.13 to 1.x migration checklist
- Refactoring decision tree

---

## 🔲 Repository Layout

| File | Description |
|------|-------------|
| `SKILL.md` | Core 7-step diagnostic workflow |
| **Failure-mode references** | |
| `references/identity-churn.md` | Address stability, count/for_each, moved blocks, dependency locals |
| `references/secret-exposure.md` | Secret leak prevention, write-only args, rotation playbook |
| `references/blast-radius.md` | Stack boundaries, environment separation, apply governance |
| `references/ci-drift.md` | Pipeline drift prevention, GitHub Actions + GitLab CI templates |
| `references/compliance-gates.md` | ISO 27001, SOC 2, FedRAMP, GDPR, PCI DSS, HIPAA |
| **Code and architecture** | |
| `references/coding-standards.md` | Naming, ordering, modern features, version management, pre-commit |
| `references/module-architecture.md` | Module hierarchy, composition rules, naming, anti-patterns |
| `references/structure-and-state.md` | Repo shape, environment segmentation, state boundaries |
| **Delivery and testing** | |
| `references/ci-delivery-patterns.md` | GitHub Actions, GitLab CI, Atlantis, Infracost, cleanup |
| `references/testing-matrix.md` | Native tests, set-type blocks, Terratest, mocking, cost |
| `references/security-and-governance.md` | Trivy, Checkov, OPA, encryption, IAM, state hardening |
| `references/migration-playbooks.md` | moved, import, refactor, upgrade, 0.12 to 1.x checklist |
| `references/quick-ops.md` | Commands, troubleshooting, TF vs OpenTofu |
| **Pattern banks** | |
| `references/examples-good.md` | Strong implementation patterns |
| `references/examples-bad.md` | Anti-pattern bank |
| `references/examples-neutral.md` | Context-dependent tradeoffs |
| `references/do-dont-patterns.md` | Quick checklist |
| **Integration** | |
| `references/mcp-integration.md` | MCP server guidance |
| `references/token-balance-rationale.md` | Content decisions and tradeoffs |

---

## Credits

- **[TerraShark](https://github.com/LukasNiessen/terrashark)** by Lukas Niessen (MIT)
- **[terraform-skill](https://github.com/antonbabenko/terraform-skill)** by Anton Babenko (Apache 2.0)

## 📄 License

MIT - See [LICENSE](LICENSE) for details.
