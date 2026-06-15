# Repository Catalog

> Every Aether component lives in its own repository with a bounded context, independent lifecycle, and clean API contract.
> The naming convention is `aether-{component}`. Each repo ships with EEIK bootstrap (CLAUDE.md, `.claude/memory/`, `.claude/agents/`).

---

## Active Repositories (Phase 1)

| Repository | GitHub | Role | Status |
|---|---|---|---|
| `aether` | [suplab/aether](https://github.com/suplab/aether) | Ecosystem hub — philosophy, standards, ADRs, architecture docs | ✅ Active |
| `aether-iel` | [suplab/aether-iel](https://github.com/suplab/aether-iel) | Intelligence Engineering Lifecycle methodology framework | ✅ Active |
| `aether-core` | [suplab/aether-core](https://github.com/suplab/aether-core) | Personal cognitive engine — memory, sessions, GDPR, decay | ✅ Phases 0–6 |
| `aether-grid` | [suplab/aether-grid](https://github.com/suplab/aether-grid) | Distributed agent governance mesh | ✅ 16 Phases |

---

## Planned Repositories

| Repository | Phase | Purpose | Entry Criteria |
|---|---|---|---|
| `aether-memory` | Phase 2 | Standalone reusable memory platform — shared team memory, federation | aether-core Phase 7 complete |
| `aether-vault` | Phase 2 | Knowledge persistence, document indexing, RAG pipelines | aether-memory scaffold |
| `aether-flow` | Phase 3 | Workflow orchestration for human-AI hybrid processes | Phase 2 complete |
| `aether-enterprise` | Phase 3 | Multi-org tenancy, compliance dashboard, policy management UI | aether-flow scaffold |
| `aether-mind` | Phase 4 | Personal AI companion — long-term memory, coaching, automation | Phase 3 complete |
| `aether-forge` | Phase 4 | Developer platform — agent templates, skill marketplace, testing harness | Phase 3 complete |
| `aether-mesh` | Phase 5 | Federated intelligence network — knowledge sharing across instances | Phase 4 complete |
| `aether-claims` | Future | Insurance domain application — claims intelligence end-to-end | aether-mesh or standalone |
| `aether-health` | Future | Healthcare domain application — clinical support intelligence | aether-mesh or standalone |
| `aether-support` | Future | Enterprise customer support intelligence | aether-mesh or standalone |

---

## Dependency Direction

The dependency graph is strict and one-directional. Higher layers call lower layers; lower layers never call higher layers.

```
Domain Applications (aether-claims, aether-health, aether-support)
          ↓
Platform Layer (aether-memory, aether-vault, aether-flow, aether-enterprise)
          ↓
Runtime Layer (aether-grid)
          ↓
Cognitive Layer (aether-core)
          ↓
Foundation (aether, aether-iel)
```

**Permitted:** AetherGrid calls AetherCore's REST API. AetherGrid publishes feedback events to Kafka that AetherCore consumes.
**Forbidden:** AetherCore calling AetherGrid. Any circular dependencies. Any direct database sharing between repositories.

---

## Repository Conventions

Every Aether repository must conform to the [Repository Guidelines standard](../standards/architecture/repository-guidelines.md). The mandatory files are:

```
{repo}/
├── CLAUDE.md                   ← EEIK project brief
├── README.md                   ← Entry point with ecosystem navigation table
├── aether.manifest.yaml        ← EEIK manifest (name, type, layer, consumes, consumed-by)
├── docs/
│   ├── index.html              ← Visual overview matching ecosystem design
│   ├── architecture.md         ← Architecture documentation
│   ├── roadmap.md              ← Development roadmap
│   └── progress.md             ← Current sprint progress
├── .claude/
│   ├── memory/                 ← 5+ memory files (project-context, glossary, patterns, decisions, session-log)
│   └── agents/                 ← Specialist agents for the domain
└── .github/
    └── workflows/
        ├── ci.yml              ← Build + test + quality gate
        └── docker-build.yml    ← Multi-arch Docker build + GHCR push (if applicable)
```

Repositories without runtime code (aether, aether-iel) omit the Docker workflow and runtime-specific files.
