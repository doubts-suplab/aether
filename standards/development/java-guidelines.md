# Java Engineering Guidelines

> Applies to all Aether repositories that contain Java source code.
> Golden rules are non-negotiable. Deviations require an ADR.

---

## Language and Framework

| Requirement | Value |
|---|---|
| Java version | 21 (with `--enable-preview`) |
| Framework | Spring Boot 3.3.x |
| Namespace | `jakarta.*` exclusively — `javax.*` is a build error |
| Compiler flags | `--enable-preview`, `-parameters` (both required) |
| Build tool | Maven multi-module |

---

## Ten Golden Rules

### 1. Constructor Injection Exclusively

```java
// ✅ Correct
public class MyService {
    private final DependencyA a;
    private final DependencyB b;

    public MyService(DependencyA a, DependencyB b) {
        this.a = a;
        this.b = b;
    }
}

// ❌ Forbidden
public class MyService {
    @Autowired private DependencyA a;   // field injection
}
```

All injected fields must be `final`. No `@Autowired` or `@Inject` on fields.

---

### 2. No Hardcoded Secrets

```yaml
# ✅ Correct
spring:
  datasource:
    password: ${POSTGRES_PASSWORD:changeme}

# ❌ Forbidden
spring:
  datasource:
    password: mysecretpassword
```

---

### 3. SLF4J with Parameterised Messages

```java
// ✅ Correct
log.debug("Saved memory id={} userId={} type={}", memory.id(), memory.userId(), memory.type());

// ❌ Forbidden
System.out.println("Saved memory " + memory.id());
```

---

### 4. SOLID Design Principles

- **S** — Single Responsibility: one class, one reason to change
- **O** — Open/Closed: extend via new classes, not by editing existing ones
- **L** — Liskov Substitution: subtypes honour their base type's contract
- **I** — Interface Segregation: small focused interfaces
- **D** — Dependency Inversion: depend on port interfaces, not concrete adapters

---

### 5. DDD Bounded Contexts

Cross-module calls go through **port interfaces only**. No module imports another module's internal implementation.

```text
core-api → PersonalMemoryStore (port)
                └── PGVectorPersonalMemoryStore (adapter — never directly imported by core-api)
```

---

### 6. Explicit Column Lists in SQL

```sql
-- ✅ Correct
SELECT id, user_id, memory_type, content, strength, access_count,
       created_at, last_accessed_at
FROM personal_memories WHERE user_id = :userId;

-- ❌ Forbidden
SELECT * FROM personal_memories WHERE user_id = :userId;
```

---

### 7. Parameterised Queries Only

```java
// ✅ Correct
var sql = "SELECT id FROM tenants WHERE api_key_hash = :hash";
jdbc.queryForObject(sql, new MapSqlParameterSource("hash", hash), String.class);

// ❌ Forbidden — SQL injection risk
var sql = "SELECT id FROM tenants WHERE api_key_hash = '" + hash + "'";
```

Use `NamedParameterJdbcTemplate` throughout.

---

### 8. Conventional Commits

Format: `type(scope): description`

| Type | When to use |
|---|---|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `chore` | Build, CI, tooling |
| `build` | Maven, Docker, Helm |
| `test` | Adding or fixing tests |
| `refactor` | No external behaviour change |

---

### 9. No `// TODO` in Committed Code

If it is not done, do not commit it.

---

### 10. `jakarta.*` Exclusively

Spring Boot 3.x uses `jakarta.*`. Any `javax.*` import is a build-breaking error.

---

## Java 21 Patterns

### Records for Immutable Domain Types

```java
public record PersonalMemory(UUID id, String userId, MemoryType type,
        String content, double strength, int accessCount,
        Instant createdAt, Instant lastAccessedAt) {
    public PersonalMemory {
        if (content == null || content.isBlank()) throw new IllegalArgumentException("content required");
    }
}
```

### Sealed Interfaces for Domain Events

```java
public sealed interface DomainEvent
    permits MemoryStoredEvent, SessionCreatedEvent, GdprErasureEvent {
    Instant occurredAt();
}
```

### Optional for Nullable Returns — Never Null

```java
Optional<CognitiveSession> findActiveSession(String userId, String tenantId);
```

---

## Test Standards

| Requirement | Rule |
|---|---|
| Coverage gate | 80% line coverage (JaCoCo) |
| Integration tests | Testcontainers with real database |
| No `Thread.sleep()` | Use Awaitility or Testcontainers wait strategies |
| No empty `catch` blocks | Always log or rethrow |
| No `Optional.get()` without guard | Use `orElseThrow()` or `orElse()` |

---

## Prohibited Patterns

| Pattern | Reason |
|---|---|
| `javax.*` imports | Spring Boot 3.x uses `jakarta.*` |
| Field `@Autowired` | Hides dependencies, breaks testability |
| `SELECT *` in SQL | Schema changes break queries silently |
| Hardcoded secrets | Security violation |
| `Thread.sleep()` in tests | Flaky tests |
| Empty `catch {}` blocks | Silent error swallowing |
| `Optional.get()` without guard | `NoSuchElementException` at runtime |
| Direct commits to `main` | All changes via pull request |
| `System.out.println()` | Use SLF4J logger |
