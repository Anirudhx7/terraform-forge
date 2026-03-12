# Testing Matrix

Use this guide to choose testing depth proportional to risk and cost.

## Testing layers

1. static checks: format, validate, lint, security scan
2. plan checks: reviewed execution intent
3. native tests: module-level assertions (`terraform test` / `tofu test`)
4. integration tests: ephemeral apply + live assertions
5. Terratest: workflow/system validation in Go for complex scenarios

## Tier A (all changes)

Required:
- `fmt -check`
- `init -backend=false` + `validate`
- lint + security scan
- reviewed plan artifact

Use for low-risk isolated updates.

## Tier B (shared modules or medium-risk changes)

Add:
- native test runs for module behavior
- targeted integration apply tests in ephemeral environment
- policy checks on plan JSON

Typical triggers:
- shared module changes
- IAM/network updates
- encryption/data-boundary updates

## Tier C (high-risk production changes)

Add:
- staged rollout (dev -> stage -> prod)
- rollback rehearsal or documented rollback proof
- manual owner approvals + security/compliance sign-off where needed
- post-apply drift detection

Typical triggers:
- state backend migration
- major refactor with address changes
- foundational platform stack changes

## Native test guidance

### When to use `command = plan`

Use for:
- input validation
- static contract checks
- argument shape assertions not relying on computed runtime values

### When to use `command = apply`

Use for:
- computed attributes known only after creation
- assertions over provider-populated fields
- set/list semantics that are unresolved in plan stage

### Frequent pitfalls

- asserting unknown values in plan mode
- indexing set-type blocks directly
- assuming mocked providers equal integration confidence

## Terratest guidance

Use Terratest when native tests are insufficient:
- cross-module workflows
- external API verification (health checks, connectivity)
- failover/disaster-recovery scenarios
- multi-step lifecycle tests (apply-change-destroy)

### Terratest test pyramid

- fast contract tests (mocked or plan-level)
- environment integration tests (ephemeral)
- limited end-to-end smoke tests for critical paths

### Terratest cost controls

- tag tests by class (`unit`, `integration`, `destructive`)
- parallelize only isolated stacks
- auto-clean resources with TTL tags
- run expensive tests nightly and on protected branches

## Test framework scaffolding

Use a thin, consistent structure so tests are easy to run in CI:

```
.
  test/
    terratest/
      go.mod
      helpers/
      examples/
      network_test.go
    native/
      main.tftest.hcl
  Makefile
```

Minimal Makefile targets:

```makefile
test-native:
\tterraform test

test-terratest:
\tgo test ./test/terratest -timeout 45m
```

## Example command flow

```bash
terraform fmt -check
terraform init -backend=false
terraform validate
terraform plan -out=plan.bin
terraform show -json plan.bin > plan.json
conftest test plan.json --policy policy/
terraform test
```

Terratest stage:

```bash
go test ./test -run TestCriticalPath -timeout 45m
```

## Selection quick rules

- tiny tag change in isolated stack: Tier A
- module contract change: Tier A + Tier B
- refactor with `moved` and shared impact: Tier A + Tier B + targeted Terratest
- production identity/network/encryption/state changes: Tier A + Tier B + Tier C + Terratest smoke

## Done criteria

Not done until:
- required tier passes
- reviewed plan is approved
- apply path is trusted and auditable
- evidence artifacts are retained

## Set-type blocks in native tests

Some blocks are sets (unordered), not lists. You cannot index sets with `[0]`.

| AWS Resource | Block Type | Indexing |
|---|---|---|
| `rule` in S3 encryption config | **set** | Cannot use `[0]` |
| `transition` in lifecycle config | **set** | Cannot use `[0]` |
| `noncurrent_version_expiration` | **list** | Can use `[0]` |

Solution: use `command = apply` with for expressions:

```hcl
run "test_encryption" {
  command = apply

  assert {
    condition = alltrue([
      for rule in aws_s3_bucket_server_side_encryption_configuration.this.rule :
      alltrue([
        for config in rule.apply_server_side_encryption_by_default :
        config.sse_algorithm == "AES256"
      ])
    ])
    error_message = "Encryption should use AES256"
  }
}
```

## Plan vs apply mode decision

```
Checking input values?        → command = plan
Checking computed values?     → command = apply
Accessing set-type blocks?    → command = apply
Need fast feedback?           → command = plan (with mocks)
Testing real behavior?        → command = apply (without mocks)
```

## Mock providers (1.7+)

```hcl
mock_provider "aws" {
  mock_resource "aws_instance" {
    defaults = {
      id  = "i-mock123"
      arn = "arn:aws:ec2:us-east-1:123456789:instance/i-mock123"
    }
  }
}
```

## Terratest patterns (Go)

```go
package test

import (
    "testing"
    "github.com/gruntwork-io/terratest/modules/terraform"
    "github.com/stretchr/testify/assert"
)

func TestS3Module(t *testing.T) {
    t.Parallel()

    terraformOptions := &terraform.Options{
        TerraformDir: "../examples/complete",
        Vars: map[string]interface{}{
            "bucket_name": "test-bucket-" + uniqueId(),
            "tags": map[string]string{
                "Environment": "test",
                "TTL":         "2h",
            },
        },
    }

    defer terraform.Destroy(t, terraformOptions)
    terraform.InitAndApply(t, terraformOptions)

    bucketName := terraform.Output(t, terraformOptions, "bucket_name")
    assert.NotEmpty(t, bucketName)
}
```

### Terratest stages for faster iteration

```go
stage := test_structure.RunTestStage

stage(t, "setup", func() {
    terraform.InitAndApply(t, opts)
})
stage(t, "validate", func() {
    // Assertions here
})
stage(t, "teardown", func() {
    terraform.Destroy(t, opts)
})
// Skip during development: SKIP_setup=true SKIP_teardown=true
```

### Terratest cost estimates

- Small module (S3, IAM): $0-5 per run
- Medium module (VPC, EC2): $5-20 per run
- Large module (RDS, ECS cluster): $20-100 per run

## LLM mistake checklist

Common model mistakes to correct:
- indexes set-type blocks with `[0]` in native tests
- checks computed values in plan mode (unknown at plan time)
- runs destructive tests without isolation or cleanup
- treats mocked provider tests as full integration coverage
- omits `expect_failures` for negative test cases
- uses `count.index` as identity in test assertions
