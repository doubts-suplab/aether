# [Component Name] — Architecture

> **Repo:** [`suplab/[repo]`](https://github.com/suplab/[repo])
> **Role:** [One-line role in the ecosystem]
> **Layer:** Foundation | Methodology | Cognitive Engine | Agent Mesh | Platform | Domain
> **IMM Level:** L1–L5
> **Status:** Planned | In Progress | Phase N Complete | Production-Ready

---

## Purpose

_Two to four paragraphs. What does this component do, and why does it need to exist as a separate component? What would be impossible or much harder without it? What are the boundaries — what does it explicitly not do?_

---

## Position in the Ecosystem

_Where does this component sit in the Aether dependency graph? What does it consume from below, and what does it provide upward?_

```
[Higher layer / consumers]
         ↓
[This component]
         ↓
[Dependencies / lower layer]
```

**Consumes from:**
- `[component]` — [what it calls and why]

**Consumed by:**
- `[component]` — [what they call and why]

**Integration contract:**
- `[endpoint or event topic]` — [description]

---

## Module Structure

_For runtime components: describe the Maven modules or equivalent package structure. For documentation-only components: describe the directory structure._

```
[repo]/
├── [module-1]/       ← [purpose]
├── [module-2]/       ← [purpose]
└── [module-3]/       ← [purpose]
```

### [Module 1 Name]

_What lives here. Key types, interfaces, or documents. Dependency rules — what this module may and may not import._

### [Module 2 Name]

_What lives here._

---

## Domain Model

_For runtime components: the key domain types. Use prose to explain why each type exists and what it represents._

| Type | Description |
|---|---|
| `[TypeName]` | |

**Port interfaces:**
- `[InterfaceName]` — [what it declares]

---

## Key Flows

_Describe the most important runtime flows. Use numbered steps. For each flow, include: trigger, steps, outcome._

### [Flow 1 Name]

1. [Trigger]
2. [Step]
3. [Step]
4. [Outcome]

---

## Data Model

_Tables, collections, or file structures. For each table, describe the key columns and their purpose. Include any indexes that are significant for performance._

| Table / Collection | Key Columns | Notes |
|---|---|---|
| | | |

---

## Non-Functional Characteristics

| Concern | Approach |
|---|---|
| Latency | |
| Availability | |
| Scalability | |
| Resilience | |
| Observability | |
| Security | |

---

## AIEL Phase Coverage

_Which AIEL phases has this component completed? Link to the relevant phase artifacts._

| Phase | Status | Primary Artifact |
|---|---|---|
| 1 — Strategy | ☐ Pending / ✅ Complete | [Intelligence Brief link] |
| 2 — Cognitive Architecture | ☐ / ✅ | [Cognitive Design link] |
| 5 — Agent Engineering | ☐ / ✅ | [Agent Contract link] |
| 7 — Evaluation | ☐ / ✅ | [Evaluation Report link] |
| 9 — Governance | ☐ / ✅ | [Policy link] |

---

## Architecture Decision Records

_ADRs that shaped this component's design._

| ADR | Decision | Status |
|---|---|---|
| ADR-001 | [Brief description] | Accepted |

---

## Open Questions / Future Considerations

_Known unknowns and deliberate deferral decisions. This section documents what was considered and deliberately left for later._

- 
