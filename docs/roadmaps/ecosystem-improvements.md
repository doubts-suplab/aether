# Æ Aether — Ecosystem Improvement Backlog

> A prioritized, feasibility-assessed backlog distilled from a structured external review of the whole
> ecosystem. This is a **living plan, not a delivery claim** — it tracks scope so independent repos
> evolve toward one coherent product rather than fragmenting. Repo-specific items also appear in each
> repository's own `docs/roadmap.md`, which links back here.
>
> Companion to [`long-term.md`](long-term.md) (the five-phase vision arc). This document is the
> cross-cutting *improvement* backlog beneath that arc.

---

## Guardrails (read first)

- **Licensing is unchanged — AGPL-3.0 across every repo.** The review's dual-/permissive-licensing
  suggestion is a **parked business decision for the owner**, explicitly *not* tracked as work here.
  Where relevant we will only *document AGPL implications more clearly*, never relicense.
- **Resist roadmap expansion.** Do not start new platforms (Mind / Forge / Mesh / Enterprise) until the
  current platforms have hardened interop and real production-usage data. Depth before breadth.
- **The core loop is the product.** Memory reinforcement → agent decision → human review → learning.
  Every item is justified by how much it hardens or exercises that loop.
- **Feasibility legend:** **S** small (docs / bounded) · **M** moderate (a feature + a design call +
  tests) · **L** large (new surface, distributed mechanism, or research-shaped) · **⧗** external
  (blocked on accounts/infra/credentials).

---

## Highest-impact next steps (the review's top 5)

In order. These are ecosystem-level and cut across repos.

| # | Initiative | Feasibility | Why first |
|---|---|---|---|
| 1 | **Shared foundations + umbrella deployment** — a parent POM / BOM, common libraries (embedding client, domain primitives, observability, GDPR/security helpers, event schemas), and an umbrella Helm chart / docker-compose for the full stack | **L** (net-new shared repo/module + migration) | Removes the biggest source of drift; every other initiative gets cheaper once the platforms share primitives. Note: an earlier decision kept the POMs **independent** on purpose — a BOM is the reconciling middle path (shared versions, still independently buildable). |
| 2 | **End-to-end integration tests + hardened cross-service contracts** — the Grid↔Core, Grid→Flow (DEFER), Memory federation, and Vault-RAG seams get versioned contracts, resilience (circuit breakers, retries, timeouts), and real E2E tests; plus a thin **Aether SDK** (Java first) and OpenAPI + client generation | **L** | The standalone design is a strength, but the seams are only lightly tested. This is what makes the ecosystem *one system*. |
| 3 | **Community & visibility + one vertical demo** — de-jargoned homepage, an "Aether in 5 minutes" path, a comparison table (vs LangGraph / LlamaIndex / CrewAI / …), short demos, CONTRIBUTING + good-first-issues, Discussions, a public roadmap board, and **one concrete vertical product** (e.g. claims) driving real requirements | **M** | External signal is very low (1–3★, single-author). The vertical demo also *validates* the core loop with real requirements. |
| 4 | **Observability consistency + operational maturity** — uniform OTel/Micrometer instrumentation, cross-service tracing, SLOs, a unified Grafana dashboard, CI parity (coverage gates + dependency scanning everywhere), and chaos/resilience testing of the mesh | **M** | The stack is already chosen; the gap is *consistency*. Prerequisite for trusting production usage. |
| 5 | **Raise Memory / Vault / Flow to production depth** — before expanding the roadmap surface, take the three younger platforms from "core complete" to production-hardened (see per-repo tables) | **M–L** | Prevents over-expansion; concentrates effort where the loop is thinnest. |

---

## Ecosystem-level themes (the review's 7)

