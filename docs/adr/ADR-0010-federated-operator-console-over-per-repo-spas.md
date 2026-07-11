# ADR-0010 — A single federated operator console, not a per-repository SPA

> **Status:** Proposed
> **Date:** 2026-07-11
> **Repository:** ecosystem-wide
> **Deciders:** Architect, Enterprise Architect, Governance Officer
> **Supersedes:** —

---

## Context

The Aether ecosystem is deliberately **headless**. Every runnable component exposes a REST API and nothing else:

| Service | App | Port |
|---|---|---|
| Aether Grid | `aether-proxy` / `aether-api` | 8080 / 8081 |
| Aether Core | `core-api` | 8082 |
| Aether Memory | `memory-api` | 8083 |
| Aether Vault | `vault-api` | 8084 |
| Aether Flow | `flow-api` | 8085 |

Two repositories — `aether` (philosophy/ecosystem index) and `aether-iel` (methodology) — have **no runtime at all**.

There is real operational value being left on the table: nobody can *see* a running workflow instance, a pending human-approval queue, an SLA breach, a Grid DEFER landing in Flow, memory decay, or a Vault document going STALE without hand-crafting API calls. The question this ADR answers is **how** we add human-facing monitoring/visualisation/light-orchestration surfaces without eroding the properties that make the ecosystem coherent: strong bounded contexts, standalone-runnable services, and no premature investment while most platforms are Phase 0 scaffolds.

The tempting default — "give each repo its own light SPA" — is the option this ADR exists to weigh against the alternatives.

---

## Decision Drivers

- **Bounded contexts must stay intact** — a UI must not become a backdoor that couples services or reaches into another module's internals (Golden Rule 5).
- **Each platform must remain standalone-runnable** — observability of a single service cannot depend on the others being present.
- **The high-value views are cross-cutting** — the interesting signal lives at the *seams* (Grid DEFER → Flow approval queue, reinforcement, freshness), which no single-service view can show.
- **Proportionate investment** — six of seven repos are Phase 0 scaffolds; two have no runtime. Effort must not be multiplied across surfaces that barely exist yet.
- **Low ongoing carrying cost** — auth, design system, build pipeline, and dependency upkeep should exist in as few places as possible.
- **Read-mostly, safety-preserving** — a console must never auto-decide an approval gate or a deferral; human-in-the-loop and the 0.8 confidence gate (ADR-0004) are inviolable.

---

## Options Considered

### Option 1 — A light SPA embedded in every repository

Each `*-api` ships its own single-page front-end for its own domain.

**Pros:**
- Symmetric — every repo looks the same.
- Each UI is co-located with, and versioned alongside, the service it fronts.
- A service run in isolation carries its own UI with it.

**Cons:**
- **7× carrying cost** — seven design systems, seven auth integrations, seven JS build pipelines, seven dependency-upgrade streams. "Seven light SPAs" is not light.
- **Structurally blind to the seams** — a per-repo SPA can only ever render one bounded context; the Grid→Flow handoff, the highest-value view, is invisible to all of them.
- **Premature** — spends UI effort on scaffolds (and two repos with no runtime) before there is behaviour to observe.
- **Drift risk** — seven front-ends diverge in look, auth, and conventions over time.

---

### Option 2 — A single federated operator console (chosen)

One separate front-end application — **Aether Console** — that consumes each service's public REST + health surface as a *client* and federates them into one operator view. Paired with a near-zero-cost per-service observability baseline (Spring Boot Actuator).

**Pros:**
- **One** design system, auth integration, and build pipeline to maintain.
- **Renders the seams** — the console sees every service, so cross-context flows (Grid DEFER → Flow approval, reinforcement, freshness) become first-class views.
- **Respects bounded contexts** — the console imports no module internals; it is just another API consumer, exactly the relationship Grid already has with Core via `PersonalContextPort`.
- **Proportionate** — one focused investment, started where value is highest (Flow, then Grid), not smeared across scaffolds.
- Services stay independently observable in isolation via Actuator, satisfying the standalone constraint without any UI.

**Cons:**
- Introduces a new component (repo/app) and a cross-service client to own.
- The console must track each service's API contract; contract changes ripple to one consumer.
- A naïve implementation could reach past public APIs — this must be guarded by convention (public REST + Actuator only).

---

### Option 3 — No UI; standardise on Actuator + external dashboards (Grafana/Prometheus) only

Expose metrics/health from every service and build all human views in an external observability stack.

