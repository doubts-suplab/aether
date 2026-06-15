# Aether in Healthcare

> Domain intelligence target: `aether-health`
> AIEL Phase: Concept — Intelligence Brief to be completed before any design begins

---

## The Opportunity

Healthcare is the domain where cognitive AI has the highest potential and the highest stakes simultaneously. Clinical decisions affect patient outcomes directly. Documentation burden is a leading cause of physician burnout. Care coordination across specialists, primary care, and patients is fragmented. And the regulatory environment — HIPAA, MDR, clinical evidence standards — demands explainability and auditability that most AI systems cannot provide.

Aether's architecture addresses the core challenge: healthcare AI needs persistent context (patient history, care preferences, prior interactions), governed decision-making (clinical decisions require human oversight), and full audit trails (every recommendation must be defensible). These are not optional features — they are the minimum viable requirements for clinical deployment.

---

## Target Use Cases for `aether-health`

### Clinical Documentation Agent
Listens to or reads clinical encounter notes and generates structured documentation in the required format (SOAP, DAP, or specialty-specific templates). Learns the physician's documentation style over time — the PROCEDURAL memory type captures preferences across encounters. Reduces documentation time without reducing clinical specificity.

### Care Coordination Agent
Maintains a longitudinal care plan across specialties, tracking referrals, pending results, follow-up actions, and care gaps. When a patient sees a specialist, the coordination agent ensures the primary care context (current medications, recent labs, active care plan) is available. When test results return, the agent routes them to the right clinician with the relevant patient context.

### Clinical Decision Support Agent
Surfaces evidence-based recommendations at the point of care — drug interactions, guideline-based care gaps, diagnostic differentials. Critically: the confidence gate ensures that any recommendation where the system is uncertain routes to the physician as a suggestion, never as a directive. The physician remains the decision-maker.

### Patient Communication Agent
Drafts after-visit summaries, medication instructions, and follow-up reminders in plain language. Adapts communication style to the patient's health literacy level (stored in PROCEDURAL memory from prior interactions). Generates communications in the patient's preferred language.

### Prior Authorization Agent
Navigates payer prior authorization requirements for procedures and medications, assembling the clinical documentation that payers require and tracking authorization status. Flags cases where authorization is unlikely based on historical payer decisions.

---

## Regulatory and Ethical Constraints

Healthcare AI carries additional obligations beyond standard enterprise governance:

**Human oversight is not optional.** In most jurisdictions, AI systems that influence clinical decisions must have a licensed clinician in the decision loop. The Aether confidence gate aligns with this requirement — decisions below the threshold never auto-enforce. In healthcare deployments, the confidence threshold for clinical recommendations may need to be set higher than the ecosystem default of 0.8.

**Explainability is a regulatory requirement.** Where MDR (EU Medical Device Regulation) or FDA guidance applies to AI-assisted clinical tools, the basis for recommendations must be auditable. Aether's immutable audit trail and mandatory reason field in every AgentOutput provide the foundation — domain-specific regulatory counsel must confirm adequacy.

**Data residency is paramount.** Patient health information is subject to jurisdiction-specific data residency requirements. AetherCore deployments for healthcare must be configured with appropriate data residency constraints confirmed before any patient data is stored.

**HIPAA minimum necessary standard.** Personal context assembled by AetherCore must only include the minimum information necessary for the clinical task at hand. Broad context retrieval appropriate for a general cognitive assistant may need to be scoped more narrowly in clinical contexts.

---

## AIEL Phase 1 Prerequisites

The Intelligence Brief for `aether-health` must address:

- [ ] Clinical scope — which decision types the system will support and which it explicitly will not
- [ ] Regulatory classification — is this a medical device under applicable law?
- [ ] Confidence thresholds per decision type — clinical recommendations may require 0.9+ rather than 0.8
- [ ] Data residency and HIPAA compliance approach
- [ ] Integration points — EHR systems (Epic, Cerner, Oracle Health), FHIR APIs, lab systems
- [ ] Clinician override and feedback mechanisms
