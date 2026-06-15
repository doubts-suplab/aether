# Related Projects & Prior Art

> Projects, frameworks, and systems that inform or relate to the Aether ecosystem.
> Organized by domain. Not endorsements — critical comparison is the intent.

---

## Cognitive and Memory Systems

**MemGPT** (Packer et al., 2023 — now Letta)
A system that gives LLMs the ability to manage their own memory — paging information in and out of context like an operating system manages virtual memory. MemGPT focuses on in-context memory management for long conversations. Aether's scope is broader: persistent cross-session memory with semantic retrieval, decay modeling, and multi-type classification — not just context window management.

**Zep**
A memory layer for AI assistants. Provides episode storage, semantic search, and a graph memory model. Closest to what AetherCore does in the open-source ecosystem. Aether differs in: the four-type memory model, reinforce-on-read decay mechanics, GDPR-first design, direct integration with an agent governance layer (Grid), and the AIEL methodology framework.

**LangChain Memory**
In-conversation memory abstractions for LangChain agents. Primarily designed for single-session use. Does not address cross-session persistence, memory decay, or enterprise governance requirements.

---

## Agent Frameworks

**LangGraph**
A graph-based framework for building stateful multi-agent systems. Strong on workflow definition and state management. Aether differs in focus: Grid is an enterprise governance layer first (policy-as-code, confidence gates, multi-tenancy, audit trails) rather than a general-purpose agent orchestration framework.

**AutoGen** (Microsoft)
A multi-agent conversation framework with human-in-the-loop capabilities. Strong on agent conversation patterns. Aether Grid's multi-agent debate pattern is conceptually similar to AutoGen's group chat, but Grid's primary context is API governance rather than conversational agents.

**CrewAI**
Role-based multi-agent framework. Agents take on personas and collaborate on tasks. Aether's agent model is governance-first — agents make decisions about requests rather than collaborating on open-ended tasks.

**Semantic Kernel** (Microsoft)
An SDK for integrating LLMs with application code via plugins and planners. Aether is not an SDK — it is a deployed platform with its own persistence layer, agent registry, and governance infrastructure.

---

## API Gateways and Governance

**Kong Gateway**
A production API gateway with rich plugin ecosystem. AetherGrid's proxy module uses Spring Cloud Gateway for the transport layer, which is conceptually similar to Kong. The difference is Grid's intelligence layer: Kong routes and rate-limits; Grid reasons about requests and personalizes responses.

**AWS API Gateway**
Managed API gateway service from AWS. Handles routing, throttling, and basic auth. No cognitive layer, no agent pipeline, no confidence-gated decision-making. AetherGrid complements rather than replaces cloud-native gateways — it can sit behind them.

---

## Vector Databases

**Pinecone**
Managed vector database service. Strong performance at scale. AetherCore uses pgvector (PostgreSQL extension) rather than Pinecone because: it keeps all data in a single PostgreSQL instance (simpler operations, stronger consistency, easier GDPR compliance), the 384-dim embeddings from all-MiniLM-L6-v2 are well within pgvector's performance envelope at the target scale, and it avoids a managed service dependency for what is core cognitive infrastructure.

**Weaviate**
Open-source vector database with a rich query model and built-in GraphQL API. A valid alternative to pgvector. The trade-off is operational complexity — running a separate vector database versus a PostgreSQL extension. Revisit if AetherMemory (Phase 2) needs to scale beyond what pgvector can support.

**ChromaDB**
Lightweight local vector database popular in development contexts. Not suitable for production AetherCore deployments due to limited persistence guarantees and multi-tenancy constraints.

---

## Methodology

**MLOps / LLMOps frameworks** (MLflow, Weights & Biases, Langfuse)
Evaluation and experiment tracking for ML and LLM systems. These tools complement AIEL — they can be used to implement the tracking and evaluation infrastructure that AIEL Phase 7 requires. AIEL is the methodology; these tools are the implementation layer for the observability and evaluation phases.

**Enterprise Architecture frameworks** (TOGAF, Zachman)
Aether IEL draws from enterprise architecture thinking — the structured lifecycle, governance gates, and artifact-first approach have parallels to TOGAF's ADM. AIEL is specifically designed for intelligence systems, where traditional EA frameworks do not address memory engineering, agent contracts, or confidence thresholds.
