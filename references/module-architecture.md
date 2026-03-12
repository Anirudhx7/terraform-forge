# Module Architecture

Use this guide when designing reusable modules and composition layers.

## Module type classification

| Type | When to Use | Scope | Example |
|------|-------------|-------|---------|
| **Resource Module** | Single logical group of connected resources | Tightly coupled | VPC + subnets, SG + rules |
| **Infrastructure Module** | Collection of resource modules for a purpose | Multiple modules in one region/account | Complete networking stack |
| **Composition** | Complete infrastructure | Spans multiple regions/accounts | Multi-region deployment |

Hierarchy: Resource → Resource Module → Infrastructure Module → Composition

## Module roles

- `primitive module`: wraps one resource family with strict interface.
- `composite module`: assembles multiple primitives for a deployable capability.
- `root composition`: injects environment values and wiring only.

Keep business policy out of primitives when it is environment-specific.

## Contract design

A good module contract has:

- strongly typed inputs
- defaults only for safe/common behavior
- explicit outputs for consumers
- preconditions for invariants

Bad contract smell:

- many loosely typed maps
- opaque passthrough variables
- outputs that mirror entire provider objects

## Suggested file layout

- `main.tf`: resources and module calls
- `variables.tf`: typed input contract and validation
- `outputs.tf`: explicit consumer interface
- `versions.tf`: runtime and provider constraints
- `locals.tf`: computed values, naming, shared labels
- `README.md`: short usage and contract notes (if repository policy requires docs)

## Composition rules

- pass only required values into child modules
- avoid circular dependencies and hidden ordering
- prefer data flow via input/output over broad `depends_on`
- keep module count manageable; over-fragmentation hurts maintainability

## Deep hierarchy model

Use this when a platform grows beyond a single composition layer.

Hierarchy levels:
- L0 primitives: one resource family, strict contract
- L1 composites: capability units built from primitives
- L2 domain stacks: bounded business domains (payments, identity, observability)
- L3 environment roots: env-specific wiring and configuration
- L4 org orchestration: account/project vending and shared policy baselines

Composition rules:
- dependencies flow downward only (L4 -> L3 -> L2 -> L1 -> L0)
- no lateral imports across same level without an explicit interface contract
- cross-state data flow is via explicit outputs or approved remote state access
- each level owns its state boundary and apply lifecycle
- environment roots should not embed business logic; keep it in L2/L1

Decision aid:
- add a new level only if ownership, lifecycle, or blast radius requires it

## Module release discipline

- tag module versions
- use bounded version constraints in consumers
- run compatibility tests before raising lower bounds

## Decision checkpoint

Create a new module only when one is true:

- reused across 2+ stacks
- ownership differs from current module
- lifecycle differs significantly
- change blast radius needs isolation

## Standard module structure

```text
my-module/
├── README.md
├── main.tf
├── variables.tf
├── outputs.tf
├── versions.tf
├── examples/
│   ├── minimal/        # Simplest working example
│   └── complete/       # Full-featured example
└── tests/
    └── module_test.tftest.hcl
```

Use `examples/` as both documentation and integration test fixtures.

## Architecture principles

1. **Smaller scopes = better performance + reduced blast radius.** Keep modules focused.
2. **Always use remote state** for collaborative environments.
3. **Use `terraform_remote_state` as glue** between stacks, not as the primary integration mechanism. Prefer explicit outputs passed in the same root.
4. **Keep resource modules simple.** No environment-specific logic.
5. **Composition layer: environment-specific values only.** Business logic goes in lower modules.

## Module naming conventions

Public modules: `terraform-<PROVIDER>-<NAME>` (e.g., `terraform-aws-vpc`)
Private modules: `<ORG>-<PURPOSE>` or follow registry naming convention

## Anti-patterns to avoid

- **God modules**: monolithic modules with too many responsibilities. Split by purpose.
- **`count`/`for_each` in root modules for environments**: use separate directories or workspaces instead.
- **`terraform_remote_state` everywhere**: creates tight coupling. Use only when necessary.
- **Untyped interfaces**: `map(any)` hides bugs. Use explicit object types.
- **Environment-specific values in resource modules**: inject via variables from composition layer.

## Testing strategy by module type

| Module Type | Testing Approach |
|---|---|
| Resource module | Static analysis + native tests (plan mode) |
| Infrastructure module | Native tests + integration tests |
| Composition | Plan validation + full integration + policy checks |
