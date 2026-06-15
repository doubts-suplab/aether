# System Context

> How the Aether ecosystem fits into the wider world — who interacts with it, what it depends on, and what it integrates with.

---

## Actors

### End Users
Individuals whose cognitive context AetherCore manages. They interact with Aether indirectly — through applications built on top of the platform (AetherMind, domain products) or through agents that call the AetherCore personal context API before making decisions. End users have GDPR rights: they can view their stored data, update their consent, and request erasure.

### Developers
Engineers building agent pipelines, domain applications, and platform extensions on top of AetherCore and AetherGrid. They interact with the system via the REST APIs, the agent registry, and the developer tooling (AetherForge in Phase 4). Developers write Agent Contracts, register agents in the registry, and define tenant-scoped policies.

### Administrators / Operators
Responsible for the health and governance of deployed Aether instances. They use the AetherGrid admin API, the observability dashboards, and the audit log review tooling. They manage tenant onboarding, policy configuration, and capacity planning. They are the first responders when the Governance Officer raises an alert.

### Governance Officers
Defined in the AIEL methodology. Governance Officers have veto power over Phase 7 Evaluation sign-offs and must approve all Evolution Strategies (Phase 11). They own the policy configuration in Grid and review the audit log on a weekly schedule. Their role is distinct from administration — they govern the *intelligence*, not the infrastructure.

---

## Internal Dependencies

| Component | Role | Technology |
|---|---|---|
| PostgreSQL 16 + pgvector | Primary data store for memories, sessions, consent, decisions, audit log | Hosted in Kubernetes as StatefulSet |
| Apache Kafka | Event mesh — feedback signals, audit events, alerts, approval queues | Optional in Core, required in Grid |
| Ollama | Local embedding inference — all-MiniLM-L6-v2 (384-dim) | Sidecar or shared service |
| LLM Provider | Agent reasoning — Anthropic, OpenAI, Groq, AWS Bedrock, or local | Injected via `LlmClient` port |
| Flyway | Database schema migration management | Runs on application startup |

---

## External Integrations

### OAuth / OIDC Providers
AetherGrid validates JWT bearer tokens from any standards-compliant OIDC provider (Keycloak, Auth0, AWS Cognito, Azure AD). Tenant ID and user ID are extracted from JWT claims. API key authentication is also supported for service-to-service calls — keys are stored as SHA-256 hashes, never in plaintext.

### Cloud Platforms
AetherCore and AetherGrid are designed to deploy on any Kubernetes distribution: vanilla Kubernetes, AWS EKS, Google GKE, Azure AKS, or Red Hat OpenShift. GitHub Actions CI/CD uses OIDC for credential-less deployment. Docker images are published to GHCR and are multi-arch (linux/amd64 + linux/arm64).

### Enterprise API Ecosystems
AetherGrid's primary use case is to sit in front of an enterprise's existing API estate as an intelligent governance proxy. It routes incoming requests through its agent pipeline without requiring changes to downstream APIs. The downstream systems see traffic that has already been classified, enriched with personal context, and evaluated for compliance.

### Document and Knowledge Systems
AetherVault (Phase 2) will integrate with enterprise document stores (SharePoint, Confluence, S3, Google Drive) to index documents, extract knowledge, and make it available to agents via RAG pipelines. The integration uses standard APIs — no proprietary connectors.

---

## Non-Functional Requirements

### Horizontal Scalability
Both AetherCore and AetherGrid are stateless Spring Boot applications that scale horizontally. Kubernetes HPA manages replica counts based on CPU utilisation (target 70%). AetherCore scales 2–8 replicas. AetherGrid proxy and admin each scale independently. The database (PostgreSQL) is the only stateful component and is managed separately.

### High Availability
Production deployments require a minimum of 2 replicas per service. Kubernetes liveness probes (`/actuator/health/liveness`) and readiness probes (`/actuator/health/readiness`) ensure that unhealthy pods are removed from service before they impact traffic. Rolling deployments with zero-downtime are the default release strategy.

### Multi-Tenancy
Every data entity in the system is scoped to a tenant. Tenant isolation is enforced at the query level — `tenant_id` appears in every `WHERE` clause and is validated on every write. There is no shared state between tenants. A failure or compromise in one tenant's data does not affect other tenants.

### Auditability
Every agent decision is written to an append-only audit log with: agent name, decision type, confidence score, reason, trace ID, tenant ID, user ID, and timestamp. The log is immutable — no UPDATE or DELETE is permitted. Audit log retention follows the tenant's data retention policy as configured in `gdpr_consent`. All BLOCK, RATE_LIMIT, and ALERT decisions are queryable by the Governance Officer.

### Security
The security posture is defence-in-depth: JWT validation at the network edge, tenant isolation at the database layer, PII redaction before any log write, container hardening (non-root uid 1000, read-only root filesystem, dropped capabilities), OWASP CVE scanning on every pull request (CVSS ≥ 9.0 fails the build), and no hardcoded secrets (all secrets via environment variables or Kubernetes Secrets).
