# Capability Map

> Maps capabilities to the ecosystem components that own them.
> A capability is a coherent set of functions that belong together conceptually.
> No two components should own the same capability — ownership is exclusive.

---

## Cognition — Owner: AetherCore

The cognitive layer provides persistent, structured memory and assembled personal context. It is the only component that stores and retrieves personal memories. All other components that need personal context call AetherCore's REST API — they do not maintain their own memory stores.

| Capability | Description |
|---|---|
| Episodic Memory | Storage and retrieval of time-anchored personal experiences |
| Semantic Memory | Storage and retrieval of general facts and domain knowledge attributed to a user |
| Procedural Memory | Storage and retrieval of learned skills, preferences, and how-to knowledge |
| Emotional Memory | Storage and retrieval of affective states and their context |
| Memory Reinforcement | Automatic strength increase on access — memories that matter stay relevant |
| Memory Decay | Scheduled strength reduction for idle memories — the system naturally forgets what is no longer relevant |
| Cognitive Sessions | Multi-turn session management with turn summaries, emotional arc, and engagement tracking |
| Personal Context Assembly | Synthesis of memories + session state + context summary into a snapshot for agent consumption |
| GDPR Compliance | Consent management, right to access, right to erasure (Article 17 hard-delete) |

---

## Agent Execution — Owner: AetherGrid

The execution layer is where decisions happen in real time. Grid intercepts traffic, routes it through agent pipelines, evaluates policy, enforces the confidence gate, and feeds outcomes back to memory. It does not store personal memories — it consumes them from AetherCore.

| Capability | Description |
|---|---|
| API Governance | Intelligent proxy that applies agent-evaluated policy to every API request |
| Agent Orchestration | Sequential and parallel agent pipeline execution with configurable routing |
| Policy Evaluation | SpEL-based policy rules evaluated against agent decisions, updateable without redeployment |
| Confidence Gate | Structural enforcement of human-in-the-loop at confidence < 0.8 |
| Decision Audit | Immutable append-only log of every agent decision with full context |
| Multi-Tenancy | Complete tenant isolation at every layer — database, agent pipeline, policy, events |
| Feedback Loop | Publishing CORRECT/INCORRECT/PARTIALLY_CORRECT signals to AetherCore after decision outcomes are known |
| Observability | End-to-end OpenTelemetry tracing, Micrometer metrics, structured logging |
| Workflow | Long-running human-AI hybrid processes, approval queues, multi-agent debate |

---

## Knowledge — Owner: AetherVault (Phase 1 ✅ — ingestion & retrieval engine)

The knowledge layer manages structured, organizational knowledge — documents, facts, and relationships that belong to a team or organization rather than an individual. It is distinct from AetherCore's personal memory.

| Capability | Description |
|---|---|
| Document Indexing | Ingest, chunk, embed, and store documents from enterprise sources |
| Vector Search | Semantic retrieval of relevant document chunks for RAG pipelines |
| Knowledge Graph | Named entities and semantic relationships extracted from indexed documents |
| RAG Pipeline | Retrieval-Augmented Generation API for agent context enrichment |
| Knowledge Freshness | Change detection and re-indexing on document updates |

---

## Shared Memory — Owner: AetherMemory (scaffolded — Phase 0 ✅)

Team and organizational shared memory, extracted from AetherCore and generalized for multi-user and multi-instance scenarios.

| Capability | Description |
|---|---|
| Team Memory | Memories that belong to a group rather than an individual |
| Shared Reinforcement | Team access patterns influence shared memory strength |
| Memory Federation API | Cross-instance memory queries with privacy-preserving defaults |
| Configurable Policies | Per-tenant decay rates, retention periods, and access controls |

---

## Workflows — Owner: AetherFlow (scaffolded — Phase 0 ✅)

| Capability | Description |
|---|---|
| Process Orchestration | BPMN-based definition of multi-step human-AI workflows |
| Human Approval Steps | Configurable human review gates with SLAs and escalation |
| State Persistence | Workflow state survives service restarts |
| Integration with Grid | DEFER decisions from Grid route into Flow approval queues |

---

## Enterprise — Owner: AetherEnterprise (Phase 3)

| Capability | Description |
|---|---|
| Multi-Org Tenancy | Organizational hierarchy with sub-tenants and policy inheritance |
| Governance Dashboard | Audit log UI, decision analytics, human review queue management |
| Compliance Tooling | GDPR consent status, erasure request tracking, retention reporting |
| Policy Management | UI for creating, testing, and activating agent policies without code changes |

---

## Developer Experience — Owner: AetherForge (Phase 4)

| Capability | Description |
|---|---|
| Agent Templates | Starting points for common agent patterns with AIEL-compliant structure |
| Skill Marketplace | Shareable agent capabilities installable across Grid deployments |
| Local Dev Stack | Single-command spin-up of Core + Grid + pgvector + Ollama |
| Testing Harness | Unit and integration testing tools for agents with mock AetherCore |

---

## Capability Ownership Rules

1. Each capability is owned by exactly one component. If two components claim the same capability, the one lower in the dependency stack wins.
2. Components may consume capabilities from lower layers via port interfaces or REST APIs. They may not re-implement them.
3. Adding a capability to a component requires an Architecture Decision Record.
4. Removing a capability requires an RFC with a migration plan for all consumers.
