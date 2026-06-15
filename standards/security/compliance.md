# Security and Compliance Standard

> Applies to all Aether repositories. Security is not a phase — it is a baseline.

---

## Secrets Management

| Rule | Detail |
|---|---|
| No hardcoded secrets | Credentials, API keys, and tokens must be in environment variables |
| No secrets in git | `.env` files, tokens, and passwords must be in `.gitignore` |
| OIDC in CI/CD | GitHub Actions must use OIDC — never static long-lived secrets |
| Secret rotation | All secrets must be rotatable without code changes |

Example `.env.example` pattern (committed — no values):
```
POSTGRES_PASSWORD=
AETHER_CORE_API_KEY=
OLLAMA_BASE_URL=
```

---

## Data Isolation — Multi-Tenancy

Every SQL query against any table that contains tenant data must include `tenant_id` in the `WHERE` clause.

```sql
-- ✅ Correct
SELECT id, content FROM personal_memories
WHERE user_id = :userId AND tenant_id = :tenantId;

-- ❌ Forbidden — cross-tenant data access
SELECT id, content FROM personal_memories WHERE user_id = :userId;
```

**Row-Level Security (RLS)** is enabled on all tables in PostgreSQL. The application-level check above is a defence-in-depth layer on top of database-level RLS.

---

## GDPR Compliance

### PII Redaction

All data that may contain PII must pass through `GdprRedactionService` before storage or logging.

Redacted patterns:
- Email addresses
- E.164 phone numbers
- Credit card numbers (Visa, Mastercard, Amex)
- Social Security Numbers
- JWT Bearer tokens
- API keys

```java
// ✅ Correct
var redacted = gdprRedactionService.redact(rawContent);
memoryStore.save(memory.withContent(redacted), embedding);

// ❌ Forbidden — raw PII stored
memoryStore.save(memory, embedding);
```

### Right to Erasure

Every service that stores user data must implement a GDPR erasure endpoint:

```
DELETE /api/v1/users/{userId}/gdpr/erase
DELETE /api/v1/tenants/{tenantId}/memories  (Grid)
```

Erasure must be hard-delete — not soft-delete. Audit log entries survive (no FK constraints by design) but PII within them is redacted.

### Memory Opt-Out

Tenants and users may opt out of memory storage. When opt-out is set:
- No embeddings are generated
- No memories are persisted
- Existing memories are purged

---

## Authentication and Authorisation

### API Gateway (AetherGrid)

- All API calls authenticated via `X-API-Key` header
- Key hashed with SHA-256 before storage — never stored in plaintext
- Unknown or suspended tenant → HTTP 401
- Actuator endpoints bypass authentication (internal health checks only)

### Admin API (AetherGrid / Core)

- Stateless JWT via OAuth2 Resource Server
- All `/api/**` paths require a valid JWT
- Swagger UI and `/dashboard/**` are open (no auth required)

### Service-to-Service (Core ↔ Grid)

- `AETHER_CORE_API_KEY` passed via `Authorization` header
- Core validates the key on every Grid request
- Circuit breaker in Grid — Core unavailability degrades gracefully, does not block Grid

---

## Container Security

All Dockerfiles must:
- Use a non-root user (`uid 1000`)
- Use multi-stage builds (JDK build → JRE runtime)
- Include a `HEALTHCHECK` instruction
- Set `--enable-preview` and `-XX:+UseContainerSupport`

```dockerfile
# ✅ Correct — non-root, multi-stage
FROM eclipse-temurin:21-jdk-noble AS build
# ... build steps ...
FROM eclipse-temurin:21-jre-noble
RUN adduser --system --uid 1000 aether
USER aether
```

---

## Kubernetes Security Posture

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop: ["ALL"]
```

All Kubernetes `ServiceAccount`s must set `automountServiceAccountToken: false`.

---

## Dependency Security

- OWASP Dependency-Check runs on all PRs targeting `main`
- Build fails on any CVE with CVSS score ≥ 9.0
- Known false positives suppressed in `.github/owasp-suppressions.xml`

---

## Audit Trail

All significant events must be written to `audit_log`:
- Tenant lifecycle changes (created, suspended, reactivated)
- Policy lifecycle changes (created, activated, archived)
- GDPR actions (opt-out, erasure)
- Agent blocking decisions

The audit log table has no FK constraints so that audit entries survive entity deletion. PII within log entries is redacted before insertion.
