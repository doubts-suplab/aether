# Engineering Principles

> These eight principles are the lens through which every architecture decision in the Aether ecosystem is evaluated.
> They are not aspirational — they are constraints. A design that violates a principle is a design that needs revision.

---

## 1. Memory First

Memory is a first-class capability, not a feature that gets added later. In most AI systems, memory is an afterthought — a flat conversation log appended to every prompt. In Aether, memory is a core primitive with its own type system (EPISODIC, SEMANTIC, PROCEDURAL, EMOTIONAL), its own lifecycle (capture → embed → associate → reinforce → decay → forget), and its own governance (retention policies, consent checks, erasure rights).

This principle means every system design must answer the memory question before any other: *What does this system need to remember, how should it be stored, and how should it decay?* Answers belong in the Phase 4 Memory Engineering artifact — not discovered during implementation.

---

## 2. Model Agnostic

No Aether component has a hard dependency on a single LLM provider. All LLM calls flow through a `LlmClient` port interface. The concrete implementation — Ollama, Anthropic, OpenAI, AWS Bedrock, Groq, or any future model — is injected at configuration time, not hardcoded.

This is not just a portability concern. It is a risk management principle. LLM providers change pricing, alter API contracts, introduce latency regressions, and occasionally shut down. A system with a hard provider dependency is one provider decision away from an outage. Aether systems degrade gracefully when the LLM is unavailable — they do not crash.

---

## 3. Event Driven

Every meaningful state change in an Aether system is an event: a memory stored, an agent decision made, a session opened, a memory reinforced, a GDPR erasure requested, a governance policy evaluated. Events are the primary unit of integration between components — not direct function calls, not shared database tables, not synchronous HTTP chains.

This principle keeps components decoupled, makes the system auditable (the event log is the audit trail), and enables temporal decoupling — Grid can publish a feedback event and Core can consume it minutes later without either needing to know about the other's internals. Kafka is the default event bus. Where Kafka is unavailable, the transactional outbox pattern preserves consistency.

---

## 4. Human in the Loop

Humans remain the authoritative decision makers in every Aether system. This is not a soft preference — it is a hard architectural constraint expressed as the confidence gate: any agent decision with a confidence score below 0.8 must not be auto-enforced. It must surface to a human reviewer.

This threshold cannot be disabled. It cannot be feature-flagged. It is in the code, not in configuration. The only way to change it is through a governance review that produces a signed Architecture Decision Record.

The confidence gate is the most important single mechanism in Aether. It is what separates a system that augments human judgment from one that replaces it without accountability.

---

## 5. Explainability

Every agent decision must be inspectable. Every BLOCK, RATE_LIMIT, and ALERT decision includes a human-readable reason field. Every memory retrieval has a provenance trail — when was this memory created, how many times has it been accessed, what is its current strength, what was the cosine similarity score that caused it to be retrieved?

Explainability is not about generating explanatory text after the fact. It is about designing the data model so that explanation is always possible. The audit log is append-only and immutable. The rationale field is required, not optional. The trace context is propagated to every downstream component.

Without explainability, governance is impossible. Without governance, enterprise deployment is impossible.

---

## 6. Composability

Aether capabilities are modular and independently deployable. AetherCore and AetherGrid can run in isolation. AetherCore does not know about AetherGrid's internals — it exposes a REST API. AetherGrid does not know about AetherCore's implementation — it calls the API. Platform components (AetherMemory, AetherVault, AetherFlow) can be consumed by any domain application without coupling to each other.

This principle is enforced at the code level through hexagonal architecture: domain logic in the core module, port interfaces as the only cross-boundary contracts, adapters as the only place where implementation details live. No module imports another module's internal classes. All cross-module calls go through declared port interfaces.

Composability means the system can grow without a big-bang rewrite. New capabilities plug in through new port implementations. New domains extend the platform without forking it.

---

## 7. Enterprise Readiness

Security and governance are built in from day one — they are not layered on after the fact. This principle has concrete implementation requirements:

- Every database table has a `tenant_id` column in every `WHERE` clause (multi-tenancy by design, not by convention)
- Parameterized queries everywhere — no string-concatenated SQL, ever
- PII is redacted before any log write (regex patterns for email, phone, credit card, SSN, JWT, API keys)
- GDPR Article 17 compliance is a hard architectural requirement: hard-delete of all user data (memories, sessions, consent records) within a single transaction on verified erasure request
- Non-root containers, read-only root filesystems, no capability escalation in Kubernetes
- OWASP CVE scan with a CVSS ≥ 9.0 gate on every pull request

Enterprise readiness cannot be achieved by adding a security team's review at the end of a project. It requires security-conscious design at every level.

---

## 8. Evolvability

The architecture is designed to grow more capable over time — not to drift stale. Memory strength evolves: memories reinforce when accessed and decay when unused, so the system's effective knowledge base naturally shifts toward what is relevant. Agents improve through feedback loops: CORRECT/INCORRECT/PARTIALLY_CORRECT signals flow back into the system and influence future confidence calibration.

At the architecture level, evolvability means: no design decision should make the next phase harder to implement. Hexagonal architecture ensures that replacing an adapter (a different embedding model, a different vector store, a different LLM) does not require touching domain logic. Port interfaces are the stability guarantee — implementations behind them can evolve freely.

The Intelligence Engineering Lifecycle (AIEL) operationalizes evolvability: Phase 11 (Evolution Engineering) and Phase 10 (Learning Engineering) are permanent phases, not one-time events. Evaluation (Phase 7) repeats with every evolution cycle. A system that does not invest in these phases will calcify.
