# AetherCore — Architecture

> **Repo:** [`suplab/aether-core`](https://github.com/suplab/aether-core)
> **Role:** Personal Cognitive Engine — the brain of the Aether ecosystem
> **IMM Level:** 4 — Cognitive Platform
> **Status:** Phases 0–6 complete, production-ready

---

## Purpose

AetherCore is the persistent cognitive engine at the centre of the Aether ecosystem. Its job is to maintain a continuously evolving model of an individual's knowledge, experience, preferences, and emotional context — and to make that model available to any agent that needs it.

Every Aether Grid agent that makes a decision about a user first calls AetherCore to retrieve a personal context snapshot. That snapshot tells the agent who this person is, what they care about, how they prefer to communicate, and what has been relevant to them recently. Without Core, agents operate with no context. With Core, agents operate with a rich, living memory.

---

## Module Structure

AetherCore uses hexagonal (ports and adapters) architecture across four Maven modules. The dependency direction is strictly one-way: `core-domain` ← `core-memory` ← `core-api`. No module imports another's internal implementations.

```
core-domain   ← Pure Java, no Spring. Domain types + port interfaces.
core-memory   ← Spring JDBC adapters implementing the port interfaces.
core-api      ← Spring Boot application. Wires ports to adapters. Exposes REST API.
core-infra    ← Docker Compose, Flyway migrations, Kubernetes manifests.
```

### core-domain

Contains the canonical domain types and port interface definitions. Has no Spring dependency — domain logic is pure Java.

**Domain records:**
- `PersonalMemory` — a single memory: content, type, strength (0.0–1.0), accessCount, embedding vector, userId, tenantId, timestamps
- `MemoryType` — enum: `EPISODIC`, `SEMANTIC`, `PROCEDURAL`, `EMOTIONAL`
- `CognitiveSession` — a multi-turn reasoning session with turn summaries, emotional state, and engagement score
- `PersonalContext` — assembled snapshot delivered to Grid agents: recent memories, active session, context summary
- `GdprConsent` — consent record: memoryStorageAllowed boolean, dataRetentionDays, timestamps

**Port interfaces:**
- `PersonalMemoryStore` — save, findSimilar (vector search), findByType, delete, reinforceMemory
- `PersonalContextProvider` — assemble and return a PersonalContext for a given user and tenant
- `CognitiveSessionStore` — save, findActiveSession, findById, findRecentByUser, close, eraseByUser
- `GdprConsentStore` — save, findByUser, isMemoryStorageAllowed (default method)

### core-memory

Spring-based adapter implementations behind the port interfaces. All database access uses `NamedParameterJdbcTemplate` — no ORM, no Hibernate, no JPA.

**Key adapters:**
- `PGVectorPersonalMemoryStore` — pgvector cosine similarity search, upsert on conflict, reinforce-on-read (+0.1 per retrieval, capped at 1.0)
- `PersonalEmbeddingService` — calls Ollama `all-MiniLM-L6-v2` to produce 384-dim embeddings; `@ConditionalOnProperty(aether.core.embedding.enabled)` for graceful degradation
- `JdbcCognitiveSessionStore` — JDBC session adapter; turn summaries stored as PostgreSQL `TEXT[]`
- `JdbcGdprConsentStore` — UPSERT on `(user_id, tenant_id)` primary key
- `MemoryDecayService` — `@Scheduled` daily decay (02:00 UTC), weekly purge at strength ≤ 0.05 (03:00 UTC Sunday), session idle expiry
- `GridFeedbackConsumer` — `@KafkaListener` on `aether.core.feedback`; CORRECT → reinforcement, INCORRECT → decay signal; `@ConditionalOnProperty(aether.core.kafka.enabled)`

### core-api

Spring Boot 3.x application that wires port interfaces to their adapter implementations via `CoreApiConfig`. Exposes the REST API consumed by AetherGrid.

**Controllers:**
- `PersonalContextController` — `GET /api/v1/personal-context/{tenantId}/{userId}` — the primary Grid integration endpoint
- `PersonalMemoryController` — POST store / GET count / DELETE by id
- `CognitiveSessionController` — POST create-or-resume / PUT add-turns / POST close / GET recent / GET by-id
- `GdprController` — GET consent / PUT consent / DELETE erase (hard-delete cascade)

---

## Memory Lifecycle

Memory in AetherCore is not a flat log. It has a lifecycle with discrete stages:

```
Capture  →  Embed  →  Store  →  Retrieve  →  Reinforce  →  Decay  →  Purge
```

**Capture:** A memory is created with content, type, userId, and tenantId. Initial strength = 1.0.

**Embed:** The content is passed to `PersonalEmbeddingService` which calls Ollama to produce a 384-dimensional vector. This vector is stored alongside the content in pgvector.

**Store:** The memory is persisted via `PGVectorPersonalMemoryStore` with `INSERT ... ON CONFLICT DO UPDATE` semantics. If a memory with the same userId + tenantId + content hash already exists, its strength is reinforced rather than duplicated.

**Retrieve:** `findSimilar()` performs a cosine similarity search using pgvector's `<=>` operator, returning the top-K most semantically relevant memories. The ivfflat index keeps this fast even at millions of memories.

**Reinforce:** Every retrieval triggers `reinforceMemory()` — strength += 0.1, accessCount++, lastAccessedAt = now. Memories that are accessed often grow stronger and persist longer. This mirrors how human memory consolidation works.

**Decay:** `MemoryDecayService` runs daily at 02:00 UTC. It applies a configurable decay factor (default 5%) to all memories that have not been accessed in the past 7 days: `strength = GREATEST(0.0, strength * (1.0 - decayFactor))`.

**Purge:** Weekly on Sunday at 03:00 UTC, all memories with strength ≤ 0.05 are hard-deleted. The threshold is configurable via `MEMORY_PURGE_THRESHOLD`.

---

## Integration with AetherGrid

AetherCore and AetherGrid are separate services. Grid calls Core's REST API. Core does not call Grid. This is a strict dependency direction that must not be violated.

The integration contract is:

```
GET /api/v1/personal-context/{tenantId}/{userId}
```

Grid's `AetherCoreBridgeAgent` calls this endpoint before each agent decision. The response contains the user's recent relevant memories, active cognitive session state, and a synthesized context summary. The agent uses this context to make a more informed decision.

Feedback flows in the other direction over Kafka. When Grid agents make a decision that is subsequently evaluated as CORRECT or INCORRECT, they publish a `FeedbackEvent` to the `aether.core.feedback` topic. Core's `GridFeedbackConsumer` processes this and updates the underlying memory strength accordingly.

---

## Database Schema

Four Flyway migrations create the database structure:

| Migration | Table | Key Columns |
|---|---|---|
| V001 | `personal_memories` | `id UUID`, `user_id`, `tenant_id`, `content TEXT`, `memory_type`, `embedding vector(384)`, `strength FLOAT`, `access_count INT` |
| V002 | pgvector extension + ivfflat cosine index | `CREATE INDEX ... USING ivfflat (embedding vector_cosine_ops)` |
| V003 | `cognitive_sessions` | `id UUID`, `user_id`, `tenant_id`, `turn_summaries TEXT[]`, `active BOOLEAN`, partial index on `active = TRUE` |
| V004 | `gdpr_consent` | `(user_id, tenant_id)` composite PK, `memory_storage_allowed BOOLEAN`, `data_retention_days INT CHECK >= 0` |

All tables include `tenant_id` in every query. Row-level isolation between tenants is enforced at the query level, not by database roles.

---

## GDPR Compliance

AetherCore implements GDPR Article 17 (Right to Erasure) as a hard architectural requirement.

When `DELETE /api/v1/users/{userId}/gdpr/erase` is called with a verified request:
1. All `personal_memories` for the user (all four MemoryType values) are hard-deleted
2. All `cognitive_sessions` for the user are hard-deleted via `eraseByUser()`
3. The `gdpr_consent` record is updated to `optOut()` status (memoryStorageAllowed = false)

All three steps happen within a single controller method. There is no soft-delete. There is no archive. Data is gone.

---

## Non-Functional Characteristics

| Concern | Approach |
|---|---|
| Latency | pgvector ivfflat index keeps similarity search sub-200ms at 1M+ memories |
| Availability | 2 replicas in Kubernetes, HPA scales to 8 on CPU pressure |
| Resilience | `@ConditionalOnProperty` gates on embedding and Kafka — Core runs standalone |
| Observability | Micrometer + Prometheus, OpenTelemetry tracing, SLF4J structured logging |
| Security | Non-root container (uid 1000), read-only root filesystem, no capability escalation |
| Build | Multi-stage Dockerfile, eclipse-temurin:21-jre-noble runtime, multi-arch (amd64 + arm64) |
