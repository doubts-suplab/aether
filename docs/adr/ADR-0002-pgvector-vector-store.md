# ADR-0002 — pgvector as the vector store (Phase 1)

> **Status:** Accepted
> **Date:** 2026-06-14
> **Repository:** ecosystem-wide (originated in aether-grid)
> **Deciders:** Architect, DBA Advisor
> **Supersedes:** —

---

## Context

Grid, Core, Memory, and Vault all need vector similarity search over embeddings. The choice was whether to run a dedicated vector database as a separate stateful service, or to use the `pgvector` extension inside the PostgreSQL 16 instance each platform already runs. Every runtime repo already depends on PostgreSQL for its relational data, so the decision materially affects operational surface area.

---

## Decision Drivers

- Minimise the number of stateful services to operate, back up, and secure.
- Preserve each platform's "boots standalone on its own Postgres schema" guarantee.
- Sufficient recall/latency for expected per-tenant vector volumes.
- Keep the store swappable behind a port.

---

## Options Considered

### Option 1 — pgvector as the default store, dedicated DB as an optional adapter

`pgvector` (IVFFlat index) is the default `MemoryStore`/chunk-store adapter; a dedicated vector DB (Chroma) exists as an alternative adapter behind the same port.

**Pros:**
- One fewer container; one backup policy; one failure domain.
- Same DB as relational data — transactional consistency, single connection pool.
- IVFFlat is sufficient for expected memory/chunk volumes.

**Cons:**
- Less specialised than a purpose-built vector DB; may strain at very large scale (>10M vectors/tenant).

---

### Option 2 — Dedicated vector database as primary (Chroma / Milvus / Weaviate)

**Pros:**
- Purpose-built ANN performance at large scale.

**Cons:**
- A second stateful service per platform to run, back up, secure, and keep healthy.
- Breaks the single-store standalone guarantee.
- Two-phase consistency between relational rows and vectors.

---

## Decision

**We choose Option 1 — pgvector as the default, with a Chroma adapter for teams that need it.**

Single-store-on-PostgreSQL is the ecosystem default: it keeps operational surface minimal and preserves the standalone-run guarantee. The `MemoryStore`/chunk-store port means the backing store can change later without touching callers, so choosing pgvector now carries no lock-in.

---

## Consequences

### Positive
- One database to operate per platform; simpler backup and failure model.
- Vectors and relational rows share a transaction boundary.

### Negative / Trade-offs
- pgvector may need index tuning (IVFFlat lists/probes) or a store swap at very high scale.

### Neutral / Notable
- Vault's knowledge graph follows the same single-store principle on relational adjacency tables (Vault decision log ADR-0008), reinforcing "one PostgreSQL, no extra stateful services."

---

## Implementation Notes

- Default adapter uses pgvector with an IVFFlat index; Chroma adapter sits behind the same port.
- Embedding dimension is fixed at 384 (see ADR-0003).
- Source: Grid decision log `D-002`.

---

## Review

| Reviewer | Role | Decision | Date |
|---|---|---|---|
| | Architect | Approve / Reject | |
| | DBA Advisor | Approve / Reject | |
| | Maintainer | Approve / Reject | |