### 1 — Visibility, community & adoption
| Item | Feasibility |
|---|---|
| Polished docs site / homepage (GitHub Pages already partial) + "Aether in 5 minutes" quickstart | **M** |
| Comparison table & positioning vs LangGraph / LlamaIndex / CrewAI / etc. | **S** |
| Short demos / screencasts | **M** (recording effort) |
| `CONTRIBUTING.md`, issue/PR templates, good-first-issues, Discussions, public roadmap board | **S** |
| Dual-/permissive-licensing consideration | **Parked** — license unchanged per owner direction |

### 2 — Shared foundations & consistency
| Item | Feasibility |
|---|---|
| Parent POM / **BOM** for aligned dependency versions (reconciles with the "independent POM" decision) | **M** |
| Common libraries: embedding client, domain primitives, observability, GDPR/security helpers, event schemas | **L** |
| Umbrella Helm chart / docker-compose for the whole stack | **M** |
| Standardize ports, env-var naming, health endpoints, OpenAPI contracts | **M** |
| **Fix org-name drift**: docs/links say `suplab/…` but the org is `doubts-suplab` | **S** (in progress — `ecosystem/ecosystem.yaml` now uses the canonical `doubts-suplab` org) |
| **EEIK adoption completeness** — every repo (incl. the hub + `aether-iel`) now carries a canonical `project-manifest.yaml`; a machine-readable `ecosystem/ecosystem.yaml` descriptor was added. Remaining: engine-driven `eeik activate/lock/verify` against *external* repos (today bound to eeik-bootstrap's own root) | **M** |
| **APEX manages the ecosystem** — APEX ingests `ecosystem.yaml` + each `project-manifest.yaml` to register projects with their governed posture and roll them up in its portfolio. Generic capability — **no Aether-specific code in APEX**; Aether is expressed as data | **M** |

### 3 — Integration depth & contracts
| Item | Feasibility |
|---|---|
| Harden the four seams (Grid↔Core, Grid→Flow, Memory federation, Vault→agents) with versioned contracts | **M** |
| Resilience on every seam: circuit breakers, retries, timeouts, fallbacks | **M** |
| Prefer explicit **event-driven contracts** over ad-hoc HTTP where it fits | **M–L** |
| Thin **Aether SDK** (Java first, others later) + OpenAPI + generated clients | **L** |
| **End-to-end integration tests** across services (Testcontainers-composed) | **M–L** |

### 4 — Observability, operations & quality
| Item | Feasibility |
|---|---|
| Consistent instrumentation (OTel/Micrometer) + cross-service tracing | **M** |
| SLOs + a unified Grafana dashboard for the mesh | **M** |
| CI parity: coverage gates + dependency scanning in **every** repo (Grid is strongest today) | **S–M** |
| Chaos / resilience testing of the mesh | **M–L** |

### 5 — Methodology enforcement & tooling (AIEL)
| Item | Feasibility |
|---|---|
| Automated maturity assessment (run the IMM against a codebase) | **M–L** |
| ADR-generation hooks + phase-gate checks in CI | **M** |
| Tighter artifact↔IEL-phase mapping so Core/Grid/etc. artifacts trace to phases/templates | **M** |

### 6 — Roadmap realism & prioritization
| Item | Feasibility |
|---|---|
| **Freeze** new-platform expansion (Mind/Forge/Mesh/Enterprise) until interop + usage data exist | **S** (a discipline/decision, tracked as a guardrail) |
| Pick **one vertical domain product** (e.g. claims) to drive real requirements | **M** |

### 7 — Security & multi-tenancy hardening
| Item | Feasibility |
|---|---|
| Full STRIDE threat modeling per service | **M** |
| Secret management (no static secrets; vault/KMS integration) | **M** |
| K8s network policies + least-privilege | **M** |
| Audit completeness across the federation surface (Memory federation audit exists — extend the pattern) | **M** |

---

## Per-repo improvement backlog

Each repo tracks the same items locally in its `docs/roadmap.md`. "Partly addressed" flags where recent
phase work already started an item.

### aether (hub)
| Item | Feasibility |
|---|---|
| Keep status tables & phase progress strictly current (ongoing discipline) | **S** |
| Expand ADRs for **cross-repo** decisions | **S–M** |
| Living architecture diagram reflecting *actual* integration points | **M** |
| Contribution-model enforcement (templates, CODEOWNERS, review gates) | **S** |
| More research references + competitive positioning | **S** |

### aether-core
| Item | Feasibility |
|---|---|
| Deeper emotional/procedural reasoning | **M–L** |
| Memory-strength dynamics + retrieval-quality metrics | **M** |
| Embedding + recall performance under load | **M** |
| Richer context assembly (beyond simple snapshots) | **M** |
| More decay/reinforcement test coverage | **S–M** |
| Clearer feedback loop into Grid | **M** |
| _Follow-ups already tracked:_ memory export (Art. 20), requester identity verification, retention purge (V008) | **S–M** |

### aether-grid (most mature)
| Item | Feasibility |
|---|---|
| Deeper integration with Memory / Vault / Flow | **M–L** |
| More sophisticated agent orchestration + evaluation | **M–L** |
| Proxy latency/throughput under realistic load | **M** |
| Stronger hallucination/reflection loops with **measurable** improvement | **L** |
| Expand agent SPI + marketplace-like extensibility | **M–L** |
| Production-harden the confidence gate + DEFER path | **M** |

### aether-memory
| Item | Feasibility |
|---|---|
| Federation robustness: failure handling, consistency, richer projection _(partly addressed — audit + rate-limit + peer fan-out shipped in Phase 2)_ | **M** |
| Distributed (shared) rate limiter + per-peer auth _(tracked as Phase 2 follow-up)_ | **M–L** |
| Multi-team / multi-org memory graphs | **M–L** |
| Richer policy language | **M** |
| Shared-reinforcement performance under concurrent access | **M** |
| Clearer ownership boundary vs Core personal memory (doc + enforcement) | **S–M** |

### aether-vault
| Item | Feasibility |
|---|---|
| More enterprise connectors (SharePoint, Confluence, Google Drive, …) | **M** each / **L** in aggregate |
| Higher-quality entity/relation extraction (LLM-assisted / hybrid) _(follow-up already noted)_ | **M–L** |
| Hybrid search (vector + keyword) | **M** |
| Multi-modal support | **L** |
| Re-indexing pipelines | **M** |
| RAG-quality evaluation (retrieval precision/recall, context usefulness) | **M** |
| Entity-aware RAG + entity resolution/de-dup _(Phase 2 follow-ups)_ | **M–L** |

### aether-flow
| Item | Feasibility |
|---|---|
| Richer step types + non-linear patterns (parallel AND fork/join) _(explicitly deferred — needs a multi-token instance model)_ | **M–L** |
| Visual/designer UI or BPMN import (if intended) | **L** |
| More sophisticated escalation chains + notifications _(partly addressed — chains + logging notifier shipped in Phase 2; webhook/email + business-hours are follow-ups)_ | **M** |
| Operator visibility + metrics | **M** |
| Resilience of long-running instances | **M** |
| Tighter contract with Grid's confidence decisions | **M** |

### aether-iel
| Item | Feasibility |
|---|---|
| More complete phase guides + examples mapped to the *actual* runtime repos | **M** |
| Runnable checklists / tooling against a codebase | **M–L** |
| Automated maturity scoring | **M–L** |
| Case studies of applying AIEL to Core / Grid / etc. | **S–M** |
| Broader adoption beyond the Aether team | **S** |

---

## The main risks this backlog manages

1. **Fragmentation** from independent evolution of the platform services → themes 2 & 3 (shared
   foundations + integration contracts).
2. **Slow adoption** from low visibility (and AGPL copyleft) → theme 1 + the vertical demo. (License
   stays; we address adoption through docs, positioning, and a real demo — not relicensing.)
3. **Over-expansion** before the core loop is battle-tested → guardrails + next-step #5.

> Items graduate to each repo's `progress.md` as they ship. Nothing here is claimed as delivered.
