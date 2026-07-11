# ADR-0005 — Kafka as the event bus between Grid and Core

> **Status:** Accepted
> **Date:** 2026-06-14
> **Repository:** ecosystem-wide (aether-grid ↔ aether-core)
> **Deciders:** Architect, AI Engineer
> **Supersedes:** —

---

## Context

Grid's proxy captures API calls on the request path; agents reason about them asynchronously; and Core learns from Grid decision feedback (the Phase 4 learning loop, topic `aether.core.feedback`). These flows must be temporally decoupled — the request path cannot block on async reasoning — and events must not be lost if a downstream consumer is briefly unavailable. A durable, replayable transport was needed between components.

---

## Decision Drivers

- Temporal decoupling between the proxy request path and async agent/learning work.
- Durability and replay — no lost events on transient consumer downtime.
- Reliable publish despite the dual-write problem (DB + broker).
- Consumers must be optional so each platform still boots standalone.

---

## Options Considered

### Option 1 — Apache Kafka with the transactional outbox pattern

`ApiCall` and `outbox_events` rows are written in one DB transaction; a relay publishes to Kafka and marks rows published. Core consumes feedback via a consumer that is disabled unless explicitly enabled.

**Pros:**
- Temporal decoupling and event replay.
- Outbox removes the dual-write race — events survive broker downtime in `outbox_events`.
- Durable enough to be the ecosystem's reliable-delivery backbone.

**Cons:**
- Operational complexity (KRaft/Zookeeper, topic management).
- Relay polling adds slight latency between capture and delivery.

---

### Option 2 — Synchronous REST between Grid and Core

**Pros:**
- Simplest; no broker.

**Cons:**
- Couples the request path to downstream availability.
- No replay; a down consumer means lost signal.

---

### Option 3 — Cloud queue (SQS / EventBridge)

**Pros:**
- Managed, less to operate in cloud-only deployments.

**Cons:**
- Cloud lock-in; breaks the local standalone story.

---

## Decision

**We choose Option 1 — Kafka with the transactional outbox pattern.**

Kafka provides the temporal decoupling and replay the learning loop needs, and the outbox pattern makes publishing reliable in the face of the dual-write problem. Crucially, consumers are optional: Core's feedback consumer is `@ConditionalOnProperty` with no `matchIfMissing`, so Core boots standalone without Kafka or Grid present, and Docker Compose enables it only because the broker ships in the stack. Cloud queues remain a viable alternative for cloud-only deployments.

---

## Consequences

### Positive
- Request path never blocks on async reasoning; events are durable and replayable.
- The outbox guarantees at-least-once delivery without a distributed transaction.

### Negative / Trade-offs
- Kafka is another stateful service to operate.
- Relay polling introduces a small capture-to-delivery delay.

### Neutral / Notable
- The optional-consumer pattern is the ecosystem norm for cross-platform integration (mirrors the optional embedding service and the optional Grid seam) — presence of a peer is never required to run.

---

## Implementation Notes

- Outbox: `ApiCall` + `outbox_events` in one transaction; a relay thread publishes and marks published.
- Core consumer gated by `aether.core.feedback.enabled` (no `matchIfMissing`).
- Source: Grid decision log `D-004`, `D-008`; Core decision log `ADR-012`.

---

## Review

| Reviewer | Role | Decision | Date |
|---|---|---|---|
| | Architect | Approve / Reject | |
| | AI Engineer | Approve / Reject | |
| | Maintainer | Approve / Reject | |
