# Long-Term Ecosystem Roadmap

> This roadmap describes the five-phase arc from the current foundation to collective intelligence.
> Each phase delivers independently valuable capabilities that also advance the long-term vision.
> For repo-specific roadmaps, see the `docs/roadmap.md` in each repository.

---

## Phase 1 — Foundation (Active)

**Goal:** Establish the cognitive engine, agent runtime, methodology framework, and ecosystem standards. Every subsequent phase depends on what is built here.

**What ships:**
- `aether` — ecosystem hub with full standards, architecture docs, and philosophy. The canonical reference for every team building on the platform.
- `aether-iel` — Intelligence Engineering Lifecycle methodology. The 12-phase framework, IMM maturity model, governance controls, and artifact templates that every subsequent component is built and governed with.
- `aether-core` — Personal cognitive engine (Phases 0–6 complete). Episodic, semantic, procedural, and emotional memory with pgvector. Cognitive sessions, GDPR erasure, Kafka feedback loop, memory decay scheduler, Kubernetes deployment.
- `aether-grid` — Distributed agent governance mesh (16 phases complete). 19 specialist agents, Spring Cloud Gateway proxy, policy engine, workflow runtime, Kafka event bus, multi-tenancy, full Kubernetes/Helm deployment.

**Status:** All Phase 1 repositories are production-ready. The foundation is in place.

---

## Phase 2 — Memory & Knowledge Platform

**Goal:** Extract memory and knowledge as standalone, reusable platform services that any domain application can consume — without building on top of AetherCore directly.

**What ships:**

### `aether-memory`
A standalone memory platform service extracted and generalized from AetherCore's memory engine. Provides:
- Multi-type memory storage (EPISODIC, SEMANTIC, PROCEDURAL, EMOTIONAL) as a service
- Shared memory graph — memories that belong to a team, a domain, or an organization (not just an individual)
- Memory federation API — multiple Core instances can query shared team memories
- Configurable decay and reinforcement policies per memory type and per tenant

### `aether-vault`
A knowledge persistence and retrieval platform:
- Document indexing from enterprise sources (S3, SharePoint, Confluence, Google Drive)
- Chunking, embedding, and vector storage for RAG pipelines
- Knowledge graph — named entities and semantic relationships extracted from indexed documents
- Retrieval API: semantic search, keyword search, hybrid ranking
- Knowledge freshness tracking — re-index on document change detection

**Entry criteria:** AetherCore Phase 7 (Knowledge Graph) must be complete before `aether-vault` design begins, so that the knowledge graph schema is informed by real usage patterns.

---

## Phase 3 — Enterprise Platform

**Goal:** Make Aether deployable as a managed enterprise platform — with workflow orchestration, compliance tooling, and multi-organization governance.

**What ships:**

### `aether-flow`
A workflow orchestration service for AI-native processes:
- BPMN-based workflow definition for human-AI hybrid processes
- Human approval steps with configurable SLAs and escalation paths
- Long-running workflow state persistence (survives service restarts)
- Multi-agent collaboration patterns: sequential, parallel, debate/consensus
- Integration with AetherGrid's approval queue (DEFER decisions routed to Flow)

### `aether-enterprise`
Enterprise deployment and governance layer:
- Multi-organization tenancy — organizations with sub-tenants and hierarchical policy inheritance
- Compliance dashboard — GDPR consent status, erasure request tracking, audit log review UI
- Governance dashboards — agent decision distribution, confidence trends, human review queue SLA tracking
- Policy management UI — create, test, and activate SpEL policies without touching code
- Agent performance analytics — which agents are helping, which are blocking unnecessarily, which are drifting

**Entry criteria:** Phase 3 requires at least one production domain application (from Phase 1) to provide real usage data that informs the enterprise feature priorities.

---

## Phase 4 — Personal Intelligence & Developer Platform

**Goal:** Bring intelligence to individual developers and end users through purpose-built applications, and lower the barrier to building new agents.

**What ships:**

### `aether-mind`
A personal AI companion for individuals:
- Long-term memory of the user's goals, projects, preferences, and knowledge
- Coaching mode — the system reflects on patterns in the user's memory and offers observations
- Personal knowledge management — ingest notes, documents, and conversations into the personal memory store
- Personal automation — the user can delegate recurring tasks to agents that have full context about them
- Privacy-first: data stays local (Ollama) unless the user opts into cloud

### `aether-forge`
A developer platform for the Aether ecosystem:
- Agent templates — starting points for common agent patterns (classifier, retriever, synthesiser, governance)
- Skill marketplace — shareable agent capabilities that can be installed into any Grid deployment
- MCP development kit — tools for building and testing MCP tools locally
- Workflow designer — visual UI for composing agent pipelines without code
- Local testing harness — spin up a complete Aether stack (Core + Grid + pgvector + Ollama) with a single command for development

---

## Phase 5 — Collective Intelligence

**Goal:** Enable multiple Aether instances to federate — sharing knowledge, learning collectively, and reasoning across organizational boundaries.

**What ships:**

### `aether-mesh`
A federated intelligence network:
- Instance-to-instance communication protocol — Aether instances discover each other and negotiate knowledge sharing agreements
- Shared semantic memory — memories that multiple instances contribute to and draw from, with provenance tracking
- Collective learning — agent feedback signals from multiple instances flow into a shared calibration model
- Distributed reasoning — a question can be routed to the instance that has the most relevant expertise
- Privacy-preserving federation — knowledge is shared at the embedding level, not the raw content level, unless explicit consent is given

### Domain Ecosystem
By Phase 5, multiple domain applications exist:
- `aether-claims` — end-to-end insurance claims intelligence (intake → validation → fraud detection → assessment → payment)
- `aether-health` — healthcare clinical support intelligence
- `aether-support` — enterprise customer support intelligence

These domain applications share the Phase 1–4 platform infrastructure and contribute their domain-specific agent pipelines and knowledge back to the ecosystem.

---

## Dependency Graph

```
Phase 5: aether-mesh, domain-ecosystem
    ↑ depends on
Phase 4: aether-mind, aether-forge
    ↑ depends on
Phase 3: aether-flow, aether-enterprise
    ↑ depends on
Phase 2: aether-memory, aether-vault
    ↑ depends on
Phase 1: aether, aether-iel, aether-core, aether-grid
```

No phase can begin until the phase beneath it has reached sufficient maturity. The AIEL maturity assessment gates each transition.
