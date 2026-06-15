# Agent Design Standard

> Applies to all agents across AetherGrid, AetherCore, and domain products.
> These rules are enforced by the `ai-governance-officer` agent in every session.

---

## Agent Anatomy

Every Aether agent implements the `Agent` interface:

```java
public interface Agent {
    String name();
    Set<AgentCapability> capabilities();
    AgentOutput execute(AgentInput input);
}
```

An agent must:
- Declare exactly the capabilities it serves
- Return a decision with a confidence score (0.0–1.0)
- Include a human-readable rationale
- Never throw — return `ALLOW` or `DEFER` on any internal error

---

## Confidence Gate (Non-Negotiable)

```
confidence >= 0.8  →  decision auto-enforced
confidence < 0.8   →  autoEnforced = false → human review required
```

**A `BLOCK` decision with confidence < 0.8 must never auto-block.** The `AgentDecision` record carries `autoEnforced: false` and the orchestrator surfaces it for human-in-the-loop review.

This is the most important safety constraint in the entire system.

---

## Decision Hierarchy

When multiple agents run in parallel, the orchestrator resolves conflicts by severity:

```
BLOCK > RATE_LIMIT > ALERT > SUGGEST > DEFER > ALLOW
```

The highest-severity decision with `confidence >= 0.8` wins. Lower-confidence decisions are downgraded to human review regardless of their action.

---

## Fast-Path Design

Agents must compute a fast answer when LLM inference is not required:

```java
// ✅ Fast-path: zero memories → DEFER without LLM call
if (episodicCount == 0 && semanticCount == 0) {
    return AgentOutput.of(AgentDecision.defer("No memory baseline — deferring"));
}
// Only invoke LLM when fast-path is insufficient
return llm.chat(buildPrompt(memories));
```

LLM calls are expensive. Every agent must exhaust rule-based fast-paths before invoking the model.

---

## LLM Protocol

All agents that invoke LLMs must:

1. Call via `LlmClient` interface — never direct HTTP
2. Request structured JSON responses with explicit field contracts
3. Parse and validate the response; fall back to a safe default on parse error
4. Log the raw LLM response at `DEBUG` level for observability

```java
var prompt = """
    Analyse the following API call patterns and return JSON:
    {"decision": "ALLOW|BLOCK|ALERT|SUGGEST|DEFER", "confidence": 0.0-1.0, "rationale": "..."}
    Patterns: %s
    """.formatted(patterns);

var raw = llmClient.chat(LlmRequest.of(prompt));
return parseDecision(raw).orElse(AgentDecision.allow("LLM parse failed — defaulting to ALLOW"));
```

---

## Feedback Loop

Every agent decision should eventually receive a feedback signal:

- `CORRECT` — the decision was validated as right
- `INCORRECT` — the decision was overridden or wrong
- `PARTIALLY_CORRECT` — the decision was partially right
- `UNKNOWN` — no outcome observed

Feedback is stored in `agent_feedback` and consumed by the `SelfImprovingAgent` for weekly learning cycles.

---

## Agent Registration

Agents are discovered automatically via Spring's `List<Agent>` injection:

```java
@Bean
public AgentRegistry agentRegistry(List<Agent> agents) {
    return new AgentRegistry(agents);
}
```

All agents must be registered as Spring `@Bean`s in the relevant `@Configuration` class. Kill-switches (`disableAgent(name)`) are available on `AgentRegistry` for runtime deactivation.

---

## Agent Types (Current)

| Agent | Capability | Purpose |
|---|---|---|
| `GovernanceAgent` | `GOVERNANCE` | Policy compliance, LLM-based decision |
| `RetryAgent` | `RETRY_ANALYSIS` | Failure pattern detection, backoff recommendation |
| `HallucinationDetectorAgent` | `HALLUCINATION_DETECTION` | LLM output validation against memory |
| `TemporalPredictionAgent` | `TEMPORAL_PREDICTION` | Future failure prediction from memory history |
| `ReflectionAgent` | `REFLECTION` | Procedural health scoring, self-assessment |
| `SelfImprovingAgent` | `SELF_IMPROVEMENT` | Weekly feedback analysis and suggestion |
| `AetherCoreBridgeAgent` | `PERSONAL_CONTEXT` | Enriches inputs with personal memory from Core |

---

## Naming Conventions

- Agent class: `{Function}Agent` (e.g., `GovernanceAgent`, `RetryAgent`)
- Capability enum value: `SCREAMING_SNAKE_CASE` matching the function
- Agent name string: `kebab-case` matching the class function (e.g., `"governance"`, `"retry-analysis"`)
- Decision rationale: plain English sentence, present tense (e.g., `"Failure rate exceeds threshold of 50%"`)

---

## Testing Requirements

Every agent must have unit tests covering:

1. Fast-path — zero/minimal data, no LLM invoked
2. LLM-path — mocked LLM response, decision parsed correctly
3. LLM failure — parse error returns safe default (ALLOW or DEFER)
4. Confidence gate — low-confidence BLOCK produces `autoEnforced=false`
5. Capability declaration — `capabilities()` returns the expected set
