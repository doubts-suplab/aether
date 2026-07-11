# Architecture Decision Records

> ADRs are the canonical record of every significant architectural decision made in the Aether ecosystem.
> They are immutable once approved — a decision reversed gets a new ADR that supersedes the old one, not an edit.

---

## What Belongs Here

This directory contains ecosystem-wide ADRs: decisions that affect more than one repository, establish cross-cutting standards, or define constraints that all components must respect.

Repository-specific ADRs live in `docs/adr/` within the relevant repository.

---

## Index

| ADR | Title | Status | Date |
|---|---|---|---|
| ADR-0001 | Hexagonal architecture as the standard module structure | Accepted | — |
| ADR-0002 | pgvector as the vector store for AetherCore (Phase 1) | Accepted | — |
| ADR-0003 | all-MiniLM-L6-v2 via Ollama as the embedding model (Phase 1) | Accepted | — |
| ADR-0004 | Confidence gate threshold set at 0.8 | Accepted | — |
| ADR-0005 | Kafka as the event bus between Grid and Core | Accepted | — |
| ADR-0006 | Conventional Commits as the commit message standard | Accepted | — |
| ADR-0007 | GDPR Article 17 implemented as hard-delete (no soft-delete) | Accepted | — |
| ADR-0008 | Constructor injection exclusively — no field injection | Accepted | — |
| ADR-0009 | Jakarta.* exclusively — no javax.* | Accepted | — |
| ADR-0010 | A single federated operator console, not a per-repo SPA | Proposed | 2026-07-11 |

> **Note:** The ADR documents above are pending formal write-up using `templates/adr-template.md`. The decisions themselves are already implemented across the ecosystem. Formal ADR documents are a near-term backlog item.

---

## How to Add an ADR

1. Copy `templates/adr-template.md` to this directory
2. Name the file `ADR-NNNN-short-title.md` (zero-padded four-digit number)
3. Fill in all sections — context, options, decision, consequences
4. Open a pull request with the ADR as the only change
5. Get review from the relevant maintainers and the Governance Officer for Category D changes
6. Merge — the ADR is now accepted and immutable

---

## ADR Lifecycle

```
Proposed → Under Review → Accepted → [Superseded by ADR-NNNN]
                       ↘ Rejected
                       ↘ Withdrawn
```

An accepted ADR is never edited. If the decision changes, a new ADR is written that references and supersedes the original. Both ADRs remain in the index permanently.

---

## Cross-Reference with AIEL

The AIEL methodology requires ADRs at several phases:
- **Phase 2** (Cognitive Architecture): technology selection decisions — vector store, embedding model, LLM provider
- **Phase 5** (Agent Engineering): agent authority boundary decisions
- **Phase 9** (Governance Engineering): policy and audit decisions
- **Phase 11** (Evolution Engineering): major capability additions

See `aether-iel/lifecycle/` for the phase-specific ADR requirements.
