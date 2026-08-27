# Æ Aether

> A distributed cognitive computing ecosystem for building persistent, memory-driven, agentic intelligence systems.

**[Visual Overview](docs/index.html)** · **[Interactive Portal](docs/portal/index.html)** · [Vision](docs/vision/vision.md) · [Blueprint](docs/architecture/ecosystem-blueprint.md) · [Standards](standards/) · [Roadmap](docs/roadmaps/long-term.md) · [Improvement Backlog](docs/roadmaps/ecosystem-improvements.md)

---

## Mission

Create a platform where intelligence is **persistent**, **explainable**, **collaborative**, **enterprise-ready**, and **domain-adaptable**.

We are not engineering software. We are engineering intelligence.

---

## Core Philosophy

Current AI systems are stateless and transactional. They answer questions but do not genuinely remember, evolve, or collaborate. Aether exists to change that.

| Principle | Meaning |
|---|---|
| **Memory First** | Episodic, semantic, procedural, and emotional memory are core primitives |
| **Model Agnostic** | All LLM calls go through `LlmClient` — Ollama, Groq, Anthropic, or any provider |
| **Event Driven** | Every meaningful action produces an event; agents react and learn |
| **Human in the Loop** | Confidence < 0.8 → never auto-block → human review required |
| **Explainability** | Every decision has rationale; every memory has provenance |
| **Composability** | Capabilities are modular and independently deployable |
| **Enterprise Readiness** | Multi-tenancy, GDPR, audit trails, and governance are built in |
| **Evolvability** | Agents learn from feedback; memories reinforce and decay |

---

## Ecosystem

```text
Aether                              (this repository — architecture & standards)
│
├── AetherCore          (suplab/aether-core)    ← Personal Cognition — Brain
│     Phases 0–6 ✅  (GDPR erasure, retention, portability; Kubernetes + Helm)
│
├── AetherGrid          (suplab/aether-grid)    ← Distributed Runtime — Nervous System
│     Phases 0–17 ✅  (roadmap fully delivered)
│
├── Aether-IEL          (suplab/aether-iel)     ← Intelligence Engineering Lifecycle
│
├── AetherMemory        (suplab/aether-memory)  ← Memory Platform
│     Phases 1–4 ✅  (federation hardening, governance + policy audit, Kubernetes + Helm)
├── AetherVault         (suplab/aether-vault)   ← Knowledge Platform
│     Phases 1–4 ✅  (knowledge graph, freshness + auto-reingest, GDPR erasure, Kubernetes + Helm)
├── AetherFlow          (suplab/aether-flow)    ← Workflow Platform
│     Phases 1–4 ✅  (parallel fork/join, SLA governance, Grid integration, Kubernetes + Helm)
├── AetherEnterprise    (planned — Phase 3)     ← Enterprise Layer
├── AetherMind          (planned — Phase 4)     ← Personal AI Companion
├── AetherForge         (planned — Phase 4)     ← Developer Platform
├── AetherMesh          (vision — Phase 5)      ← Federated Intelligence
└── Domain Products
      └── AetherClaims  (vision)                ← Insurance Platform
```

---

## Active Repositories

