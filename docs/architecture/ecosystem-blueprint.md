# Aether Ecosystem Blueprint

> This document is the canonical architectural reference for the full Aether ecosystem.
> It describes the ecosystem's structure, the role of each component, their relationships, and the principles that govern how they fit together.

---

## Guiding Metaphor

The Aether ecosystem is modelled on the human cognitive system:

- **AetherCore** is the brain — the seat of memory, identity, personal context, and reasoning about the self
- **AetherGrid** is the nervous system — the distributed runtime that processes signals from the world, coordinates agents, and acts on behalf of the organism
- **Platform components** (AetherMemory, AetherVault, AetherFlow) are specialized organs — each optimized for one function, all connected through the nervous system
- **Domain applications** (AetherClaims, AetherHealth) are the organism's behaviors — specific ways the cognitive system engages with the world

This is not just a metaphor. It encodes a real architectural principle: each component has a bounded, exclusive responsibility. No two components should own the same capability. The brain does not do what the nervous system does. Organs do not replicate each other.

---

## The Full Ecosystem

```
┌─────────────────────────────────────────────────────────────────────┐
│  aether  ·  aether-iel                  Foundation & Methodology    │
└──────────────────────────┬──────────────────────────────────────────┘
                           │ governs
┌──────────────────────────▼──────────────────────────────────────────┐
│  aether-core                               Personal Cognitive Engine │
│  Memory · Sessions · GDPR · Context API                             │
└──────────────────────────┬──────────────────────────────────────────┘
                           │ personal context
┌──────────────────────────▼──────────────────────────────────────────┐
│  aether-grid                              Agent Governance Mesh      │
│  Proxy · Agents · Policy · Kafka · Audit                            │
└──────┬───────────────────┬──────────────────────┬───────────────────┘
       │                   │                       │
┌──────▼───────┐  ┌────────▼────────┐  ┌──────────▼──────────────────┐
│ aether-memory│  │  aether-vault   │  │  aether-flow                │
│ Shared Memory│  │  Knowledge+RAG  │  │  Workflow Orchestration      │
└──────────────┘  └─────────────────┘  └─────────────────────────────┘
       │                   │                       │
┌──────▼───────────────────▼───────────────────────▼─────────────────┐
│  aether-enterprise                     Enterprise Governance Layer   │
│  Multi-org · Compliance · Dashboards · Policy Management UI         │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────────┐
│  Domain Applications                                                 │
│  aether-claims · aether-health · aether-support                     │
│  aether-mind (personal) · aether-forge (developer)                  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Component Descriptions

### aether — Ecosystem Hub

The `aether` repository has no runtime code. It is the ecosystem's source of truth: vision, philosophy, architecture documentation, standards that every other component must follow, Architecture Decision Records, and the roadmap. Every team building on the Aether platform starts here before writing any code.

**What lives here:** Vision, principles, ecosystem blueprint (this document), per-component architecture docs, AIEL integration, standards (Java, agent design, security, observability, repository structure), ADRs, templates (ADR, proposal, architecture), domain briefings, research references.

**What does not live here:** Runtime code, configuration, deployment manifests.

---

### aether-iel — Intelligence Engineering Lifecycle

The AIEL methodology framework. Defines how intelligence systems are designed, built, governed, and evolved. Every Aether component was built using AIEL — and every future component must be too.

**What lives here:** 12-phase lifecycle documentation, 5-level Intelligence Maturity Model, artifact templates (Intelligence Brief, Cognitive Design, Agent Contract, Evaluation Report), governance controls standard, evaluation criteria standard, EEIK bootstrap files.

**Why it is a separate repository:** The methodology must be usable independently of any specific platform component. A team building an AI system on a completely different stack can adopt AIEL. Keeping it separate prevents tight coupling to any one implementation.

---

### aether-core — Personal Cognitive Engine

AetherCore is the persistent cognitive engine for individual users. It maintains a continuously evolving model of a person's knowledge, experiences, preferences, and emotional context — and makes that model available to any agent via a REST API.

**Current state:** Phases 0–6 complete. Production-ready. Deployed on Kubernetes.

**Module structure:**
- `core-domain` — domain types (`PersonalMemory`, `CognitiveSession`, `GdprConsent`) and port interfaces. Zero Spring dependency.
- `core-memory` — pgvector adapter, embedding service (Ollama all-MiniLM-L6-v2, 384-dim), session adapter, GDPR adapter, decay scheduler, Kafka feedback consumer
- `core-api` — Spring Boot app, REST controllers, CoreApiConfig wiring, Flyway migrations V001–V004
- `core-infra` — Docker Compose, Kubernetes manifests (namespace, deployment, service, HPA), GitHub Actions CI/CD

**Primary integration contract:**
```
GET /api/v1/personal-context/{tenantId}/{userId}
→ PersonalContext { recentMemories, activeSession, contextSummary }
```

This endpoint is called by AetherGrid's `AetherCoreBridgeAgent` before every agent decision. It is the most critical API contract in the ecosystem — changes require coordinated cross-repository updates.

---

### aether-grid — Agent Governance Mesh

AetherGrid is the distributed runtime that governs every API interaction at enterprise scale. It sits as an intelligent proxy in front of the enterprise API estate, routing every request through a pipeline of specialist agents, evaluating policy, enforcing the confidence gate, and feeding outcomes back to AetherCore.

**Current state:** 16 phases complete. Production-ready. Deployed on Kubernetes.

**Module structure:**
- `aether-proxy` — Spring Cloud Gateway, AgentFilterChain, tenant resolution (port 8080)
- `aether-admin` — Decision API, agent registry, policy management, audit trail (port 8081)
- Shared: PostgreSQL (decisions, audit, policies), Kafka (events, feedback, alerts), AetherCore (personal context)

**19 specialist agents:** AetherCoreBridgeAgent, IntentClassifierAgent, RiskScorerAgent, AnomalyDetectorAgent, FraudDetectorAgent, PolicyEvaluatorAgent, ComplianceCheckerAgent, RateLimiterAgent, LoadBalancerAgent, ContextEnricherAgent, TemporalPredictorAgent, MemoryRetrievalAgent, KnowledgeGraphAgent, SelfDebugAgent, FeedbackAgent, PerformanceOptimiserAgent, ResponseFormatterAgent, ExplanationGeneratorAgent, AuditAgent.

---

### aether-memory — Shared Memory Platform (Phase 2)

Extracts and generalizes AetherCore's memory engine from personal to shared. Enables team memory, organizational memory, and cross-instance memory federation. While AetherCore stores memories for individuals, AetherMemory stores memories for groups.

**Planned capabilities:** Multi-user memory storage, shared memory graph, configurable decay policies per tenant, cross-instance federation API.

**Entry criteria:** AetherCore Phase 7 (Knowledge Graph) must be complete, to ensure the graph schema is informed by real production usage.

---

### aether-vault — Knowledge Platform (Phase 2)

A standalone knowledge persistence and retrieval service. Indexes documents from enterprise sources, builds a knowledge graph from extracted entities and relationships, and provides a RAG pipeline API that agents use to ground their responses in organizational knowledge.

**Planned capabilities:** Document ingestion (S3, SharePoint, Confluence, Google Drive), chunking and embedding, semantic and hybrid search, knowledge graph, freshness tracking and re-indexing.

**Distinction from AetherCore memory:** AetherCore stores personal, user-attributed memories. AetherVault stores organizational knowledge that belongs to the enterprise, not to any individual.

---

### aether-flow — Workflow Orchestration (Phase 3)

Handles long-running, human-AI hybrid processes that span multiple API calls, require human approval gates, or involve coordination between multiple agents over time. Integrates directly with AetherGrid — DEFER decisions from Grid route into Flow's approval queue.

**Planned capabilities:** BPMN-based workflow definition, human approval steps with SLAs and escalation, workflow state persistence, multi-agent debate patterns, timeout handling.

---

### aether-enterprise — Enterprise Layer (Phase 3)

The governance and compliance layer for multi-organization deployments. Provides tooling that operations and compliance teams need: dashboards, policy management, audit review, and multi-organization tenancy hierarchy.

**Planned capabilities:** Organization hierarchy with sub-tenants and policy inheritance, GDPR compliance dashboard, decision analytics, human review queue SLA tracking, policy management UI.

---

### aether-forge — Developer Platform (Phase 4)

The platform for developers building on the Aether ecosystem. Lowers the barrier to creating new agents, deploying new capabilities, and testing intelligence systems locally.

**Planned capabilities:** Agent templates, skill marketplace, local dev stack (one command to run Core + Grid + pgvector + Ollama), testing harness with mock AetherCore, MCP development kit.

---

### aether-mind — Personal Companion (Phase 4)

A personal AI companion for individuals. Uses AetherCore for memory storage and builds a personal intelligence layer on top — coaching, knowledge management, personal automation, and skill tracking.

**Planned capabilities:** Long-term personal memory, coaching mode with pattern reflection, personal knowledge management, personal automation agents, privacy-first local deployment.

---

### aether-mesh — Federated Intelligence (Phase 5)

Enables multiple Aether instances to connect, share knowledge, and reason collectively. The technical and ethical challenges of federation — privacy-preserving knowledge sharing, consensus without central authority, emergent capability — are the Phase 5 frontier.

**Planned capabilities:** Instance discovery and pairing, embedding-level knowledge sharing, collective feedback calibration, distributed reasoning routing, privacy-preserving federation.

---

## Dependency Direction

The dependency graph is strictly hierarchical. Higher layers call lower layers. Lower layers never call higher layers. Circular dependencies are forbidden.

```
Domain Applications
      calls ↓
