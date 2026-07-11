# ADR-0003 — all-MiniLM-L6-v2 via Ollama as the embedding model (Phase 1)

> **Status:** Accepted
> **Date:** 2026-06-14
> **Repository:** ecosystem-wide
> **Deciders:** Architect, AI Engineer
> **Supersedes:** —

---

## Context

Grid, Core, Memory, and Vault embed text for similarity search. The embedding model determines the vector dimension baked into every schema, so it is a cross-cutting contract: for two platforms ever to compare vectors (e.g. cross-referencing personal memory against enterprise API patterns), model and dimension must match. A single model and runtime had to be chosen for the whole ecosystem.

---

## Decision Drivers

- One dimension across the ecosystem so vectors are comparable.
- Local, dependency-light default (no mandatory cloud/API key to run).
- The runtime must be replaceable — no direct HTTP coupling to one vendor.
- Small, fast, good-enough quality for the scaffold phase.

---

## Options Considered

### Option 1 — all-MiniLM-L6-v2 (384-dim) served via Ollama, behind an embedding port

**Pros:**
- Small and fast; runs locally with no external API key.
- 384-dim standardised across all repos → comparable similarity scores.
- Reached through `*EmbeddingService` / `LlmClient` ports, so Ollama is swappable.

**Cons:**
- Lower ceiling than large hosted embedding models.
- Changing the model/dimension later requires a full re-embedding migration.

---

### Option 2 — A hosted embedding API (OpenAI / Cohere / Voyage)

**Pros:**
- Higher-quality embeddings, no local model to run.

**Cons:**
- Mandatory external dependency and API key — breaks standalone-run.
- Per-call cost and network latency; data leaves the boundary.

---

## Decision

**We choose Option 1 — all-MiniLM-L6-v2 (384-dim) via Ollama, behind an embedding port.**

It runs locally with no external dependency, standardises the vector dimension at 384 across the ecosystem, and stays swappable: all embedding calls go through a service port (`PersonalEmbeddingService`, `SharedEmbeddingService`, `KnowledgeEmbeddingService`, or Grid's `LlmClient`), never direct HTTP. Grid additionally allows other LLM providers via `AETHER_LLM_PROVIDER` behind the same interface.

---

## Consequences

### Positive
- Standalone-runnable with no cloud dependency.
- 384-dim is a stable, comparable contract across platforms.

### Negative / Trade-offs
- Embedding quality is capped by a small model.
- **Changing the dimension is a breaking change** requiring a full re-embedding migration in every affected platform.

### Neutral / Notable
- The port abstraction means a better model can be adopted later without touching callers — only the re-embedding migration and the dimension constant change.

---

## Implementation Notes

- Dimension `384` is a hard constraint in each platform's CLAUDE.md; changing it is an explicit, ecosystem-wide re-embedding migration.
- Embedding access is exclusively through the platform's embedding port, never direct HTTP to Ollama.
- Source: Core decision log `ADR-005`; embedding stack in every runtime repo's CLAUDE.md.

---

## Review

| Reviewer | Role | Decision | Date |
|---|---|---|---|
| | Architect | Approve / Reject | |
| | AI Engineer | Approve / Reject | |
| | Maintainer | Approve / Reject | |