| Repository | Purpose | Status |
|---|---|---|
| [`aether`](.) | Ecosystem index — vision, architecture, standards, governance, ADRs | Active |
| [`aether-core`](https://github.com/suplab/aether-core) | Personal cognitive engine — memory, sessions, emotional context | Phases 0–6 ✅ · GDPR erasure + legal holds + retention + portability · Kubernetes + Helm |
| [`aether-grid`](https://github.com/suplab/aether-grid) | Distributed agent mesh — enterprise API governance | Phases 0–17 ✅ (roadmap fully delivered) |
| [`aether-iel`](https://github.com/suplab/aether-iel) | Intelligence Engineering Lifecycle methodology framework | Active |
| [`aether-memory`](https://github.com/suplab/aether-memory) | Shared team/org memory platform — federation, per-tenant policy | Phases 1–4 ✅ · federation hardening (audit + rate-limit + redaction + peer auth + distributed limiter) · governance (retention purge + GDPR erasure + export + policy-change audit) · Kubernetes + Helm |
| [`aether-vault`](https://github.com/suplab/aether-vault) | Knowledge platform — document indexing, vector search, RAG, knowledge graph | Phases 1–4 ✅ · knowledge-graph extraction + entity-aware RAG · freshness policy + auto-reingestion + GDPR collection erasure · Kubernetes + Helm |
| [`aether-flow`](https://github.com/suplab/aether-flow) | Workflow platform — process orchestration, human approval, SLA escalation, Grid DEFER intake | Phases 1–4 ✅ · parallel AND fork/join + SLA governance + business-hours calendars · Grid DEFER closed loop + agent steps + GDPR erasure · Kubernetes + Helm |

---

## Interactive Portal

An **offline, self-contained interactive portal** lives at [`docs/portal/`](docs/portal/index.html) — open
`docs/portal/index.html` in any browser (no server, no network). Each Aether platform has a faithful, hands-on
simulation of its **real core loop**:

| Page | Simulates |
|---|---|
| [**Ecosystem Dashboard**](docs/portal/dashboard.html) | **Mission-control umbrella view** — every repo as a node, animated cross-service data flows, and live KPI tiles + an event stream driven by a simulated workload |
| [Core](docs/portal/core.html) | Personal memory reinforcement on recall, decay when idle, and personal-context assembly |
| [Grid](docs/portal/grid.html) | The confidence gate — adjust confidence and watch a decision auto-enforce or DEFER to a human |
| [Memory](docs/portal/memory.html) | Shared team reinforcement by distinct contributor, and the privacy-preserving federation projection |
| [Vault](docs/portal/vault.html) | The RAG pipeline — embed, vector-search within one collection, assemble a length-bounded context |
| [Flow](docs/portal/flow.html) | A workflow state machine, a human approval gate with an SLA, and the escalation sweep |
| [How it fits](docs/portal/architecture.html) | Dependency direction, the DEFER seam, the feedback loop, AIEL, and the HALO runtime cross-link |

The simulations are illustrative teaching models (real thresholds and domain rules; no LLM, database, or network).
The portal is **Aether-scoped** — the [HALO](https://github.com/doubts-suplab/agent-harness) runtime Grid consumes is
cross-linked, not embedded, since it is a separate vendor-neutral peer platform.

---

## What This Repository Contains

This repository has **no runtime code**. It is the canonical source of truth for the entire ecosystem.

```
aether/
├── docs/
│   ├── index.html                    ← Visual ecosystem overview
│   ├── portal/                       ← Offline interactive portal (per-platform simulations)
│   ├── vision/                       ← Why Aether exists, strategic goals
│   ├── architecture/                 ← System context, blueprints, component designs
│   ├── domains/                      ← Healthcare, insurance, developer productivity
│   ├── research/                     ← Cognitive architecture research
│   ├── roadmaps/                     ← Long-term delivery plan
│   └── adr/                          ← Architecture Decision Records
├── ecosystem/
│   ├── repositories.md               ← Repository catalog and dependencies
│   └── capabilities.md               ← Capability map by owner
├── standards/
│   ├── development/java-guidelines.md
│   ├── ai/agent-design.md
│   ├── security/compliance.md
│   ├── operations/observability.md
│   └── architecture/repository-guidelines.md
├── governance/
│   └── contribution-model.md
├── references/
│   ├── papers.md                     ← Research papers
│   └── projects.md                   ← Related open-source projects
└── templates/
    ├── adr-template.md
    ├── architecture-template.md
    └── proposal-template.md
```

---

## Architecture Overview

```text
                  AETHER COGNITIVE FABRIC
                          │
       ┌──────────────────┼──────────────────┐
       │                  │                  │
  AetherCore         AetherGrid         Domain Products
       │                  │                  │
  Intelligence          Runtime          Applications
   (The Brain)     (Nervous System)     (Specialized Organs)
       │                  │
  ┌────┴────┐      ┌──────┴──────┐
  Memory    │      │   Agents    │
  Sessions  │      │   Policies  │
  Context   │      │   Events    │
  Identity  │      │   Gateway   │
  └─────────┘      └─────────────┘
```

**Dependency direction:**
```
Domain Applications → AetherGrid → AetherCore
```

---

## Technology Stack

All Aether runtime components share the same foundation:

| Layer | Technology |
|---|---|
| Language | Java 21 (`--enable-preview`) |
| Framework | Spring Boot 3.3.x (`jakarta.*`) |
| Messaging | Apache Kafka |
| Database | PostgreSQL 16 + pgvector |
| Cache | Redis 7 |
| Embedding | all-MiniLM-L6-v2 via Ollama (384-dim) |
| LLM Runtime | Ollama · Groq · Anthropic Claude |
| Resilience | Resilience4j |
| Observability | OpenTelemetry + Micrometer + Prometheus + Grafana |
| Migrations | Flyway |
| Build | Maven multi-module |
| Local Dev | Docker Compose |
| Production | Kubernetes + Helm |
| CI/CD | GitHub Actions (OIDC) |

---

## Publications

**AetherGrid: A Distributed Cognitive-Orchestration Framework Linking Sensory-Emotional Memory Encoding (AetherCore) with Autonomous Agent Governance**
[Preprint on Zenodo](https://doi.org/10.5281/zenodo.17477224) · doi:10.5281/zenodo.20394197