# ADR-0004 — Confidence gate threshold set at 0.8

> **Status:** Accepted
> **Date:** 2026-06-14
> **Repository:** ecosystem-wide
> **Deciders:** Architect, Governance Officer
> **Supersedes:** —

---

## Context

Aether agents make decisions (Grid governance decisions; downstream, any agentic action) that can have real consequences. The ecosystem's safety posture is human-in-the-loop: an agent must not act autonomously when it is not confident. A single numeric threshold was needed as the ecosystem-wide gate between "act" and "defer to a human," so the rule is uniform and auditable rather than per-component.

---

## Decision Drivers

- A clear, uniform boundary between autonomous action and human review.
- Conservative default — favour deferral over wrong autonomous action.
- The gate must be enforced identically wherever agents decide.
- Deferrals must reach a human queue, not be dropped.

---

## Options Considered

### Option 1 — Fixed ecosystem-wide threshold of 0.8

Confidence `< 0.8` → never auto-act (never auto-block, in Grid's case) → route to a human.

**Pros:**
- One number, uniform across every component and easy to audit.
- Conservative enough to catch low-confidence decisions.
- Simple to reason about and test.

**Cons:**
- A single global constant ignores that some decisions are riskier than others.

---

### Option 2 — Per-decision / per-policy configurable thresholds

**Pros:**
- Tunable risk per decision type.

**Cons:**
- Many thresholds to govern; inconsistency and drift across components.
- Harder to audit "what makes us defer?" — no single answer.

---

## Decision

**We choose Option 1 — a fixed 0.8 threshold, ecosystem-wide.**

Below 0.8 confidence, an agent never decides autonomously; the decision goes to a human. In Grid this means "never auto-block"; the deferral is projected as a bounded `DeferredDecision` and posted to Aether Flow, which parks it at a human approval gate. The single constant keeps the safety rule uniform and auditable across every component and is a stated golden rule in the AIEL methodology.

---

## Consequences

### Positive
- One unambiguous safety boundary across the ecosystem.
- Low-confidence decisions are always caught and routed to a human.

### Negative / Trade-offs
- A global constant cannot express per-decision risk appetite; genuinely low-risk decisions still defer below 0.8.

### Neutral / Notable
- This gate defines the Grid → Flow DEFER seam: Grid owns the gate; Flow receives what Grid defers and never auto-decides it. Escalation in Flow raises visibility only — it never decides on a human's behalf.

---

## Implementation Notes

- Enforced as an Aether-specific constraint in Grid and mirrored in Flow's constraints (Flow never auto-decides a deferral).
- Codified as AIEL Golden Rule 4: "Confidence < 0.8 → human review always."
- Source: Grid CLAUDE.md constraints; Flow CLAUDE.md; `aether-iel` decision log `D-005` context and golden rules.

---

## Review

| Reviewer | Role | Decision | Date |
|---|---|---|---|
| | Architect | Approve / Reject | |
| | Governance Officer | Approve / Reject | |
| | Maintainer | Approve / Reject | |
