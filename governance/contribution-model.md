# Contribution Model

> How changes are proposed, reviewed, and integrated across the Aether ecosystem.
> This model applies to all repositories under `suplab/aether-*`.

---

## The Core Rule: Documentation First

No significant change begins with code. Every non-trivial change — a new capability, an architectural shift, a new agent, a cross-repository integration — begins with a written artifact that describes the change, its rationale, and its consequences.

This is not bureaucracy. It is how the ecosystem maintains coherence as it grows. Code is easy to write and hard to understand. A well-written proposal or ADR is easy to understand and hard to misinterpret.

---

## Change Categories

### Category A — Routine Implementation
Bug fixes, small feature completions, documentation updates, and test additions within a single repository. No proposal or ADR required.

- Open a pull request against `main` with a conventional commit message
- At least one reviewer approval required
- CI must pass (build, test, JaCoCo coverage ≥ 80%, OWASP scan)

### Category B — Significant In-Repository Change
A new module, a new agent, a new API endpoint set, a Flyway migration, or a change to the memory schema. Requires an Architecture Decision Record before implementation begins.

Process:
1. Write an ADR using the template at `templates/adr-template.md`
2. Open a draft pull request with the ADR as the sole content
3. Get ADR reviewed and approved by the repository maintainer
4. Implement the change, referencing the ADR in commit messages
5. Open the implementation pull request (CI must pass)

### Category C — Cross-Repository Change
Any change that affects the public API contract of a repository, changes how two repositories integrate, or alters a shared standard in `suplab/aether`. Requires a formal RFC in addition to ADRs in each affected repository.

Process:
1. Write a Proposal using `templates/proposal-template.md`
2. Open the proposal as an issue or draft PR in `suplab/aether`
3. Tag all affected repository maintainers for review
4. Allow a minimum 5 business day review window
5. Resolve all substantive objections before proceeding
6. Write ADRs in each affected repository
7. Coordinate the implementation order (lower layers first)
8. Deploy with a coordinated release sequence

### Category D — Governance Change
Changes to the confidence gate threshold, AIEL methodology phases, IMM level definitions, or ecosystem-wide standards. Requires Governance Officer approval in addition to the RFC process.

These changes are rare and consequential. The Governance Officer's approval is not advisory — it is required. If the Governance Officer vetoes a change, the veto escalates to programme leadership, not back to the team.

---

## Repository Ownership

Every repository has:

| Role | Responsibility |
|---|---|
| Maintainer | Owns the repository's direction. Final approval on all PRs. Writes the roadmap. |
| Reviewer | Reviews code and documentation PRs. Can approve Category A changes independently. |
| Governance Officer | Approves all Phase 7 Evaluation sign-offs and Evolution Strategies. Holds veto on Category D changes. |

Maintainers and Governance Officers are named in each repository's `CLAUDE.md`.

---

## Commit Standards

All commits must follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

Allowed types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `perf`, `build`, `ci`

Examples:
```
feat(memory): add reinforce-on-read to PGVectorPersonalMemoryStore
fix(gdpr): ensure session erasure is included in hard-delete cascade
docs(adr): ADR-004 — decision to use Kafka for Grid-Core feedback
refactor(agent): extract confidence gate into AgentOutput builder
```

Commits with TODO, FIXME, or hardcoded secrets fail the CI quality gate.

---

## Pull Request Requirements

All pull requests must:
- Have a clear title that describes what changed, not how
- Reference the ADR or proposal if one exists (`ADR-NNN` in the PR description)
- Pass CI (build, test, coverage, OWASP scan)
- Include updated documentation if the change affects public contracts or README
- Update `docs/roadmap.md` if a phase is completed
- Update `docs/progress.md` if in-sprint work is being tracked

---

## Architectural Decision Records

Every significant architectural decision — why a technology was chosen, why a pattern was adopted, why an alternative was rejected — must be recorded in an ADR. ADRs live in `docs/adr/` within the relevant repository, or in `aether/docs/adr/` for ecosystem-wide decisions.

ADRs are immutable once approved. If a decision is reversed, a new ADR is written that supersedes the original. The original is never deleted — it is marked as superseded with a reference to the new ADR.

Use `templates/adr-template.md` to create new ADRs.

---

## Branching Strategy

| Branch | Purpose |
|---|---|
| `main` | Always deployable. Protected. Requires PR + CI pass. |
| `feature/{description}` | Feature branches. Short-lived. Merge to main via PR. |
| `release/{version}` | Release candidates. Triggers Docker build and GHCR push. |
| `hotfix/{description}` | Emergency fixes. Merge directly to main with expedited review. |

There are no long-lived development or staging branches. All work is merged to `main` frequently via small, complete pull requests.
