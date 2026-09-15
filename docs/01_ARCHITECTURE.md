# T3N Opportunity Scout — Architecture

## 1. Architectural Goal

T3N Opportunity Scout is a focused agent application that demonstrates a secure, maintainable workflow for researching and ranking opportunities.

Core flow:

`User Request → Agent → T3N Identity/Permissions → Research → Normalize → Score → Recommend`

The system is intentionally small. The goal is to prove a useful agent pattern rather than build a large platform.

## 2. Component Boundaries

### Agent Interface

Accepts the user's research intent and converts it into an internal request model.

Responsibilities:

- validate input;
- define research criteria;
- invoke the research provider;
- pass normalized records to the scoring engine;
- return structured recommendations.

### T3N Integration Layer

Owns T3N-specific authentication, identity, session, and delegated authorization concerns.

Responsibilities:

- initialize the official T3N SDK/ADK;
- authenticate using documented flows;
- obtain tenant/agent identity from the active session or documented registration flow;
- enforce scoped permissions;
- isolate T3N-specific details from business logic.

### Research Provider Adapter

Provides a stable internal contract around external opportunity retrieval.

```text
OpportunityProvider.search(request)
```

The adapter owns provider-specific transport and response parsing. The rest of the application must not depend directly on provider-specific response shapes.

### Normalizer

Converts raw provider records into the internal opportunity schema.

Unknown data remains unknown. The normalizer must never manufacture reward, deadline, eligibility, or other facts.

### Scoring Engine

Pure deterministic business logic. It should not perform network calls or mutate external state.

Input:

- normalized opportunity;
- scoring configuration;
- request criteria.

Output:

- component scores;
- weighted total;
- explanation;
- confidence;
- priority/recommendation state.

### Configuration / Secrets Boundary

Configuration controls non-secret behavior such as scoring weights and provider settings.

Secrets include API keys and private identity material. Secrets must be supplied through environment variables or the official supported secret mechanism and must never be committed.

## 3. Data Flow

```text
1. User submits research request.
        ↓
2. Agent validates and structures request.
        ↓
3. T3N layer establishes required identity/session.
        ↓
4. Scoped authorization permits only required capabilities.
        ↓
5. Research adapter queries approved source.
        ↓
6. Raw records enter normalizer.
        ↓
7. Normalized records are validated.
        ↓
8. Deterministic scoring engine evaluates records.
        ↓
9. Agent ranks opportunities.
        ↓
10. Structured response includes evidence, confidence,
    missing data, and recommended action.
```

## 4. Internal Models

### ResearchRequest

```text
query
skills
opportunityTypes
minimumReward
maxEffortHours
deadlineWindow
eligibilityConstraints
```

Fields may be optional depending on the request.

### Opportunity

```text
id
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
missingFields
```

### ScoreBreakdown

```text
skillFit
reward
_effort
_deadline
eligibility
competition
weightedTotal
explanation
```

Implementation naming may use idiomatic language-specific names while preserving this semantic contract.

## 5. Provider Isolation

The application must be able to replace the first research source without rewriting the scoring engine or T3N integration layer.

Recommended dependency direction:

```text
Agent
 ├── T3N Integration
 ├── OpportunityProvider interface
 │    └── Concrete Provider
 ├── Normalizer
 └── Scoring Engine
```

The core domain must not import provider-specific SDK types.

## 6. T3N Security Boundary

T3N identity and authorization are infrastructure concerns, not scoring concerns.

The application must distinguish:

- tenant identity;
- agent identity;
- delegated permissions;
- external host access;
- application business logic.

Permissions must be narrow enough that a compromised or malfunctioning component cannot gain unnecessary capabilities.

No wallet signing, trading, token issuance, or irreversible financial action belongs in this MVP.

## 7. Runtime Considerations

T3N documentation indicates that the SDK includes WASM components and that some web-framework bundlers may require special handling. Prefer a server-side/plain Node execution boundary if that produces the most reproducible implementation.

Framework complexity should not be introduced unless it materially improves the submission.

## 8. Failure Model

Failures are explicit and observable.

```text
Credential failure      → configuration/auth error
T3N failure             → integration error
Permission denial       → authorization error
Provider timeout        → provider unavailable
Malformed record        → validation error
No opportunities        → empty result
Missing fields          → lower confidence + explanation
```

The system should avoid silently converting failures into successful-looking results.

## 9. Observability

For each run, log only non-sensitive operational information such as:

- run identifier;
- request type;
- provider status;
- number of records retrieved;
- number normalized;
- scoring completion;
- final result count;
- error category.

Never log API keys, private keys, tokens, PII, or raw secret values.

## 10. Reproducibility

A reviewer should be able to reproduce the demonstrated flow from a clean checkout using:

1. documented prerequisites;
2. `.env.example` as configuration guidance;
3. documented T3N setup;
4. documented provider setup;
5. one documented run command;
6. documented expected output shape;
7. test commands.

Any environment-specific limitation must be documented rather than hidden.

## 11. Evolution Path

If the MVP proves useful, later versions may add:

- additional opportunity providers;
- persistent opportunity database;
- richer evidence tracking;
- user-specific scoring profiles;
- action/execution adapters;
- additional agent-to-agent capabilities.

These are deliberately excluded from the first submission so the core agent remains understandable and reliable.
