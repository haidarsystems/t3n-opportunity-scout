# T3N Opportunity Scout — Implementation Specification

**Status:** Ready for implementation
**Project:** `t3n-opportunity-scout`
**Target:** T3N / Terminal 3 Agent Build Challenge

## 1. Purpose

Build a small, credible AI agent that turns a research request into a structured list of opportunities, normalizes the available evidence, applies transparent deterministic scoring, and recommends the next action.

The project is intentionally a focused demonstration of an operator pattern:

`Request → Research → Normalize → Score → Recommend → Act`

It must demonstrate useful agent behavior without becoming a generalized platform.

## 2. MVP Scope

### In scope

1. Accept a structured or natural-language opportunity research request.
2. Query one approved research source through a provider adapter.
3. Normalize returned opportunities into one internal schema.
4. Calculate a deterministic fit/priority score from available evidence.
5. Return structured results with confidence and missing-data indicators.
6. Use T3N identity/authentication according to the official SDK/ADK flow.
7. Demonstrate least-privilege delegated access where required by the selected T3N flow.
8. Keep secrets outside source control.
9. Include tests, documentation, example output, and a reproducible run path.

### Explicitly out of scope

- Trading or financial execution.
- Wallet custody or transaction signing.
- Token issuance.
- Full CRM/database product.
- Browser automation platform.
- Multi-provider orchestration beyond what is needed to prove the adapter boundary.
- Fabricated opportunity data or scores based on unavailable facts.

## 3. Primary Use Case

A user asks the agent to find relevant Web3 opportunities such as bounties, grants, hackathons, or agent-building challenges.

Example request:

> Find open Web3 opportunities relevant to an AI/backend builder, prioritize meaningful rewards, manageable effort, and deadlines that can realistically be met.

The agent gathers permitted source data, normalizes it, scores it, and returns the highest-priority opportunities with reasons and recommended actions.

## 4. Architecture

```text
USER
  ↓
T3N Opportunity Agent
  ↓
T3N Identity / Authentication / Permissions
  ↓
Agent Runtime
  ├── Research Provider Adapter
  ├── Normalizer
  ├── Scoring Engine
  ├── Configuration
  └── Secure State / Secrets Boundary
  ↓
Approved External Source
  ↓
Normalized Opportunity Records
  ↓
Deterministic Scoring
  ↓
Recommendation / Structured Output
```

The architecture must keep external research behind an adapter interface so the core agent is not coupled to one provider.

## 5. T3N Integration Requirements

Use the official Terminal 3 documentation as the source of truth. Do not invent SDK methods, endpoints, permission models, or authentication flows.

The implementation must:

- Use the official T3N SDK/ADK supported by the current documentation.
- Keep the T3N API key in an environment variable or supported secret store.
- Never commit credentials.
- Obtain tenant identity from the authenticated T3N session rather than hardcoding or deriving it.
- Keep tenant identity and agent identity conceptually separate.
- If an agent acts on behalf of a user, use the documented agent identity and scoped delegation model.
- Restrict external hosts/capabilities to the minimum required by the MVP.
- Prefer server-side/plain Node execution if framework bundling creates WASM issues.
- Record any T3N integration limitation or bug discovered during implementation.

## 6. Agent Identity and Authorization

Authentication proves who the agent/tenant is. Authorization determines what the agent may access or perform.

The implementation must follow least privilege:

- No wildcard permissions unless demonstrably required.
- Only required functions/scopes are delegated.
- External hosts are explicitly restricted where supported.
- No financial or irreversible actions are granted.
- Agent identity keys remain local and are never committed or exposed in logs.

## 7. Research Provider Contract

Define an internal provider interface similar to:

```text
search(request) -> raw opportunity candidates
```

The provider is responsible for retrieval. The application layer is responsible for normalization and scoring.

Each normalized opportunity should support, where available:

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

If a field is unavailable, represent it as unknown/null and reduce confidence where appropriate. Never invent values.

## 8. Scoring Model

Use a transparent deterministic score. Initial weighting:

- Skill fit: 35%
- Reward: 20%
- Effort: 15%
- Deadline: 10%
- Eligibility: 10%
- Competition/complexity: 10%

The implementation may adjust a weight only when the available source data makes a component impossible to evaluate. Any adjustment must be documented.

Scores must be explainable. The output should show why an opportunity ranked highly and which data is missing.

## 9. Output Contract

The agent should return a structured response containing:

1. Research request.
2. Timestamp/run identifier.
3. Source/provider used.
4. Ranked opportunities.
5. Score and score explanation.
6. Confidence.
7. Missing/unknown data.
8. Recommended next action.
9. Errors or partial-result warnings, if any.

Example recommendation states:

- `APPLY_NOW`
- `RESEARCH_MORE`
- `LOW_PRIORITY`
- `NOT_ELIGIBLE`
- `INSUFFICIENT_DATA`

## 10. Error Handling

The agent must fail safely and visibly.

Handle at minimum:

- Missing API key.
- Authentication failure.
- T3N initialization failure.
- Provider timeout/failure.
- Empty search results.
- Malformed provider records.
- Missing deadline/reward/eligibility fields.
- Rate limiting.
- Unsupported request.

Partial results should be clearly marked rather than silently presented as complete.

## 11. Security Requirements

- No secrets in Git.
- No API keys in README examples.
- No secrets in logs or screenshots.
- Use `.env.example` with placeholders only.
- Restrict external access to required hosts/capabilities.
- Avoid real PII in tests and demonstrations.
- Do not add wallet signing or financial execution.
- Validate external data before scoring or displaying it.

## 12. Testing

Minimum test layers:

### Unit

- Normalization.
- Score calculation.
- Missing-data behavior.
- Priority classification.
- Input validation.

### Integration

- T3N initialization/authentication path.
- Provider adapter against a safe test fixture or approved test endpoint.
- End-to-end request → normalized output.

### Failure tests

- Missing credentials.
- Provider unavailable.
- Invalid provider response.
- Empty result set.
- Permission denial.

## 13. Documentation Deliverables

The repository should contain:

- `README.md`
- `docs/00_IMPLEMENTATION_SPEC.md`
- `docs/01_ARCHITECTURE.md`
- `docs/02_SECURITY.md`
- `docs/03_AGENT_BEHAVIOR.md`
- `docs/04_TESTING.md`
- `docs/05_SUBMISSION_CHECKLIST.md`
- `docs/06_BUG_FINDINGS.md` (when applicable)
- `.env.example`

Only create additional documentation when it improves reproducibility or judging evidence.

## 14. Submission Evidence

Prepare a concise evidence package showing:

- What problem the agent solves.
- T3N identity/authentication working.
- A real or properly fixture-backed research run.
- Structured normalized output.
- Explainable scoring.
- Security/secret handling.
- Tests passing.
- Maintainability/documentation.
- Any discovered T3N bugs or limitations.

Do not fabricate screenshots, benchmark numbers, bug reports, or successful API calls.

## 15. Definition of Done

The MVP is done when a reviewer can:

1. Clone the public repository.
2. Follow the setup instructions.
3. Configure required secrets without exposing them.
4. Start the agent.
5. Submit a research request.
6. Observe a real T3N-backed identity/authentication flow.
7. Receive structured opportunity results.
8. Understand how scores were calculated.
9. Run the test suite successfully.
10. Understand the architecture and security model from the repository.
11. Reproduce the demonstrated behavior without undocumented manual steps.

## 16. Engineering Priorities

When trade-offs are necessary:

**Correctness > Security > Reproducibility > Maintainability > Feature Count**

Do not add features merely to make the project look larger. A small working agent with clear boundaries is preferable to a broad but fragile demo.
