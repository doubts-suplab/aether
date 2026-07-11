# ADR-0009 — `jakarta.*` exclusively (no `javax.*`)

> **Status:** Accepted
> **Date:** 2026-06-14
> **Repository:** ecosystem-wide
> **Deciders:** Architect
> **Supersedes:** —

---

## Context

The ecosystem standardises on Spring Boot 3.x, which moved the enterprise APIs from the `javax.*` namespace to `jakarta.*`. A stray `javax.*` import compiles in some IDE setups but fails or misbehaves at runtime against the Jakarta EE 9+ APIs Spring Boot 3 ships. A hard, unambiguous rule was needed to keep the namespace clean across every repo.

---

## Decision Drivers

- Correctness on Spring Boot 3.x (Jakarta EE 9+).
- A rule that can be enforced as a build-breaking error, not a style preference.
- No accidental `javax.*` creeping in via copy-paste or old snippets.

---

## Options Considered

### Option 1 — `jakarta.*` only; `javax.*` is a build-breaking error

**Pros:**
- Correct for Spring Boot 3.x runtime.
- Unambiguous; enforceable in CI/review.

**Cons:**
- Older snippets/tutorials must be translated when adopted.

---

### Option 2 — Allow `javax.*` where it still resolves

**Pros:**
- Slightly less friction pasting legacy code.

**Cons:**
- Runtime failures against Jakarta APIs; a latent trap.
- Inconsistent imports across the codebase.

---

## Decision

**We choose Option 1 — `jakarta.*` exclusively; any `javax.*` import is treated as a build-breaking error.**

Spring Boot 3.x is a hard ecosystem baseline and it is Jakarta-namespaced. Allowing `javax.*` invites runtime failures, so it is prohibited outright rather than discouraged. This is Golden Rule 10 across the ecosystem.

---

## Consequences

### Positive
- Guaranteed namespace correctness on Spring Boot 3.x.
- No latent `javax.*` runtime traps.

### Negative / Trade-offs
- Legacy code must be translated to `jakarta.*` before reuse.

### Neutral / Notable
- Listed as a prohibited pattern in every runtime repo; pairs with the Spring Boot 3.3.x baseline in the shared stack.

---

## Implementation Notes

- Treat `javax.*` as a compile failure (enforce in CI where practical).
- Source: Golden Rule 10 and the prohibited-patterns list in every runtime repo's CLAUDE.md.

---

## Review

| Reviewer | Role | Decision | Date |
|---|---|---|---|
| | Architect | Approve / Reject | |
| | Maintainer | Approve / Reject | |
