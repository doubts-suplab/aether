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
| [ADR-0001](ADR-0001-hexagonal-architecture-module-structure.md) | Hexagonal architecture as the standard module structure | Accepted | 2026-06-14 |
| [ADR-0002](ADR-0002-pgvector-vector-store.md) | pgvector as the vector store (Phase 1) | Accepted | 2026-06-14 |
| [ADR-0003](ADR-0003-embedding-model-minilm-ollama.md) | all-MiniLM-L6-v2 via Ollama as the embedding model (Phase 1) | Accepted | 2026-06-14 |
| [ADR-0004](ADR-0004-confidence-gate-threshold.md) | Confidence gate threshold set at 0.8 | Accepted | 2026-06-14 |
| [ADR-0005](ADR-0005-kafka-event-bus-grid-core.md) | Kafka as the event bus between Grid and Core | Accepted | 2026-06-14 |
| [ADR-0006](ADR-0006-conventional-commits.md) | Conventional Commits as the commit message standard | Accepted | 2026-06-14 |
| [ADR-0007](ADR-0007-gdpr-article-17-hard-delete.md) | GDPR Article 17 (Right to Erasure) implemented as hard-delete | Accepted (phased) | — |
| [ADR-0008](ADR-0008-constructor-injection-exclusively.md) | Constructor injection exclusively — no field injection | Accepted | 2026-06-14 |
| [ADR-0009](ADR-0009-jakarta-namespace-exclusively.md) | `jakarta.*` exclusively — no `javax.*` | Accepted | 2026-06-14 |
| [ADR-0010](ADR-0010-federated-operator-console-over-per-repo-spas.md) | A single federated operator console, not a per-repo SPA | Proposed | 2026-07-11 |

> ADR-0001–0009 were formally written up from the per-repository decision logs (see the register below); the decisions themselves predate the documents and are already in force across the ecosystem. ADR-0007's approach is accepted ecosystem-wide, with implementation phased (Aether Core Phase 3).

---

## Repository-Specific Decision Register

Per the charter above, decisions scoped to a single repository stay in that repository — each runtime repo keeps a lightweight log at `.claude/memory/decisions.md`. This register indexes them so the hub is the single point of discovery. Promote any entry to a formal ecosystem ADR here only if it grows cross-cutting.

| Repo | Log | Decisions (local IDs) | Themes |
|---|---|---|---|
| aether-grid | `.claude/memory/decisions.md` | D-001 … D-008 | Spring Cloud Gateway, pgvector/Chroma, Agent SPI, Kafka, SpEL policy, Flyway, modular monolith, transactional outbox |
| aether-core | `.claude/memory/decisions.md` | ADR-001 … ADR-013 | Stack parity, port 8082, isolated DB, REST↔Grid, 384-dim, reinforce-on-read, standalone POM, single active session, session-over-memory precedence, JSONB, phase ordering, optional Kafka consumer, archive-via-CTE |
| aether-memory | `.claude/memory/decisions.md` | ADR-0001 … ADR-0006 | Standalone platform, team scope, visibility ladder, privacy-preserving federation, set-based lifecycle, JaCoCo gate |
| aether-vault | `.claude/memory/decisions.md` | ADR-0001 … ADR-0008 | Standalone platform, collection scope, checksum freshness, bounded RAG, idempotent ingestion, scoped graph edges, char-chunking, adjacency-table graph |
| aether-flow | `.claude/memory/decisions.md` | ADR-0001 … ADR-0007 | Standalone platform, workflow scope, no BPMN/vector/LLM, construction-time validation, per-transition persistence, escalation-marks-only, bounded Grid DEFER intake |
| aether-iel | `.claude/memory/decisions.md` | D-001 … D-005 | Methodology-only (no runtime), 12 phases, 5-level IMM, EEIK bootstrap, governance-from-Phase-1 |

> Numbering note: `aether-core` labels its local log entries `ADR-0NN`, but those are **repository-local** decisions, distinct from the ecosystem `ADR-00NN` documents in this directory. The ecosystem ADRs consolidate cross-cutting rationale that several of those local logs contributed to.

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
