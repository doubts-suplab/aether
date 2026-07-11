# ADR-0008 — Constructor injection exclusively (no field injection)

> **Status:** Accepted
> **Date:** 2026-06-14
> **Repository:** ecosystem-wide
> **Deciders:** Architect
> **Supersedes:** —

---

## Context

Spring supports several injection styles. Mixing them across seven repos would produce inconsistent, harder-to-test classes and would allow mutable, hidden dependencies. A single injection rule was needed for the whole ecosystem.

---

## Decision Drivers

- Immutable, explicit dependencies (`final` fields).
- Trivially unit-testable without a Spring context or reflection.
- Fail-fast on missing dependencies at construction.
- One uniform rule, enforceable in review.

---

## Options Considered

### Option 1 — Constructor injection only; fields `final`

No field-level `@Autowired`, no `@Inject`.

**Pros:**
- Dependencies are explicit and immutable.
- Tests instantiate with plain constructors — no reflection, no context.
- Missing collaborators fail at construction, not at first use.

**Cons:**
- Verbose constructors for classes with many dependencies (also a design smell signal).

---

### Option 2 — Field / setter `@Autowired`

**Pros:**
- Less boilerplate.

**Cons:**
- Fields cannot be `final`; dependencies are hidden and mutable.
- Tests need reflection or a Spring context.
- Encourages over-wide classes because adding a dependency is "free."

---

## Decision

**We choose Option 1 — constructor injection exclusively, fields `final`.**

Every Spring-managed class receives its collaborators through its constructor; field `@Autowired`/`@Inject` is prohibited. This makes dependencies explicit and immutable, keeps classes testable without a container, and lets a wide constructor act as an honest smell for a class doing too much. It is Golden Rule 1 across the ecosystem.

---

## Consequences

### Positive
- Immutable, explicit dependency graphs; container-free unit tests.
- Fail-fast wiring.

### Negative / Trade-offs
- Verbose constructors for high-dependency classes (usually a prompt to refactor).

### Neutral / Notable
- Field `@Autowired`/`@Inject` is listed as a prohibited pattern in every runtime repo.

---

## Implementation Notes

- Fields holding collaborators must be `final`.
- Source: Golden Rule 1 and the prohibited-patterns list in every runtime repo's CLAUDE.md.

---

## Review

| Reviewer | Role | Decision | Date |
|---|---|---|---|
| | Architect | Approve / Reject | |
| | Maintainer | Approve / Reject | |
