# Cognitive Architecture Research

> This document captures the research questions and design principles that inform how Aether models cognition.
> It is a living document — answers evolve as the system matures and real usage data accumulates.

---

## What We Mean by Cognitive Architecture

A cognitive architecture is the fixed structure that underlies a system's intelligence — the mechanisms by which it perceives, remembers, reasons, and acts, independent of any particular domain or task. In AI systems, cognitive architecture choices determine what the system can learn, how it represents knowledge, and what kinds of reasoning it can perform.

Aether is opinionated about cognitive architecture in ways that distinguish it from general-purpose LLM wrappers. Where most AI systems delegate all cognitive work to an LLM at inference time, Aether builds persistent cognitive structure in the database — memory that accumulates over time, relationships that are explicitly maintained, and emotional context that shapes how information is weighted.

---

## Memory Systems

### The Four-Type Model

Aether models memory as four distinct types, each serving a different cognitive function:

**Episodic memory** encodes specific events with their temporal context. "I presented to the board on 12 June" is episodic — it is bound to a specific time and place. Episodic memories are highly specific and decay quickly if not reinforced. They are the most human-like form of memory and the hardest to replicate in AI systems.

**Semantic memory** encodes general knowledge — facts that are true regardless of when they were learned. "PostgreSQL supports partial indexes" is semantic. Semantic memories are more stable than episodic ones and form the basis for reasoning about the world.

**Procedural memory** encodes how things are done — learned skills, preferred workflows, and tacit knowledge. "I prefer concise bullet-point summaries over long paragraphs" is procedural. These memories shape behavior without necessarily being consciously accessed.

**Emotional memory** encodes affective states and their context. "I felt energised after completing the sprint" is emotional. These memories influence how future similar situations are processed — a finding consistent with how human emotional memory modulates decision-making.

### Reinforcement and Decay

The fundamental insight behind Aether's memory model is that not all memories are equal, and relevance changes over time. The strength model addresses this:

- Every memory has a `strength` value between 0.0 and 1.0
- **Reinforce-on-read:** each retrieval adds 0.1 to strength (capped at 1.0). Memories that are repeatedly relevant grow stronger.
- **Temporal decay:** memories that are not accessed lose strength over time (5% of current strength per day of inactivity by default)
- **Purge threshold:** memories below 0.05 strength are hard-deleted. The system naturally forgets what it no longer uses.

This mirrors the psychological finding that memory strength is a function of recency and frequency of access — the spacing effect in human learning.

### Open Questions

**How should decay rates differ by memory type?** Episodic memories may warrant faster decay than semantic ones, since specific events become less decision-relevant over time while general knowledge stays useful. The current implementation uses a single decay rate. A per-type decay model is a candidate for Phase 7/8 of AetherCore.

**How should memory consolidation work?** In human memory, experiences are consolidated during sleep — episodic memories become semantic over time as details fade and patterns are retained. AetherCore's Reflection Engine (planned) will need a consolidation mechanism. The design question is: what triggers consolidation, and what is the consolidation rule?

**How should competing memories be resolved?** If a user has stored conflicting semantic memories (two different answers to the same factual question), how should the system handle retrieval? Current behavior returns the top-K by cosine similarity without conflict detection.

---

## Emotional Context

Emotions are not ornamental in Aether's cognitive model. They are signals that modulate how information is processed and weighted.

The research basis for this is the somatic marker hypothesis (Damasio) and broader work in affective computing: human decision-making is not separable from emotional state. A technically optimal decision made in a context of high stress may be worse than a slightly suboptimal decision made in a calm, high-confidence state.

Aether captures emotional context through the `EMOTIONAL` memory type and through `emotionalState` and `engagementScore` fields in `CognitiveSession`. These are currently stored as strings (user-reported or agent-inferred) rather than structured representations.

**Open Questions:**

**How should emotional context influence memory retrieval?** Should a user in a high-stress state retrieve memories differently than in a calm state — perhaps with more emphasis on reliable, high-strength memories rather than novel associations? This is unexplored in the current implementation.

**How should emotional state be inferred rather than reported?** Asking users to self-report emotional state is unreliable. Signal inference from language patterns, response latency, and session behavior is the direction — but this requires LLM integration in AetherCore that does not yet exist.

---

## Reflection

Reflection is the capacity to evaluate one's own prior reasoning — to ask not just "what do I know?" but "how well did my past beliefs serve me?" In human cognition, reflection is associated with metacognition and is linked to learning efficiency.

In Aether, reflection is planned as a scheduled process that runs over accumulated memories and session history:

- Identify patterns across episodic memories ("I tend to underestimate tasks involving infrastructure")
- Surface implicit procedural memories that should be made explicit
- Flag contradictions between stored semantic memories
- Generate learning insights that feed into the Learning Engineering phase (AIEL Phase 10)

The reflection engine does not exist yet. It is a candidate for AetherCore Phase 8.

---

## Meta-Reasoning

Meta-reasoning is reasoning about how to reason — deciding which cognitive strategy to apply to a problem, given constraints on time, confidence, and available information.

In the current AetherGrid agent model, meta-reasoning is implicit: the `AgentFilterChain` always runs all agents sequentially. A meta-reasoning layer would allow the system to:

- Skip agents that are unlikely to add signal for a given request type (fast-path design)
- Decide when to invoke LLM reasoning versus when to return from fast-path
- Route requests to specialized sub-pipelines based on intent classification
- Manage latency budgets dynamically

The fast-path pattern in the `Agent` interface is a first step toward meta-reasoning: `execute()` returns early when confidence is high enough without LLM invocation. A more sophisticated meta-reasoning layer is a Grid evolution target.

---

## Collective Intelligence

The long-term vision for Aether includes multiple instances working together — sharing knowledge, learning from each other's experiences, and reasoning across organizational boundaries. This raises fundamentally different design questions from single-instance cognition.

**Knowledge aggregation without overfitting.** If ten Aether instances share their memories, the shared knowledge base will reflect the collective experience — but it may also amplify shared biases. How do we aggregate usefully without homogenizing?

**Privacy-preserving federation.** Personal memories cannot be shared across organizational boundaries without explicit consent. The federation design must share knowledge at an abstraction level that does not expose raw personal content. Embedding-level sharing (sharing vectors, not text) is the leading candidate.

**Emergent capability.** The most interesting question is whether collective intelligence produces capabilities that no individual instance has. Can a mesh of Aether instances discover patterns across domains that no single instance could see? This is the Phase 5 research frontier.
