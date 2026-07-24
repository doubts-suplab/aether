# ADR-0007 — GDPR Article 17 (Right to Erasure) implemented as hard-delete

> **Status:** Accepted (approach); implementation phased — Aether Core Phase 3. Amended for
> multi-jurisdiction retention holds (see Amendment below, V007).
> **Date:** —
> **Repository:** ecosystem-wide (first implemented in aether-core)
> **Deciders:** Architect, Governance Officer
> **Supersedes:** —

---

## Context

Aether Core stores personal memories and cognitive state; other platforms store tenant-scoped data. Under GDPR Article 17 a data subject can demand erasure. The ecosystem had to decide *what "erased" means* in the data model: physically remove the rows, or retain them behind a soft-delete/anonymisation flag. The choice sets a cross-cutting precedent for how every platform satisfies a right-to-erasure request.

> **Status note:** the *decision* (hard-delete) is accepted ecosystem-wide. Its Core implementation is scheduled for Phase 3 (migration V006); this ADR is the record of the chosen approach, not a claim that every platform has shipped it.

---

## Decision Drivers

- A verifiable erasure guarantee — no residual personal data after a valid request.
- Auditability of *that* an erasure happened, without retaining the erased data itself.
- Distinguish erasure (a legal obligation) from the ecosystem's ordinary memory *decay/archive* lifecycle, which is reversible by design.

---

## Options Considered

### Option 1 — Hard-delete (physical removal of personal data)

An erasure request physically removes the subject's rows; proof-of-erasure lives in a separate audit trail, not in the data.

**Pros:**
- True erasure — no residual PII to leak or subpoena.
- Simple, verifiable compliance story.

**Cons:**
- Irreversible; must cascade correctly across all related rows.
- Cannot "undo" an erasure — correctness of the cascade is critical.

---

### Option 2 — Soft-delete / anonymisation flag

Rows are retained with a `deleted`/`anonymised` marker and PII columns nulled.

**Pros:**
- Reversible; preserves referential history.

**Cons:**
- Personal data (or re-identifiable fragments) may linger; weaker erasure guarantee.
- Every read path must honour the flag forever — easy to leak by omission.

---

## Decision

**We choose Option 1 — hard-delete, with no soft-delete for erasure.**

Right to Erasure is satisfied by physically deleting the subject's data; the fact of erasure is recorded in an audit trail rather than by keeping the data. This gives an unambiguous, verifiable guarantee and avoids the "forever honour the flag" fragility of soft-delete.

This is deliberately distinct from the ecosystem's **memory lifecycle**, where faded memories are *archived, never hard-deleted*: Core moves faded memories to `personal_memories_archive` and Memory archives via a data-modifying CTE (Core decision log `ADR-013`; Memory `ADR-0005`). Decay/archive is graceful and reversible; Article-17 erasure is intentional and permanent. The two paths must not be conflated. Grid's `GdprRedactionService` is complementary — it redacts PII *before* persistence so less exists to erase later.

---

## Consequences

### Positive
- Strong, verifiable erasure — nothing residual to leak.
- Clear separation between reversible decay/archive and permanent legal erasure.

### Negative / Trade-offs
- Irreversible; the delete cascade must be complete and correct across every related table.
- No recovery path once executed.

### Neutral / Notable
- Core Phase 3 delivers this via migration V006 (V005 is consumed by the decay archive table); other platforms adopt the same approach as they add erasure endpoints.

---

## Implementation Notes

- Erasure is physical `DELETE`, cascaded across the subject's rows; audit records prove erasure without retaining data.
- Do **not** route erasure through the decay/archive CTE — archive is for reversible forgetting, not legal erasure.
- Source: ecosystem ADR index; Core decision log `ADR-011` (Phase 3 / V006), `ADR-013` (archive contrast); Memory `ADR-0005`; Grid `GdprRedactionService`.

---

## Amendment — Multi-jurisdiction: hard-delete is the *default*, subject to statutory retention holds

> **Added:** Aether Core Phase 3 (session 5), migration V007.

Hard-delete is the default outcome of an erasure request, **not an absolute**. The "delete on
request + immutable audit" primitive is jurisdiction-neutral and satisfies the delete right of GDPR,
CCPA/CPRA, the ~20 US state privacy laws, LGPD, PIPL, DPDP and others alike. What differs across
regimes is **statutory retention exceptions** — GDPR Article **17(3)** (legal claims, legal
obligation), CCPA **§1798.105(d)** (transaction completion, security, legal compliance), and the
retention *mandates* of sectoral US law (HIPAA, GLBA), which can legally **forbid** deletion.

To honour these uniformly, Core adds a `legal_holds` seam (`LegalHoldStore`): a hold marks a
`DataCategory` that must be retained, the erasure service **skips held categories** while still
deleting everything unheld, and the audit event records the retained categories in `held_categories`
so a partial erasure is demonstrable rather than silent. This refines — does not reverse — the
hard-delete decision: unheld data is still physically deleted, and once a hold lifts a subsequent
erasure removes it.

Follow-ups that complete the multi-jurisdiction posture (not yet built): requester identity
verification (CCPA verifiable consumer request), memory export (GDPR Art. 20 / CCPA right-to-know),
and response-SLA orchestration (a natural Aether Flow process, 30 days GDPR / 45 days CCPA).

---

## Review

| Reviewer | Role | Decision | Date |
|---|---|---|---|
| | Architect | Approve / Reject | |
| | Governance Officer | Approve / Reject | |
| | Maintainer | Approve / Reject | |
