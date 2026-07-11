# ADR-0006 — Conventional Commits as the commit message standard

> **Status:** Accepted
> **Date:** 2026-06-14
> **Repository:** ecosystem-wide
> **Deciders:** Architect
> **Supersedes:** —

---

## Context

Seven repositories share conventions, CI patterns, and a documentation-sync discipline. A uniform commit-message format was needed so history is machine-parseable (for changelogs/release tooling), scannable by humans across repos, and consistent enough to enforce in review.

---

## Decision Drivers

- Machine-parseable history for changelog/release automation.
- Uniform, low-friction format across all repos.
- A widely understood standard rather than a bespoke convention.

---

## Options Considered

### Option 1 — Conventional Commits (`type(scope): description`)

Types: `feat, fix, docs, chore, build, test, refactor`.

**Pros:**
- Industry standard; tooling already understands it.
- Enables automated changelog/semver derivation.
- `type(scope)` reads well and scans fast in review.

**Cons:**
- Requires author discipline; needs a lint/review gate to stay honest.

---

### Option 2 — Free-form commit messages

**Pros:**
- Zero rules to learn.

**Cons:**
- No machine parsing; inconsistent history; no automated changelogs.

---

## Decision

**We choose Option 1 — Conventional Commits.**

Every commit uses `type(scope): description` with the standard type set. It is Golden Rule 8 in every runtime repo and is what this ADR consolidation itself follows (`docs(adr): …`). The format gives machine-parseable history and a uniform review experience across the ecosystem at the cost of author discipline, which review enforces.

---

## Consequences

### Positive
- Consistent, parseable history across all seven repos.
- Foundation for automated changelogs and release tooling.

### Negative / Trade-offs
- Authors must learn and apply the format; review must catch violations.

### Neutral / Notable
- Pairs with the ecosystem's other committed-code rules (no `// TODO`, no direct commits to `main`).

---

## Implementation Notes

- Type set: `feat, fix, docs, chore, build, test, refactor`.
- Source: Golden Rule 8, present verbatim in every runtime repo's CLAUDE.md.

---

## Review

| Reviewer | Role | Decision | Date |
|---|---|---|---|
| | Architect | Approve / Reject | |
| | Maintainer | Approve / Reject | |
