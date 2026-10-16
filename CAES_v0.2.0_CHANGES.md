# CAES v0.2.0 — Changes and Rationale
**Standards Hardening Pass: v0.1.1 → v0.2.0**

---

## Changes Made

### 1. Core Principles — Added Behavioral Outputs Line
**Change:** Added: *"Governance applies to any output capable of producing consequential effect, including behavioral outputs."*

**Rationale:** v0.1.1 scoped governance implicitly to execution actions. AI systems produce consequential effects through language, instruction, and interaction that do not involve system mutations. Without explicit inclusion of behavioral outputs, a system could deny governance obligations over its most prevalent effect vector.

---

### 2. Scope — Expanded Effect Boundary Coverage
**Change:** Added `BehavioralInfluence` and `SensoryCapture` to the Scope section's example list.

**Rationale:** Aligns the intro scope statement with the new normative Effect Boundary Definition section. Ensures readers understand behavioral and recording capabilities are in scope before reaching the detailed section.

---

### 3. NEW: Effect Boundary Definition
**Change:** Added a complete normative section defining seven Effect Boundary categories with a table and rules for extension and narrowing.

**Rationale:** v0.1.1 referenced Effect Boundaries without defining them. Undefined scope enables selective compliance — a system could claim conformance while excluding entire effect classes. The new section closes that gap. Implementations MUST NOT narrow the listed categories.

---

### 4. NEW: Fail-Closed Semantics
**Change:** Added a dedicated section defining fail-closed operationally with five testable conditions and explicit prohibition on silent degradation.

**Rationale:** v0.1.1 listed "fail-closed enforcement" as a principle without defining what fail-closed means operationally. An implementation that degrades gracefully — permitting execution when receipt verification is unavailable — could claim fail-closed while providing no actual constraint guarantee. The new section defines it precisely and requires that failures produce recorded denial events.

---

### 5. NEW: Authorization-Execution Separation
**Change:** Added a dedicated section making the temporal ordering of Decide → Execute explicit, including write-then-verify persistence and prohibition on post-hoc receipt production.

**Rationale:** v0.1.1 described the concept but did not specify temporal ordering requirements, write-then-verify semantics, or the prohibition on receipt reconstruction after the fact. Without these requirements, a system could produce receipts concurrent with or after execution and still claim conformance. The new section eliminates that loophole.

---

### 6. NEW: Decision Dispositions
**Change:** Added a normative section defining four required dispositions: Approved, Modified, Denied, RequiresHumanAuthorization. Added per-disposition requirements.

**Rationale:** v0.1.1 implied governance decisions but never enumerated the required outcome states. Without a defined disposition set, implementations could omit escalation paths, skip denial receipts, or apply silent defaults. The new section makes the complete disposition set normative and requires explicit naming for every outcome.

---

### 7. Core Primitives — Decision Receipt: Strengthened
**Change:** Rewrote Decision Receipt specification to require: pre-execution timing, named signing algorithm, PolicyHash binding, unique action scope binding, write-then-verify persistence, Governed Spine entry, and equal requirements for Denial Receipts.

**Rationale:** v0.1.1 defined a receipt as "a cryptographically signed authorization artifact" without specifying when it must be produced, how it must be persisted, whether denials require receipts, or how it binds to the specific action being authorized. Each omission creates a surface for superficial compliance.

---

### 8. Core Primitives — PolicyHash: Strengthened
**Change:** Rewrote PolicyHash specification to require: named canonicalization method, SHA-256 minimum, evaluation-time computation, immutability after issuance, receipt embedding (not pointer), and offline recomputability.

**Rationale:** v0.1.1 described PolicyHash as "a deterministic hash of the canonical policy state" without specifying the canonicalization method, hash algorithm, or how binding is verified. An implementation using a version label as PolicyHash, or recomputing at execution time, would satisfy the v0.1.1 language while providing no meaningful tamper evidence.

---

### 9. Core Primitives — Governed Spine: Strengthened
**Change:** Rewrote Governed Spine specification to require: append-only with explicit mutation prohibition, platform-assigned ordering, tenant isolation as structural property, fail-closed on append failure, canonical platform-assigned IDs, causal linkage field, and Dead Letter mechanism.

**Rationale:** v0.1.1 defined the Governed Spine as "append-only, causally ordered, tenant-scoped" without specifying enforcement mechanisms. A log database with a soft-delete field and application-enforced ordering would technically satisfy the v0.1.1 language. The new specification requires structural enforcement, not policy enforcement.

---

### 10. Compliance Levels — All Three: Strengthened with Testable Requirements
**Change:** Rewrote all three compliance levels to include explicit testable requirement tables. Added Level 1 exclusions (what is NOT required at Level 1). Added Level 3 chaos mode enumeration, structured error code requirement, and complete artifact coverage list.

**Rationale:** v0.1.1 compliance levels were narrative descriptions. A system could claim any level by asserting alignment with the narrative. Testable requirements tables make conformance falsifiable — an auditor can verify or refute each row independently.

---

### 11. Non-Compliant Patterns — Expanded
**Change:** Added categories for Behavioral Output Failures and Recording/Privacy Failures. Tightened language on existing patterns (e.g., PolicyHash as version label, post-hoc receipt production).

**Rationale:** The original list omitted the two largest new governance domains introduced in v0.2.0. Non-compliant patterns must mirror the normative requirements to be effective.

---

### 12. NEW: Behavioral Governance
**Change:** Added complete normative section covering behavioral effect boundary, policy requirements, pre-emission receipt requirement, enforcement model (pre-delivery + fail-closed), behavioral dispositions, and causal recording requirements.

**Rationale:** AI systems produce the majority of their consequential effects through behavioral outputs — language, instructions, recommendations. Governance that applies only to system mutations leaves the primary effect vector ungoverned. This section extends the CAES primitives to cover behavioral output while remaining implementation-agnostic.

---

### 13. NEW: Privacy, Recording, and Data Stewardship
**Change:** Added complete normative section covering sensory effect boundaries, consent and authorization requirements, use and purpose limitation, retention enforcement, deletion as governed action with permanent receipt, recording dispositions, and enforcement timing.

**Rationale:** AI systems with sensing capabilities (audio, video, environmental, inferential) can produce irreversible privacy harms before any execution action occurs. CAES without recording governance creates a governance gap for embodied, ambient, and multimodal AI systems. This section extends accountability to observation and capture as first-class governed operations.

---

### 14. Conformance Claims — Strengthened
**Change:** Replaced narrative list with a complete required-field table. Added: signing algorithm, canonicalization method, write-then-verify mechanism, chaos attestation coverage, and error code namespace as required fields. Added third-party audit recommendation for Level 3.

**Rationale:** v0.1.1 Conformance Statement requirements were insufficient to distinguish meaningful from superficial claims. The new table makes the minimum disclosure set unambiguous.

---

## What Was Not Changed

- Why CAES Exists — unchanged; rationale remains accurate
- Reference Implementation section — unchanged
- Working Group, Amendment Process, Mark Usage, License — unchanged
- The three-primitive structure is preserved; primitives are strengthened, not replaced
- No Keon-specific terminology was introduced
- No internal architecture was referenced
- The standard remains implementation-agnostic

---

*CAES Working Group · v0.2.0 Hardening Pass · March 2026*
