# Repository Guidelines

> Every Aether repository must follow this structure. Consistency enables onboarding,
> tooling, and cross-repo navigation.

---

## Required Files (Every Repository)

```
{repo-name}/
├── README.md                    ← Entry point: what it is, how to run it, key APIs
├── CLAUDE.md                    ← AI session brief: tech stack, golden rules, commands
├── pom.xml                      ← Parent POM (if Java)
├── .gitignore
├── docs/
│   ├── index.html               ← Visual concept page (dark theme, cross-linked)
│   ├── architecture.md          ← Technical deep-dive
│   ├── progress.md              ← Live development tracker (updated every commit)
│   └── roadmap.md               ← Phased delivery plan
├── .claude/
│   ├── agents/                  ← Specialist agent definitions
│   ├── commands/                ← Slash commands (/estimate, /review, /adr, etc.)
│   └── memory/                  ← Session memory files
└── .github/
    └── workflows/               ← CI/CD pipelines
```

---

## README Structure

Every `README.md` must contain:

1. **One-line description** — what the repo does
2. **Ecosystem position** — table showing this repo's place in Aether
3. **Quick Start** — commands to run locally (`docker compose up` + `mvn spring-boot:run`)
4. **Module table** — module names and their purpose
5. **Key API Endpoints** — the most important REST endpoints
6. **Configuration** — environment variables table with defaults
7. **Ecosystem links** — sister repos and their relationship

---

## CLAUDE.md Structure

Every `CLAUDE.md` must contain:

1. **What this project is** — 2–3 sentence description + ecosystem navigation table
2. **Technology stack** — complete table of all technologies used
3. **Bounded context** — package root, ports, Kafka topics
4. **Pre-coding checklist** — questions to answer before writing any code
5. **Ten Golden Rules** — non-negotiable rules (same across all repos, with repo-specific additions)
6. **Slash commands** — available `/commands`
7. **Memory files** — index of `.claude/memory/` contents
8. **Prohibited patterns** — explicit list of forbidden patterns
9. **Documentation sync rule** — which files to update on every commit

---

## EEIK Bootstrap

Every new repository must run the EEIK bootstrap as its **first commit**. The bootstrap provides:

- `CLAUDE.md` — full project brief
- `.claude/memory/` — 7 seed memory files:
  - `project-context.md` — service inventory, ports, environments
  - `domain-glossary.md` — project-specific terms
  - `decisions.md` — architectural decisions log
  - `constraints.md` — golden rules + project-specific constraints
  - `patterns.md` — approved patterns with code examples
  - `tech-debt.md` — debt register (initially empty)
  - `session-log.md` — rolling session log
- `.claude/agents/` — specialist agent definitions
- `.claude/commands/` — slash commands
- `.claude/hooks/` — safety hooks
- `aether.manifest.yaml` — EEIK project manifest

Commit message: `chore(bootstrap): integrate eeik governance layer`

---

## Module Structure (Java)

```
{project-name}-parent/
├── {project}-domain/       ← Pure domain: records, enums, port interfaces (no Spring)
├── {project}-memory/       ← Persistence adapters: pgvector, JDBC, Kafka consumers
├── {project}-api/          ← Spring Boot application: controllers, config, Flyway
└── {project}-infra/        ← Docker Compose, K8s manifests, Helm chart
```

Rules:
- `domain` module has zero Spring dependencies
- `memory` module implements port interfaces from `domain`
- `api` module wires everything via `@Configuration` classes
- `infra` module has no Java source files

---

## Dependency Direction (Strict)

```
{project}-api
    ├── {project}-domain
    └── {project}-memory
          └── {project}-domain

{project}-infra  (no Java)
```

No circular dependencies. Enforce with Maven Enforcer plugin.

---

## Database Migration Rules

- All migrations via Flyway
- Forward-only — no rollback migrations
- Migration files in `core-api/src/main/resources/db/migration/`
- Naming: `V{version}__{description}.sql` (double underscore)
- Never modify an already-applied migration
- Test migrations with Testcontainers in CI

---

## Branch and Commit Strategy

- `main` — protected, requires PR + CI pass
- Feature branches: `feat/{description}` or `claude/{generated-name}`
- Commits follow Conventional Commits format
- Each phase completes in one clean commit (or a small set of related commits)
- Never commit directly to `main`

---

## Documentation Sync Rule

Every commit that changes system behaviour must update:

| File | Update when |
|---|---|
| `docs/progress.md` | Any new feature or phase completion |
| `README.md` | Architecture or scope changes |
| `docs/index.html` | Tech stack or concept changes |
| `docs/roadmap.md` | Milestone shifts |
| `docs/architecture.md` | Architectural decisions change |

The HTML page mirrors the README — they must stay in sync.

---

## CI/CD Requirements

Every repository must have:

| Workflow | Trigger | Purpose |
|---|---|---|
| `ci.yml` | Push to any branch | Build + test + JaCoCo coverage |
| `quality-gate.yml` | PR to `main` | Checkstyle + OWASP dependency-check |
| `docker-build.yml` | Push to `main` or `release/**` | Multi-arch Docker build + GHCR push |

Use OIDC for all cloud authentication. No static secrets in CI.
