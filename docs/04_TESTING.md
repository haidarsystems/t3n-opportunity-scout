# T3N Opportunity Scout — Testing Strategy

## Testing objective

Prove that the MVP is useful, deterministic, secure, reproducible, and maintainable before submission.

Priority:

**Correctness → Security → Reproducibility → Maintainability → Feature count**

## Test layers

### Unit tests

Test pure logic without network access:

- request parsing
- schema validation
- normalization
- score calculation
- recommendation state
- confidence calculation
- deduplication behavior
- error classification

### Integration tests

Test boundaries between the application and T3N/provider components:

- authenticated T3N session initialization
- tenant identity retrieval
- authorized provider call
- secret loading
- provider response normalization
- end-to-end research flow

Use testnet/sandbox resources and synthetic data wherever possible.

### Security tests

Verify that:

- missing credentials fail safely
- invalid credentials do not produce an authenticated state
- unauthorized capability use is rejected
- secrets are absent from logs
- secrets are absent from generated output
- external prompt-injection text cannot alter agent policy
- malformed external data cannot bypass validation
- unexpected external hosts are not silently accepted

### Failure tests

Explicitly test:

- provider timeout
- provider 4xx/5xx response
- malformed JSON
- missing required fields
- empty result
- partial result
- duplicate records
- expired opportunity
- missing reward/deadline/effort information
- T3N authentication failure
- T3N authorization failure

## Deterministic scoring tests

Given the same normalized input and configuration, the score must be identical across runs.

Create fixtures covering:

1. high-fit/high-reward opportunity
2. high-fit/low-reward opportunity
3. low-fit/high-reward opportunity
4. unknown fields
5. expired opportunity
6. ineligible opportunity
7. boundary values for every scoring dimension

The test suite must verify the documented weights and rounding rules.

## Contract tests

The normalized opportunity schema is an internal contract. Test that provider fixtures map consistently to:

```text
title
description
reward
currency
deadline
skills
eligibility
source
sourceUrl
estimatedEffortHours
fitScore
confidence
priority
recommendedAction
```

Schema changes should require explicit test updates.

## Reproducibility

A reviewer should be able to clone the repository, configure test credentials according to the README, and reproduce the documented test suite.

Tests must not depend on a developer's machine-specific paths or undeclared environment variables.

## Test data policy

- Prefer static fixtures for scoring and normalization.
- Do not depend on live third-party data for unit tests.
- Use synthetic credentials.
- Do not include real personal information.
- Clearly label mocked versus live integration evidence.

## Acceptance gates

### Gate A — Logic

- [ ] Unit tests pass.
- [ ] Score calculations are deterministic.
- [ ] Invalid records are handled safely.

### Gate B — Integration

- [ ] T3N authentication works with documented configuration.
- [ ] Authorized research flow works.
- [ ] Provider adapter produces normalized records.

### Gate C — Security

- [ ] No secrets are committed.
- [ ] Secret leakage tests pass.
- [ ] Permission failures fail closed.
- [ ] Prompt-injection fixture does not change agent policy.

### Gate D — Evidence

- [ ] Clean setup instructions verified.
- [ ] Test command documented.
- [ ] Example output captured.
- [ ] Screenshots contain no secrets.
- [ ] Known limitations documented.

## Definition of test-complete

Testing is complete for the MVP when the core happy path and critical failure/security paths pass, the results are reproducible from a clean checkout, and remaining limitations are explicitly documented rather than hidden.
