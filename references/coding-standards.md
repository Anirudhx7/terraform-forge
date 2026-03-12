# Coding Standards

Use this guide for implementation-level consistency in Terraform/OpenTofu code generation.

## File organization

| File | Purpose |
|------|---------|
| `main.tf` | Primary resources and data sources |
| `variables.tf` | Input variable declarations |
| `outputs.tf` | Output value declarations |
| `versions.tf` | Terraform and provider version constraints |
| `locals.tf` | Local value declarations |
| `data.tf` | Data sources (optional, for complex setups) |

## Naming and metadata

- use names that reflect business purpose (`payments_api`, `audit_kms`), not temporary implementation details
- reserve `this` for real singleton resources only
- centralize tags/labels in `locals` and apply consistently
- normalize provider-constrained identifiers before use (for example, RDS identifiers and snapshot identifiers)

### Identifier normalization pattern

Some resources reject characters that are valid in domains or service names. Example: `.` in RDS identifiers.

Use a normalization local and reuse it consistently:

```hcl
locals {
  raw_name        = "${var.domain}-prod"
  normalized_name = regexreplace(lower(local.raw_name), "[^a-z0-9-]", "-")
}
```

Then use `local.normalized_name` for fields like `identifier` and `final_snapshot_identifier`.

## Resource block ordering

Preferred order inside resource blocks:
1. identity/core arguments
2. behavior/config arguments
3. nested blocks
4. tags/labels
5. lifecycle/meta-arguments

This keeps diffs predictable and reviewable.

## Variable contract style

Variable attribute order:
1. `description`
2. `type`
3. `default` (if used)
4. `nullable`
5. `sensitive`
6. `validation`

Prefer explicit objects with `optional()` over untyped maps for long-lived module contracts.

## Iteration and identity

- `count`: optional singleton toggles (`0` or `1`)
- `for_each`: any collection with stable logical identities
- never use list index as long-lived identity key

## Set-type handling

Use these rules to avoid ordering bugs and test failures:

- never index sets directly; convert with `sort(tolist(...))` when order matters
- use `for_each` with `toset(...)` only when identity is the value itself
- for stable diff output, transform sets to sorted lists in outputs
- in tests, prefer `contains()` or set equality over positional assertions

## Outputs

- expose only stable interfaces needed by consumers
- mark secret-bearing outputs as `sensitive = true`
- avoid dumping entire provider objects

## Version and lock discipline

- set runtime floor in `required_version`
- bound provider and module versions
- commit lockfile changes intentionally
- keep upgrade PRs separate from functional changes where possible

## Feature guard table (ordered by common LLM mistakes)

Use this when deciding what to emit for a given runtime floor.

| Feature | Min version | LLM error pattern |
|---|---|---|
| `moved` blocks | 1.1+ | omitted during refactor, causing destroy/create |
| `for_each` over `count` for stable identities | 0.12+ | model defaults to `count` for every collection |
| `write_only` arguments | 1.11+ | model uses `sensitive` and assumes state is safe |
| `optional()` defaults | 1.3+ | model emits wrapper variables and loose maps |
| cross-variable validation | 1.9+ | model pushes checks into postconditions only |
| declarative `import` blocks | 1.5+ | model recommends ad-hoc CLI import only |
| `check` blocks | 1.5+ | model ignores runtime assertions entirely |
| `removed` blocks | 1.7+ | model deletes resources with no lifecycle transition |
| provider-defined functions | 1.8+ | model overuses data sources for transformations |

If target runtime is below a feature floor, emit fallback guidance explicitly.

## Modern feature examples

```hcl
# try() — safe fallbacks (0.13+)
output "sg_id" {
  value = try(aws_security_group.this[0].id, "")
}

# optional() — optional attributes with defaults (1.3+)
variable "database_config" {
  description = "Database configuration"
  type = object({
    name           = string
    engine         = string
    instance_class = string
    backup_days    = optional(number, 7)
    monitoring     = optional(bool, true)
    tags           = optional(map(string), {})
  })
}

# cross-variable validation (1.9+)
variable "backup_retention" {
  description = "Backup retention in days"
  type        = number
  validation {
    condition     = var.environment == "prod" ? var.backup_retention >= 7 : true
    error_message = "Production requires backup_retention >= 7 days"
  }
}

# provider-defined functions (1.8+)
locals {
  bucket_arn = provider::aws::arn_build("s3", "my-bucket", data.aws_region.current.name)
}
```

## Version constraint syntax

```hcl
version = "5.0.0"      # Exact (avoid — inflexible)
version = "~> 5.0"     # >= 5.0, < 6.0 (recommended)
version = "~> 5.0.1"   # >= 5.0.1, < 5.1.0
version = ">= 5.0"     # Minimum only (risky)
```

| Component | Strategy | Example |
|-----------|----------|---------|
| Terraform | Pin minor | `required_version = "~> 1.9"` |
| Providers | Pin major | `version = "~> 5.0"` |
| Modules (prod) | Pin exact | `version = "5.1.2"` |
| Modules (dev) | Allow patch | `version = "~> 5.1"` |

## Pre-commit hooks

```yaml
# .pre-commit-config.yaml
- repo: https://github.com/antonbabenko/pre-commit-terraform
  hooks:
    - id: terraform_fmt
    - id: terraform_validate
    - id: terraform_tflint
```

## Review checklist

- typed inputs and validations present
- iteration model chosen for address stability
- no plaintext secret defaults
- version constraints and lockfile strategy are explicit
- migration-sensitive changes include `moved`/`import` plan