Enterprise Layer
      calls ↓
Platform (Memory, Vault, Flow)
      calls ↓
Agent Mesh (Grid)
      calls ↓
Cognitive Engine (Core)
      governed by ↓
Foundation (aether, aether-iel)
```

**The one exception:** Grid publishes feedback events to Kafka that Core consumes. This is not a direct call — it is an event flowing through a topic. Core does not know about Grid; it only knows about the Kafka topic. This is intentional and acceptable under the event-driven principle.

---

## EEIK Bootstrap and the Methodology Layer

EEIK (Enterprise Engineering Intelligence Kit) is the bootstrap framework that every Aether repository is initialized with: `CLAUDE.md`, `.claude/memory/`, `.claude/agents/`, `.claude/commands/`.

EEIK is not a runtime component. It is the development-time scaffolding that gives Claude Code agents the context they need to contribute intelligently to each repository. It is distinct from AIEL (the methodology) — EEIK is how you set up a repository, AIEL is how you design and govern what goes in it.

```
EEIK Bootstrap            →  How you set up the repository
AIEL Methodology          →  How you design what goes in it
aether standards          →  What rules the code must follow
aether-[component]        →  What gets built
```

Every new Aether repository begins with EEIK bootstrap as its first commit, then follows AIEL from Phase 1.

---

## Cross-Cutting Concerns

These concerns apply to every component in the ecosystem. They are not the responsibility of any single component — they are ecosystem-wide constraints enforced through the standards defined in `suplab/aether`.

| Concern | Standard | Enforcement |
|---|---|---|
| Agent confidence gate | confidence < 0.8 → `autoEnforced = false` | Code — not configuration |
| GDPR erasure | Hard-delete cascade on verified request | `GdprController` pattern |
| Multi-tenancy | `tenant_id` in every query | Code review gate |
| PII in logs | Redaction before write | Regex pattern library |
| Conventional commits | `type(scope): description` | CI quality gate |
| Coverage | JaCoCo ≥ 80% line coverage | CI quality gate |
| CVE scanning | CVSS ≥ 9.0 fails build | OWASP Dependency Check in CI |
| Container security | Non-root uid 1000, read-only root filesystem | Dockerfile standard |
| Observability | Micrometer + OTel + SLF4J | Spring Boot autoconfigure |
