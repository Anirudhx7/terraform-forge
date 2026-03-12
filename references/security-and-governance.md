# Security and Governance

Use this guide for security controls in IaC delivery. For framework mappings and evidence gates, use `compliance-gates.md`.

## Identity controls

- least privilege for CI identities
- separate `plan` and `apply` roles where possible
- short-lived credentials via workload identity federation
- deny direct human write access to production backends

## Secret controls

- prohibit plaintext secret defaults in code
- source sensitive values from managed secret stores
- mark secret variables and outputs as sensitive
- sanitize logs/artifacts and restrict access

## Supply-chain controls

- pin provider/module versions with bounded constraints
- commit lockfile and review lockfile diffs
- verify action/container versions in CI workflows

## Policy layers

Use layered controls, not single-tool reliance:
1. static scanners (`trivy`, `checkov`)
2. plan-policy checks (Sentinel/OPA/Conftest)
3. approval gates by risk class

### Trivy (successor to tfsec)

```bash
brew install trivy          # macOS
trivy config .              # scan

# In CI:
- uses: aquasecurity/trivy-action@master
  with:
    scan-type: 'config'
    scan-ref: '.'
```

### Checkov

```bash
checkov -d . --framework terraform
checkov -d . --skip-check CKV_AWS_23       # skip specific
checkov -d . -o json > checkov-report.json  # CI report
```

### terraform-compliance (BDD-style)

```bash
pip install terraform-compliance
terraform plan -out=tfplan && terraform show -json tfplan > tfplan.json
terraform-compliance -f compliance/ -p tfplan.json
```

```gherkin
Feature: AWS Resources must be encrypted
  Scenario: S3 buckets must have encryption
    Given I have aws_s3_bucket defined
    When it has aws_s3_bucket_server_side_encryption_configuration
    Then it must contain rule
    And it must contain apply_server_side_encryption_by_default
```

## High-impact change controls

Require elevated approval for:
- IAM privilege expansion
- network exposure/public ingress changes
- encryption disablement/key-policy weakening
- backend/state changes
- production replacement/destruction actions

## Minimal OPA example

```rego
package main

deny[msg] {
  r := input.resource_changes[_]
  r.type == "aws_security_group_rule"
  r.change.after.cidr_blocks[_] == "0.0.0.0/0"
  r.change.after.from_port == 22
  msg := sprintf("Public SSH is not allowed: %s", [r.address])
}
```

## Operational governance

- serialize applies for shared foundations
- require explicit opt-in for destroy
- keep break-glass runbook and test it periodically
- retain run metadata and policy outputs for auditability

## Encryption defaults

Always enable encryption at rest:

```hcl
resource "aws_s3_bucket_server_side_encryption_configuration" "this" {
  bucket = aws_s3_bucket.this.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "aws:kms"
      kms_master_key_id = var.kms_key_arn
    }
  }
}

resource "aws_s3_bucket_public_access_block" "this" {
  bucket                  = aws_s3_bucket.this.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

## Network security

- never deploy to default VPC
- restrict security groups to specific ports and CIDRs
- no ingress from `0.0.0.0/0` without explicit justification
- enable VPC flow logs for audit trails

## IAM least privilege

```hcl
resource "aws_iam_policy" "app" {
  name = "${var.project}-app-policy"
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect   = "Allow"
      Action   = ["s3:GetObject", "s3:PutObject"]
      Resource = "${aws_s3_bucket.app_data.arn}/*"
    }]
  })
}
```

Never use wildcard `"*"` for actions or resources.

## State bucket hardening

```hcl
resource "aws_s3_bucket" "terraform_state" {
  bucket = "my-terraform-state"
}

resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration { status = "Enabled" }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  rule {
    apply_server_side_encryption_by_default { sse_algorithm = "AES256" }
  }
}

resource "aws_s3_bucket_public_access_block" "terraform_state" {
  bucket                  = aws_s3_bucket.terraform_state.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

Restrict access via IAM policy to only the Terraform execution role. Always use DynamoDB locking.
