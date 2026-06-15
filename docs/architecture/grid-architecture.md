# AetherGrid — Architecture

> **Repo:** [`suplab/aether-grid`](https://github.com/suplab/aether-grid)
> **Role:** Distributed Agent Governance Mesh — the nervous system of the Aether ecosystem
> **IMM Level:** Targets L5 — Distributed Intelligence
> **Status:** 16 phases complete, production-ready

---

## Purpose

AetherGrid is the distributed runtime that sits in front of every API in the enterprise. It is not a traditional API gateway — it is an intelligent governance mesh. Every request that passes through Grid is evaluated by a pipeline of specialist agents, each contributing a decision about whether to allow, rate-limit, alert on, or block the request.

If AetherCore is the brain — persistent memory, identity, personal context — then AetherGrid is the nervous system: the real-time processing layer that acts on information, enforces policy, coordinates agents, and feeds experience back to memory.

---

## Architecture Overview

```
                     User / Tenant
                           │
           ┌───────────────▼────────────────┐
           │  aether-proxy  (port 8080)      │
           │  Spring Cloud Gateway           │
           │  Custom AgentFilterChain        │
           └───────────────┬────────────────┘
                           │
           ┌───────────────▼────────────────┐
           │  aether-admin  (port 8081)      │
           │  Decision API · Policy API      │
           │  Agent Registry · Audit Trail   │
           └───────────────┬────────────────┘
                           │
          ┌────────────────┼──────────────────┐
          │                │                   │
    ┌─────▼─────┐   ┌──────▼──────┐   ┌───────▼───────┐
    │ AetherCore│   │  Kafka Bus   │   │  PostgreSQL   │
    │ (port 8082)│   │  Event mesh  │   │  + pgvector   │
    └─────┬─────┘   └──────┬───────┘   └───────────────┘
          │                │
          └────────────────┘
              Feedback loop
```

---

## Major Components

### Agent Runtime

The Agent Runtime is responsible for the lifecycle of every agent decision. When a request arrives at the proxy, the `AgentFilterChain` invokes each registered agent in sequence. Each agent implements the `Agent` interface:

```java
public interface Agent {
    String name();
    List<String> capabilities();
    AgentOutput execute(AgentInput input);
}
```

Every `AgentOutput` carries a `decision` (BLOCK, RATE_LIMIT, ALERT, SUGGEST, DEFER, ALLOW), a `confidence` score (0.0–1.0), a `reason` string, and an `autoEnforced` boolean. If `confidence < 0.8`, then `autoEnforced = false` — the decision surfaces to human review regardless of the decision type.

The confidence gate is enforced in the `AgentOutput` builder, not in individual agents. No agent can self-certify as auto-enforceable at low confidence.

### MCP Runtime

The Model Context Protocol Runtime manages the tool ecosystem. Agents that need to call external systems — fetch a document, query a database, look up a user record — do so through registered MCP tools. The MCP Runtime handles tool registration, capability discovery, input validation, and execution logging.

Tool access is declared per-agent in the Agent Contract (AIEL Phase 5 artifact). Agents cannot invoke tools that are not in their declared access list.

### Policy Engine

The Policy Engine evaluates Spring Expression Language (SpEL) policy rules against each agent decision. Policies are stored in the database and can be updated without redeployment. A policy can: override a decision, add conditions to auto-enforcement, require specific confidence thresholds for specific decision types, or route decisions to specific human reviewer queues.

Policy evaluation is synchronous — it happens within the request path, not asynchronously. This ensures that policy changes take effect immediately on the next request.

### Workflow Runtime

For decisions that require human approval or multi-step processing, Grid routes them to the Workflow Runtime. This provides:

- Human approval steps with configurable SLAs
- Multi-agent debate patterns (two agents evaluate a decision independently; a supervisor resolves disagreement)
- Long-running processes that span multiple API calls
- Timeout and escalation handling

Workflow state is persisted in PostgreSQL. Workflows survive Grid restarts.

### Event Bus

Kafka is the backbone of Grid's event architecture. Key topics:

| Topic | Purpose |
|---|---|
| `aether.decisions` | All agent decisions, consumed by the audit service |
| `aether.core.feedback` | Feedback signals (CORRECT/INCORRECT/PARTIALLY_CORRECT) published to AetherCore |
| `aether.alerts` | ALERT decisions routed to notification consumers |
| `aether.approvals` | DEFER decisions pending human review |

All Kafka writes use the transactional outbox pattern — the event is written to a database table within the same transaction as the decision, and a separate process reliably publishes to Kafka. This prevents the dual-write problem.

### Agent Registry

The Agent Registry is the source of truth for which agents exist, what capabilities they declare, and which tenants they are active for. Agents register on startup. The registry supports:

- Dynamic agent discovery — new agents are picked up without redeployment
- Capability-based routing — requests can be routed to agents that declare specific capabilities
- Tenant-scoped agent activation — different tenants can have different agent pipelines
- Version management — multiple versions of the same agent can coexist during rolling upgrades

### Observability

Every agent decision is traced end-to-end with OpenTelemetry. Trace context is propagated through HTTP headers and Kafka message headers so that a full trace of a request — from proxy ingress through every agent, through AetherCore, and into any downstream system — can be reconstructed in Jaeger or any OTLP-compatible backend.

Micrometer metrics expose decision counts, latency histograms, confidence distributions, and error rates on `/actuator/prometheus`. Grafana dashboards alert on: error rate > 5%, p99 latency > 2s, low-confidence surge, and circuit breaker trips.

---

## The Specialist Agents

AetherGrid ships with 19 specialist agents covering the full lifecycle of enterprise API governance:

| Category | Agents |
|---|---|
| Context | `AetherCoreBridgeAgent` — fetches personal context from AetherCore before every decision |
| Classification | `IntentClassifierAgent` — classifies request intent using LLM |
| Risk | `RiskScorerAgent`, `AnomalyDetectorAgent`, `FraudDetectorAgent` |
| Policy | `PolicyEvaluatorAgent`, `ComplianceCheckerAgent` |
| Rate & Load | `RateLimiterAgent`, `LoadBalancerAgent` |
| Context | `ContextEnricherAgent`, `TemporalPredictorAgent` |
| Memory | `MemoryRetrievalAgent`, `KnowledgeGraphAgent` |
| Self-Improvement | `SelfDebugAgent`, `FeedbackAgent`, `PerformanceOptimiserAgent` |
| Communication | `ResponseFormatterAgent`, `ExplanationGeneratorAgent` |
| Governance | `AuditAgent` — logs every decision to the immutable audit trail |

---

## Integration with AetherCore

Grid calls Core. Core does not call Grid. This direction is a hard architectural rule.

Before every agent decision, `AetherCoreBridgeAgent` calls:
```
GET /api/v1/personal-context/{tenantId}/{userId}
```

The response is attached to the `AgentInput` for all downstream agents to use. It contains the user's recent memories, active session state, and a synthesized context summary.

After a decision is evaluated as CORRECT or INCORRECT (through user feedback, outcome monitoring, or supervisor review), `FeedbackAgent` publishes a `FeedbackEvent` to the `aether.core.feedback` Kafka topic. Core's `GridFeedbackConsumer` processes this and updates memory strength accordingly.

This feedback loop is what makes the ecosystem self-improving. Grid agents get better data from Core over time as memories accumulate and strengthen. Core gets richer feedback from Grid as agents evaluate their decisions.

---

## Multi-Tenancy

Every concept in Grid is tenant-scoped. Tenant isolation is enforced at:

- The database layer: `tenant_id` in every `WHERE` clause
- The agent layer: agent pipelines are activated per-tenant in the registry
- The policy layer: policies belong to a tenant and cannot cross boundaries
- The event layer: Kafka messages carry `tenant_id` in headers
- The audit layer: audit log entries are scoped to tenant, user, and trace ID

A single Grid deployment serves multiple tenants concurrently with complete data isolation between them.

---

## Execution Flow

A request arrives and travels through the following stages:

```
1. Ingress          — Spring Cloud Gateway receives the request
2. Tenant resolve   — TenantResolver extracts tenant context from JWT / API key
3. Context load     — AetherCoreBridgeAgent fetches personal context
4. Agent pipeline   — each registered agent evaluates the request in sequence
5. Policy eval      — PolicyEvaluatorAgent applies tenant-scoped SpEL rules
6. Decision merge   — AgentFilterChain merges all agent outputs into a final decision
7. Confidence gate  — if confidence < 0.8, autoEnforced = false → human review queue
8. Enforcement      — the decision is enforced: allow, rate-limit, block, or defer
9. Audit write      — AuditAgent writes the full decision to the immutable audit log
10. Feedback        — outcome signals published to Kafka for Core to consume
```

Total p99 target: under 2 seconds end-to-end including AetherCore lookup and LLM inference.
