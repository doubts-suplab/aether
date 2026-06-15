# Research References

> Papers and publications relevant to the design of the Aether ecosystem.
> Organized by topic.

---

## Published Work

### AetherGrid

> **AetherGrid: A Distributed Cognitive-Orchestration Framework Linking Sensory-Emotional Memory Encoding (AetherCore) with Autonomous Agent Governance**
> Zenodo Preprint · doi:[10.5281/zenodo.20394197](https://doi.org/10.5281/zenodo.20394197)

The foundational paper describing the AetherGrid architecture, its relationship to AetherCore, and the cognitive-orchestration model that underlies the ecosystem.

---

## Memory Systems

**Episodic Memory and the Brain** — Endel Tulving. The original framework distinguishing episodic memory (time-anchored experiences) from semantic memory (general knowledge). The foundation for Aether's four-type memory model.

**The Spacing Effect** — Ebbinghaus's forgetting curve and subsequent research on spaced repetition. Informs the reinforce-on-read and temporal decay design in AetherCore. Memories accessed frequently retain strength; memories not accessed decay — mirroring empirical findings on memory retention in humans.

**Memory Consolidation** — Research on how episodic memories consolidate into semantic memories during sleep and reflection. Relevant to the planned AetherCore Reflection Engine (Phase 8) and its consolidation mechanism.

---

## Emotional Intelligence and Affective Computing

**Descartes' Error** — Antonio Damasio. The somatic marker hypothesis: human decision-making is inseparable from emotional context. Emotions are not noise in the decision-making process — they are signal. This is the theoretical basis for Aether's EMOTIONAL memory type and the `emotionalState` field in `CognitiveSession`.

**Affective Computing** — Rosalind Picard. The academic foundation for building systems that recognize, interpret, and simulate human emotions. Informs the design philosophy for emotional context in Aether — not to simulate emotion, but to understand and respond appropriately to human emotional states.

---

## Multi-Agent Systems

**ReAct: Synergizing Reasoning and Acting in Language Models** — Yao et al. (2022). The ReAct pattern — interleaving reasoning traces and action steps — is foundational to AetherGrid's agent execution model. `AgentInput` and `AgentOutput` in the agent contract embody this reasoning + action structure.

**Toolformer** — Schick et al. (2023). Language models that learn to use tools autonomously. Relevant to the MCP Runtime design in AetherGrid and the tool access model in Agent Contracts.

**Constitutional AI** — Anthropic. The principle of defining agent behavior through explicit principles rather than RLHF alone. Informs Aether's approach to Agent Contracts and the AIEL Phase 5 artifact structure.

---

## Cognitive Architectures

**ACT-R** — John Anderson. A cognitive architecture from cognitive science that models human cognition through declarative and procedural memory. The distinction maps onto Aether's SEMANTIC and PROCEDURAL memory types.

**SOAR** — Laird, Newell, Rosenbloom. A general cognitive architecture emphasizing problem-solving through a working memory and long-term knowledge base. Relevant to the design of AetherCore's context assembly and the planned Reasoning Engine.

---

## Distributed Systems and Governance

**The Byzantine Generals Problem** — Lamport, Shostak, Pease. Consensus in distributed systems under faulty or adversarial conditions. Relevant to Phase 5 (AetherMesh) federation design — how multiple instances agree on shared knowledge without requiring a central authority.

**Differential Privacy** — Dwork et al. Mathematical framework for sharing statistical information about a population without revealing individual data. Candidate approach for privacy-preserving knowledge federation in AetherMesh.
