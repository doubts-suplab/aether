# Æ Aether

> A distributed cognitive computing ecosystem for building persistent, memory-driven, agentic intelligence systems.

**[Visual Overview](docs/index.html)** · [Vision](docs/vision/vision.md) · [Blueprint](docs/architecture/ecosystem-blueprint.md) · [Standards](standards/) · [Roadmap](docs/roadmaps/long-term.md)

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
│     Phases 0–1 ✅  Phase 2 🔄
│
├── AetherGrid          (suplab/aether-grid)    ← Distributed Runtime — Nervous System
│     Phases 0–16 ✅  Phase 17 🔄
│
├── Aether-IEL          (suplab/aether-iel)     ← Intelligence Engineering Lifecycle
│
├── AetherMemory        (planned — Phase 2)     ← Memory Platform
├── AetherVault         (planned — Phase 2)     ← Knowledge Platform
├── AetherFlow          (planned — Phase 3)     ← Workflow Platform
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
| [`aether-core`](https://github.com/suplab/aether-core) | Personal cognitive engine — memory, sessions, emotional context | Phase 2 🔄 |
| [`aether-grid`](https://github.com/suplab/aether-grid) | Distributed agent mesh — enterprise API governance | Phase 17 🔄 |
| [`aether-iel`](https://github.com/suplab/aether-iel) | Intelligence Engineering Lifecycle methodology framework | Active |

---

## What This Repository Contains

This repository has **no runtime code**. It is the canonical source of truth for the entire ecosystem.

```
aether/
├── docs/
│   ├── index.html                    ← Visual ecosystem overview
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
[Preprint on Zenodo](https://doi.org/10.5281/zenodo.20394197) · doi:10.5281/zenodo.20394197