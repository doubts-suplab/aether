# ADR-0001 — Hexagonal architecture as the standard module structure

> **Status:** Accepted
> **Date:** 2026-06-14
> **Repository:** ecosystem-wide
> **Deciders:** Architect
> **Supersedes:** —

---

## Context

Every runtime platform in the Aether ecosystem (Grid, Core, Memory, Vault, Flow) is a multi-module Maven project that must: enforce DDD bounded contexts, keep domain logic free of framework coupling, and allow infrastructure adapters (JDBC stores, HTTP clients, message consumers) to be swapped without touching the domain. A shared, repeatable module shape was needed so that a reviewer moving between repos sees the same structure and the same dependency rules everywhere.

This ADR retro-documents the decision that was applied at ecosystem inception and is now realised identically in all five runtime repos.

---

## Decision Drivers

- Domain logic must not depend on Spring or any framework — it must be plain Java 21 records/interfaces.
- Cross-module calls must go through port interfaces, never another module's internals (Golden Rule 5).
- Adapters (vector store, embedding, HTTP, Kafka) must be replaceable without domain changes.
- A single mental model that transfers across every repo.

---

## Options Considered

### Option 1 — Hexagonal (ports & adapters) multi-module layout

A `*-domain` module of pure records + port interfaces, a `*-engine`/`*-memory` module of adapters and services, a `*-api` Spring Boot application, and a no-Java `*-infra` module for Docker/k8s.

**Pros:**
- Domain is framework-free and independently testable.
- Adapters swap behind ports (pgvector ↔ Chroma, Ollama ↔ other) with no domain change.
- Identical shape across every repo — legible and reviewable.

**Cons:**
- More modules and indirection than a single-module app.
- Requires discipline to keep ports meaningful rather than pass-through.

---

### Option 2 — Single-module layered application

One module with `controller` / `service` / `repository` packages.

**Pros:**
- Fewer moving parts; faster to bootstrap.

**Cons:**
- No compile-time enforcement of the domain/framework boundary.
- Encourages the domain to leak Spring annotations and JDBC types.
- Diverges from the ecosystem norm.

---

## Decision

**We choose Option 1 — the hexagonal multi-module layout.**

Every runtime repo uses `*-domain` (pure Java, no framework), an engine module of adapters + services, a Spring Boot `*-api`, and a no-Java `*-infra`. The dependency graph is always `api → {domain, engine}` and `engine → domain`. This makes the domain/framework boundary a compile-time guarantee, not a convention, and gives one structure across the ecosystem.

---

## Consequences

### Positive
- Domain modules compile without Spring on the classpath — the boundary cannot silently erode.
- Adapters are swappable behind ports; standalone-run guarantees hold.
- One structure to learn; reviews transfer across repos.

### Negative / Trade-offs
- More modules to manage than a flat app.
- Ports must be curated to stay purposeful.

### Neutral / Notable
- Grid runs as a modular monolith (two apps sharing a schema; see the Grid decision log D-007); the module discipline keeps future service extraction cheap because ports/adapters are already clean.

---

## Implementation Notes

- `*-domain` has zero framework dependencies — enforced by its POM.
- Cross-module access is via port interfaces defined in `*-domain`.
- Source: Grid `D-007`; the shared module layout documented in every repo's CLAUDE.md.

---

## Review

| Reviewer | Role | Decision | Date |
|---|---|---|---|
| | Architect | Approve / Reject | |
| | Governance Officer | Approve / Reject | |
| | Maintainer | Approve / Reject | |
