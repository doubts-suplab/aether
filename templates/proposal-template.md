# RFC-NNN — [Proposal Title]

> **Type:** Feature | Integration | Breaking Change | Standard Change | Deprecation
> **Status:** Draft | Under Review | Accepted | Rejected | Withdrawn
> **Author:** 
> **Date:** YYYY-MM-DD
> **Affects:** [list all repositories and teams impacted]
> **Review deadline:** YYYY-MM-DD (minimum 5 business days from submission)

---

## Summary

_One paragraph. What are you proposing and why? A reader should understand the full scope of the proposal from this section alone._

---

## Motivation

_Why is this change needed now? What problem does it solve, what opportunity does it unlock, or what risk does it mitigate? Provide enough context that a reviewer who is unfamiliar with the day-to-day work can understand the urgency or importance._

---

## Detailed Design

_The full technical specification. For a cross-repository change, describe each affected repository separately. Include:_
- _API contract changes (new endpoints, changed fields, removed fields)_
- _Data model changes (new tables, schema migrations)_
- _Integration changes (how the interaction between components changes)_
- _Backward compatibility analysis_

### [Repository / Component 1]

_Changes in this repository:_

### [Repository / Component 2]

_Changes in this repository:_

---

## Alternatives Considered

_What other approaches were evaluated? Why were they not chosen? If no alternatives were considered, explain why this was the obvious choice._

| Alternative | Why Rejected |
|---|---|
| | |

---

## Impact Analysis

### Breaking Changes
_Does this change break any existing API contracts, data formats, or behavioral guarantees? If yes, describe the migration path for each._

### Dependency Order
_In what order must repositories be updated? Lower layers (Core, Grid) must be updated before higher layers (platform, domain applications) that consume them._

```
Step 1: [repository] — [what changes]
Step 2: [repository] — [what changes]
Step 3: [repository] — [what changes]
```

### Rollback Plan
_If this change is deployed and causes problems, how is it rolled back? What is the rollback time target?_

### Performance Impact
_Does this change affect latency, throughput, or resource consumption? Provide estimates if possible._

### Security Impact
_Does this change introduce new attack surface, change authentication/authorization boundaries, or affect compliance posture?_

---

## Migration Plan (if applicable)

_For breaking changes or deprecations: what must existing users do to migrate? What tooling or documentation will be provided? What is the deprecation timeline?_

---

## Open Questions

_List any unresolved questions that reviewers should weigh in on. Assign each question to a decision-maker if possible._

| Question | Assigned To | Resolution |
|---|---|---|
| | | Open |

---

## ADRs Required

_List the Architecture Decision Records that must be written and approved before implementation begins._

| Repository | ADR Title | Status |
|---|---|---|
| | | Pending |

---

## Review Sign-off

| Reviewer | Repository / Team | Decision | Date | Notes |
|---|---|---|---|---|
| | | Approve / Request Changes / Reject | | |
| | | | | |

_All substantive objections must be resolved before this RFC is accepted. "Substantive" means: concerns about correctness, safety, compliance, or fundamental approach. Style preferences and minor suggestions do not block acceptance._