**Pros:**
- Lowest bespoke-code cost; leans on Grid's existing OpenTelemetry + Prometheus + Grafana stack.
- Excellent for time-series operational monitoring.

**Cons:**
- Grafana is for metrics, not **domain** views — it cannot render a workflow graph, drive an approval queue, or let a human approve/reject/escalate a task.
- Leaves the actual product gap (operate the flows) unfilled.
- No light-orchestration path at all.

---

## Decision

**We choose Option 2 — a single federated operator console (Aether Console), backed by a per-service Actuator baseline.**

We reject giving each repository its own SPA. The value of monitoring this ecosystem is inherently cross-cutting and lives at the service seams; a per-repo UI is structurally incapable of showing it, while costing 7× to build and maintain. A single console that consumes public APIs as a client delivers the cross-context view, keeps bounded contexts intact (it is a consumer, not a shared module), and concentrates the design/auth/build cost in one place. The per-service Actuator baseline preserves the hard requirement that each platform remains observable when run standalone, at near-zero cost and with no UI dependency.

The console is **read-mostly**: it visualises and monitors freely, and permits only bounded, human-driven orchestration actions (e.g. start a workflow, approve/reject/escalate a task). It never auto-decides an approval gate or a Grid deferral — escalation raises visibility only, and the 0.8 confidence gate (ADR-0004) is preserved end to end.

**Delivery tiers:**

- **Tier 0 — per-service baseline (near-zero code):** enable Spring Boot Actuator (`/actuator/health`, `/info`, `/metrics`, `/prometheus`) on every `*-api`. Optionally one static read-only status page per service for standalone humans. No SPA, no JS build.
- **Tier 1 — Aether Console (the investment):** one front-end federating all services, sequenced by value — **Flow first** (workflow graph, running instances, approval queue, SLA view), then **Grid** (policy/decision governance, DEFER stream), then read-only stats for Core/Memory/Vault.

**Home and technology** are settled as implementation notes below, not re-opened per repo.

---

## Consequences

### Positive
- The Grid→Flow DEFER seam, workflow state, and approval queues become observable and operable for the first time.
- One design system, one auth story, one build pipeline — minimal carrying cost.
- Bounded contexts are strengthened, not weakened: the console formalises the "public API is the only contract" boundary by being a pure consumer of it.
- Every service gains standard, standalone observability via Actuator regardless of console availability.

### Negative / Trade-offs
- A new component to own, with a build/deploy/auth lifecycle of its own.
- The console is coupled to the *public contracts* of every service; a breaking API change now has a downstream consumer to update.
- Centralisation means a console outage removes the unified view (individual Actuator endpoints remain as fallback).

### Neutral / Notable
- `aether` and `aether-iel` (no runtime) contribute nothing to the console — correct and expected; not every repo needs a surface.
- This ADR governs *operator/monitoring* UI only. It does not commit the ecosystem to any end-user/product UI, which would be a separate decision.
- Maps onto AIEL **Phase 8 (Operational Engineering)** — "Is it observable?" — and should be reflected in each platform's roadmap when console work is scheduled.

---

## Implementation Notes

- **Home:** a new `aether-console` repository (preferred), or a `console/` app under the `aether` hub. It must **not** live inside any single platform repo — that would violate the bounded-context boundaries this ADR protects.
- **Integration contract:** the console consumes each service's **public REST API + Actuator** only. It imports no `*-domain`/`*-engine` internals and shares no database — data isolation per each platform's CLAUDE.md is preserved. Same posture as `AetherCoreHttpAdapter` / `PersonalContextPort`.
- **Technology:** default to **server-rendered Thymeleaf + htmx** to stay genuinely "light" (no separate JS toolchain, renders straight from the APIs). Reserve a full SPA framework only if/when Flow needs rich interactive graph editing.
- **Safety:** all mutating actions route through the owning service's existing API and its existing authority checks. The console adds no new decision authority; it cannot approve/reject/escalate or resolve a deferral outside the human-in-the-loop path.
- **Sequencing:** Tier 0 (Actuator) can land per service independently and immediately; Tier 1 starts with Flow as the proof-of-concept.
- **Docs sync:** when console work is scheduled, update each affected platform's `docs/roadmap.md` / `docs/progress.md` and note the Phase 8 mapping in `aether-iel`.

---

## Review

| Reviewer | Role | Decision | Date |
|---|---|---|---|
| | Architect | Approve / Reject | |
| | Governance Officer | Approve / Reject | |
| | Maintainer | Approve / Reject | |
