# Observability Standard

> Every Aether service must be fully observable from day one. Observability is not optional.

---

## Three Pillars

| Pillar | Tool | Notes |
|---|---|---|
| Metrics | Micrometer + Prometheus | Scraped by Prometheus, visualised in Grafana |
| Tracing | OpenTelemetry + OTLP | Distributed traces across services |
| Logging | SLF4J + structured JSON | Parameterised messages, no PII in logs |

---

## Required Endpoints

Every Spring Boot service must expose:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus
  endpoint:
    health:
      probes:
        enabled: true   # liveness + readiness for Kubernetes
```

| Endpoint | Purpose |
|---|---|
| `GET /actuator/health` | Liveness + readiness probes |
| `GET /actuator/health/liveness` | Kubernetes liveness probe |
| `GET /actuator/health/readiness` | Kubernetes readiness probe |
| `GET /actuator/prometheus` | Prometheus scrape endpoint |

---

## Metrics Naming Conventions

All custom metrics must follow the `aether.{service}.{noun}.{verb}` pattern:

| Metric | Type | Labels | Description |
|---|---|---|---|
| `aether.agent.executions` | Counter | `agent`, `decision` | Agent execution count by decision type |
| `aether.agent.latency` | Timer | `agent` | Agent execution time |
| `aether.memory.operations` | Counter | `operation`, `type` | Memory store read/write/delete count |
| `aether.proxy.requests` | Counter | `tenant`, `outcome` | API proxy request count |
| `aether.policy.evaluations` | Counter | `action` | Policy evaluation results |
| `aether.session.created` | Counter | — | Cognitive session creation count |
| `aether.memory.decay.applied` | Counter | — | Memory decay scheduler runs |

---

## Logging Standards

### Mandatory Fields

Every log line must include structured context at appropriate levels:

```java
// ✅ Service operation — DEBUG
log.debug("Saved memory id={} userId={} type={} strength={}",
    memory.id(), memory.userId(), memory.type(), memory.strength());

// ✅ Business event — INFO
log.info("Session created sessionId={} userId={} tenantId={}",
    session.sessionId(), session.userId(), session.tenantId());

// ✅ Degraded path — WARN
log.warn("Ollama unavailable, falling back to zero vector userId={}", userId);

// ✅ Failure — ERROR with exception
log.error("Failed to store memory userId={} type={}", userId, type, e);
```

### Prohibited in Logs

- Passwords, API keys, or tokens
- Raw PII (email, phone, SSN) — run through `GdprRedactionService` first
- `System.out.println()` — use SLF4J

---

## Distributed Tracing

Configure OpenTelemetry in every service:

```yaml
management:
  tracing:
    sampling:
      probability: 1.0   # 100% in dev; reduce in production
  otlp:
    tracing:
      endpoint: ${OTEL_EXPORTER_OTLP_ENDPOINT:http://localhost:4318/v1/traces}
```

Trace context must be propagated across service boundaries:
- AetherGrid → AetherCore calls must forward `traceparent` / `tracestate` headers
- Kafka messages must include trace context in headers

---

## Prometheus Configuration

Services are scraped at:
- AetherProxy: `host.docker.internal:8080/actuator/prometheus`
- AetherAPI: `host.docker.internal:8081/actuator/prometheus`
- AetherCore: `host.docker.internal:8082/actuator/prometheus`

Kubernetes deployments expose a `ServiceMonitor` resource for Prometheus Operator.

---

## Grafana Dashboards

Each service must have a Grafana dashboard covering:
- Request rate (RPS)
- Error rate (4xx, 5xx)
- Latency (p50, p95, p99)
- Service-specific metrics (agent decisions, memory operations, session counts)
- JVM heap, GC, and thread pool metrics

Dashboard JSON files are stored in `aether-infra/grafana/dashboards/`.

---

## Alert Thresholds

| Alert | Threshold | Severity |
|---|---|---|
| Error rate | > 5% for 5 minutes | WARNING |
| Error rate | > 15% for 2 minutes | CRITICAL |
| p99 latency | > 2s for 5 minutes | WARNING |
| JVM heap | > 85% for 3 minutes | WARNING |
| Circuit breaker open | any opening | WARNING |
| Agent BLOCK rate | > 20% of decisions | WARNING |
