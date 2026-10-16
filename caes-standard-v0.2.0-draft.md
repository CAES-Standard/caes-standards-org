# CAES
## Constitutional AI Execution Standard

**Version:** v0.2.0 (Draft)
**Status:** Public Review
**Working Group:** CAES-WG
**Category:** Normative Specification
**Supersedes:** v0.1.1

---

## Overview

CAES (Constitutional AI Execution Standard) defines the minimum structural requirements for AI systems that execute actions with legal, financial, regulatory, or operational consequence.

A CAES-conformant system does not merely log what happened.
It proves — before execution occurs — that the action was authorized.

This repository contains the normative specification, version history, and working group materials for CAES.

---

## Why CAES Exists

AI systems have moved beyond advisory roles. They now:

- Commit transactions
- Modify infrastructure
- Trigger automated workflows
- Interact with physical systems
- Influence consequential human decisions

Yet there is no widely adopted standard defining what it means to **authorize** an AI action and **prove** that authorization occurred.

CAES addresses this structural accountability gap.

---

## Scope

CAES applies to any AI system that crosses an **Effect Boundary**, including:

- Digital state mutation
- External system invocation
- Communication with consequential impact
- Behavioral influence in regulated contexts
- Physical actuation (robots, vehicles, automation)
- Cross-boundary access to regulated data
- Behavioral output that influences a decision with material consequence
- Recording, observation, or retention of personal or sensitive data

If an AI system can produce consequential effects without independent human mediation, CAES scope applies.

Purely advisory systems are out of scope.

---

## Core Principles

A conformant CAES implementation MUST provide:

- Deterministic policy evaluation
- Pre-execution authorization receipts
- Cryptographically verifiable evidence
- Append-only causal event recording
- Fail-closed enforcement
- Offline-verifiable artifacts
- Explicit authority delegation for high-stakes actions

Which of these are required at each level is the applicability matrix in Interoperability Contracts. Cryptographic evidence, the Governed Spine, and offline artifacts are Level 2 and Level 3 requirements. Level 1 requires a pre-effect Decision Record and fail-closed enforcement.

**Governance applies to any output capable of producing consequential effect, including behavioral outputs.**

CAES defines structural requirements — not implementation architecture.

---

## Effect Boundary Definition

An **Effect Boundary** is crossed whenever an AI system produces an output that can cause a consequential change in state outside the system itself.

Effect Boundaries MUST be treated as governed action triggers. The following categories are normatively recognized:

| Category | Description |
| :--- | :--- |
| **ExternalSideEffect** | Network calls, filesystem writes, API invocations, database mutations |
| **HumanFacingOutput** | Messages, instructions, recommendations, or communications with material consequence |
| **GovernanceRelevantState** | Policy changes, permission changes, authorization scope changes |
| **SafetyCriticalActuation** | Physical actuation beyond defined safety parameters |
| **WorkflowTransition** | Workflow gate passage, run state transition, process-level commitment |
| **BehavioralInfluence** | AI-generated output that influences a user decision with legal, financial, or regulatory consequence |
| **SensoryCapture** | Recording, observation, or retention of audio, video, personal, or sensitive data |

Implementations MAY define additional Effect Boundary categories. Implementations MUST NOT narrow or exclude the categories above.

Categories are not mutually exclusive. One action can cross several categories at once. Every crossed category remains in force. See Effect Composition. (CAES-EFF-01)

Internal computation, model inference, and non-effecting reasoning are out of scope unless they cross one of the above categories.

---

## Fail-Closed Semantics

**Fail-closed** is the mandatory default behavior under uncertainty, failure, or missing information. It is not configurable.

A system is fail-closed if and only if all of the following hold:

1. **Absence of authorization is denial.** An action MUST NOT proceed if the Decision Record required at the claimed level cannot be produced and checked. At Level 1 that check is structural. At Level 2 and Level 3 it includes signature and PolicyHash verification. There is no implicit authorization path.
2. **Verification failure is denial.** If receipt verification fails — for any reason including infrastructure failure, timeout, or mismatch — execution MUST halt.
3. **Policy evaluation failure is denial.** If policy cannot be evaluated to a deterministic result, the disposition MUST be `Denied` or `RequiresHumanAuthorization`. No action proceeds.
4. **No silent fallback.** A system MUST NOT fall back to a permissive state under failure. Degraded operation that permits execution without receipts is non-compliant.
5. **Failure is recorded.** Failures that trigger fail-closed behavior MUST produce a recorded denial event. Silent failure is non-compliant.

A fail-closed event MUST:
- produce a denial record with the persistence requirements of that level
- at Level 2 and Level 3, append that record to the Governed Spine
- surface a structured error indicator to the caller

> **Uncertainty is denial. Silence is non-compliant.**

---

## Authorization-Execution Separation

CAES requires strict temporal separation between the authorization decision and the execution of the governed action.

Requirements:

1. **Decision precedes execution.** A Decision Record MUST be produced, persisted, and checked before any effect-bearing action is initiated. At Level 2 and Level 3 that record is a Decision Receipt.
2. **Record is the gate.** The execution layer MUST check for the required Decision Record as its first action. No record means no execution.
3. **No reconstruction after the fact.** A receipt produced after execution has occurred does not satisfy this requirement. Post-hoc receipts are non-compliant.
4. **Write-then-verify persistence.** Receipt persistence MUST include an immediate readback and byte-level verification step. Acknowledgment from a storage system is not sufficient. Verification MUST confirm durable storage before execution is authorized.
5. **Receipt bound to action scope.** The receipt MUST uniquely identify the specific action authorized. A receipt for Action A cannot authorize Action B, even if the same policy was evaluated.

Non-compliant patterns:
- Recording a receipt at execution time rather than before execution
- Using a previously issued receipt to authorize a new action
- Treating a storage acknowledgment as equivalent to verified persistence
- Allowing execution to proceed when receipt verification is unavailable

---

## Decision Dispositions

Every governance evaluation MUST yield exactly one of the following dispositions. Dispositions MUST be explicitly named — implicit or inferred dispositions are non-compliant.

| Disposition | Meaning | Execution Outcome |
| :--- | :--- | :--- |
| **Approved** | The policy accepted the action as proposed | The effect may proceed only while execution authority is live |
| **Modified** | The policy accepted a transformed action | The effect may proceed only on the transformed payload, and only while execution authority is live |
| **Denied** | The policy rejected the action | No effect. A denial record is produced |
| **RequiresHumanAuthorization** | The policy requires a human decision before any effect | No effect until a later execution authority records that decision |

Requirements:

- A `Modified` disposition MUST specify the transformation applied and bind it to the receipt
- A `Denied` disposition MUST produce a denial record with the persistence requirements of that level. Signing applies at Level 2 and Level 3
- A `RequiresHumanAuthorization` disposition MUST define an explicit escalation pathway; timeout or failure of that pathway MUST resolve to `Denied`
- Implementations MUST NOT define additional dispositions that bypass execution gating

---

## Core Primitives

CAES formalizes three foundational objects. All three MUST be present for Level 2 conformance. Level 1 requires a Decision Record. A Level 2 Decision Receipt is a signed Decision Record and satisfies the Level 1 record requirements.

### 1. Decision Record and Decision Receipt

A **Decision Record** is the pre-effect governance artifact. A **Decision Receipt** is a Decision Record with the Level 2 signature and PolicyHash profile.

**Level 1 Decision Record (CAES-PRE-01, CAES-PRE-02, CAES-PRE-03, CAES-DSP-01):**

- **Pre-effect.** MUST be produced, persisted, and structurally checked before any governed effect. A record produced after the effect is non-compliant.
- **Write-then-verify.** Persistence MUST include an immediate readback and byte-level comparison. A storage acknowledgment is not enough.
- **Structural check.** At Level 1, "unverifiable" means a missing required field or a failed readback. It does not mean signature verification. Signature failure is a Level 2 check.
- **Disposition.** MUST name exactly one policy disposition: `Approved`, `Modified`, `Denied`, or `RequiresHumanAuthorization`.
- **Action binding.** MUST identify the action, including tenant, intended executor, and enough of the payload to distinguish it from a different action. This binding does not by itself make the record single-use. Single-use is the replay lifecycle below.
- **Time.** `iat` MUST be an RFC 3339 timestamp asserted by the producer. At Level 1 this timestamp is not a trusted-time proof.

Signature, PolicyHash, spine append, and offline verification are not Level 1 requirements.

**Level 2 Decision Receipt** adds CAES-SIG-01 and CAES-HASH-01. The receipt MUST be a JWS under the signing profile in this draft, MUST embed the PolicyHash, and MUST be appended to the Governed Spine before the effect. If the signature or PolicyHash is absent or does not verify, execution authority MUST NOT be granted.

**Denial records.** A `Denied` disposition uses the same persistence rules as an approval at that level. At Level 2 the denial is signed. A denial is evidence that the constraint functioned. It is not an error.

---

### 2. PolicyHash

A PolicyHash is the SHA-256 fingerprint of the effective policy bundle that governed the decision. It is required at Level 2 and Level 3 (CAES-HASH-01, CAES-HASH-02). It is not a Level 1 blocker.

The hashed bytes are the UTF-8 RFC 8785 (JCS) encoding of the **effective policy bundle**, not a version label and not a pointer. The bundle MUST contain the declared policy after imports are resolved and defaults are expanded. Runtime context is not part of the bundle. Semantically meaningful array order MUST be preserved. JCS rejects duplicate property names, constrains number representation, and does not Unicode-normalize strings.

```text
PolicyHash = lowercase-hex(SHA-256(JCS(effective_policy_bundle)))
```

The hash MUST be computed at evaluation time, embedded in the receipt, and left unchanged after signing. A verifier with the bundle MUST be able to recompute it offline. If the policy changes before execution, the receipt is not valid execution authority.

---

### 3. Governed Spine

The Governed Spine is an append-only, causally ordered, tenant-scoped record of governed events. Spine requirements apply at Level 2 and Level 3. Level 1 does not require a spine append (CAES-SPN-01).

**Required properties:**

- **Append-only:** Entries MUST NOT be modified or deleted after creation. Revocation, correction, and tombstoning are explicit append events — not mutations.
- **Causal ordering:** Events MUST be ordered within their partition scope. Ordering MUST be assigned by the platform at ingestion — not by the emitting actor.
- **Tenant scoping:** Every event MUST carry a tenant identifier. Cross-tenant access MUST be structurally impossible, not merely policy-restricted.
- **Fail-closed on append failure:** If a spine append fails, the associated action MUST NOT proceed. No partial state. No silent continuation.
- **Canonical identifiers:** All spine objects MUST carry platform-assigned identifiers. Actor-assigned or self-generated identifiers as primary spine keys are non-compliant.
- **Causal linkage:** Each event MUST carry a reference to its causal parent where one exists. Events that begin a new causal chain MAY carry a null parent reference, but MUST NOT fabricate a causal link.
- **Dead Letter handling:** Events that cannot be successfully appended after bounded retry MUST be routed to a Dead Letter mechanism. DLQ entries are evidence of system boundary encounters and MUST be accessible for audit.

---

## Compliance Levels

CAES defines three conformance tiers. Each tier is a strict superset of the previous.

Conformance claims MUST specify:
- The CAES version
- The claimed level
- A complete Conformance Statement (see Conformance Claims section)

Partial conformance claims are non-compliant.

---

### Level 1 — Receipt-Bounded Execution

**Minimum requirement:** All effect-boundary actions require a structurally checked Decision Record before the effect. Fail-closed enforcement. (CAES-PRE-01)

**Testable requirements:**

| Requirement | Test |
| :--- | :--- |
| Decision Record produced before execution | Verify the record timestamp precedes the effect |
| Receipt persisted with write-then-verify | Demonstrate readback verification step in implementation |
| Execution blocked if receipt absent | Inject missing receipt; verify execution does not proceed |
| Execution blocked if the Level 1 record is structurally invalid | Remove a required field or fail the readback; verify execution does not proceed |
| Denial produces Denial Receipt | Trigger denial; verify denial record exists in ledger |
| Fail-closed on verification failure | Simulate storage failure; verify execution halts |
| No silent degradation path | Audit all execution paths; confirm no receipt-free path exists |

**Not required at Level 1:** PolicyHash, RFC 8785 canonicalization, JWS signature, spine ordering, and offline Evidence Pack verification. Implementations SHOULD include them. Their absence is not a Level 1 failure. Their presence does not remove the Level 1 pre-effect rule. The applicability matrix in Interoperability Contracts is controlling where this draft used to state both "every receipt is signed" and "Level 1 need not sign."

---

### Level 2 — Verifiable Evidence Chain

**Minimum requirement:** All Level 1 requirements plus cryptographic signing, PolicyHash canonicalization, append-only spine, and offline-verifiable sealed artifacts.

**Testable requirements (in addition to Level 1):**

| Requirement | Test |
| :--- | :--- |
| Receipts signed with named algorithm | Inspect receipt; verify signature and algorithm declaration |
| PolicyHash present in every receipt | Inspect receipt; verify PolicyHash field present and populated |
| PolicyHash deterministic | Evaluate same policy twice; verify byte-identical hash |
| PolicyHash recomputable offline | Provide policy definition to verifier; verify independent hash matches |
| Spine is append-only | Attempt modification of existing entry; verify rejection |
| Spine ordering is platform-assigned | Inspect sequence field; verify actor cannot control it |
| Evidence Pack is sealed and offline-verifiable | Export pack; verify on air-gapped machine with zero network calls |
| Pack tamper-evident | Modify one byte in sealed pack; verify signature invalidation |
| Decision dispositions explicitly named | Inspect all governance outcomes; verify each maps to a named disposition |

---

### Level 3 — Full Constitutional Conformance

**Minimum requirement:** All Level 1 and Level 2 requirements plus human authority delegation, complete causal invariants, effect classification, chaos attestation, and structured error codes.

**Testable requirements (in addition to Level 1 and Level 2):**

| Requirement | Test |
| :--- | :--- |
| Human authority delegation produces binding artifact | Trigger human authorization; verify DelegationChain artifact in pack |
| Delegation chain depth enforced | Exceed configured max depth; verify hard rejection with structured error |
| Delegation chain expiry enforced | Use expired delegation; verify rejection with structured error |
| Delegation scope enforced | Use delegation for out-of-scope action; verify rejection |
| Delegation binding target mandatory | Omit target binding; verify rejection with `BINDING_MISSING` error |
| Fail-closed under 8 chaos modes | Execute the chaos harness. Each mode MUST meet its own pass condition in Chaos fixtures. A shared error code is not a pass |
| Chaos modes covered: DB write failure, key rotation conflict, clock skew, network partition during pack export, disk full during pack write, duplicate receipt injection, PolicyHash mismatch, delegation chain expiry | Each mode independently tested |
| Evidence Pack contains PolicyHash artifact | Inspect exported pack; verify PolicyHash manifest present |
| Evidence Pack contains DelegationChain artifact | Inspect exported pack for delegated actions; verify artifact present |
| Evidence Pack contains ChaosTestAttestation | Inspect exported pack; verify attestation covering all 8 chaos modes |
| Cross-pack provenance chain | Link packs across operations; verify broken link produces structured error |
| All public-surface failures produce structured error codes | Enumerate failure paths; verify 100% emit typed error codes, no raw exceptions |
| Evidence Pack export is deterministic | Export same scope twice; verify byte-identical output |
| Offline verification requires zero network calls | Air-gap verify; confirm zero outbound packets |
| Effect Boundary categories declared | Inspect Conformance Statement; verify all governed categories listed |

**Level 3 structured error code requirement:**

Implementations MUST define a typed error code namespace. Every failure mode reachable from the public surface MUST map to a named code in that namespace. Raw exceptions or untyped error strings surfacing to callers are non-compliant.

---

## Non-Compliant Patterns (Explicitly Disallowed)

CAES prohibits the following patterns. An implementation exhibiting any of these patterns MUST NOT claim CAES conformance at any level.

**Authorization Failures**
- Prompt-only "policy" enforcement without deterministic evaluation
- Post-hoc logging presented as authorization proof
- Non-deterministic model-only policy evaluation without deterministic verification layer
- Receipts produced after execution rather than before
- Reuse of a prior receipt to authorize a new action
- Treating storage acknowledgment as equivalent to verified persistence

**Evidence Integrity Failures**
- Mutable logs masquerading as evidence ledgers
- PolicyHash implemented as a version label, pointer, or name rather than a cryptographic hash of canonical policy inputs
- PolicyHash recomputed at execution time rather than bound at evaluation time
- Evidence artifacts that require live system access for verification

**Enforcement Failures**
- Silent degradation under failure conditions
- Fallback to permissive execution when receipt verification is unavailable
- Implicit authorization paths not gated by receipt verification
- Invocation-layer governance bypass (governance that applies only to some invocation surfaces)
- Governance that applies only when explicitly enabled

**Behavioral and Output Failures**
- Post-hoc content filtering presented as governance of behavioral outputs
- Prompt-only behavioral constraints without deterministic evaluation
- Silent modification of outputs without audit trace
- Emission of behavior when policy evaluation fails or is unavailable

**Recording and Privacy Failures**
- Recording without verifiable authorization
- Passive or continuous recording without policy evaluation
- Use of recorded data outside authorized purpose
- Silent collection of personal or sensitive data
- Deletion without audit trace or verification

---

## Behavioral Governance

### Overview

CAES applies not only to traditional execution actions (e.g., transactions, system mutations), but also to **effect-bearing behaviors** produced by AI systems.

An AI system may produce consequential outcomes through:
- language (responses, instructions, recommendations)
- interaction (user guidance, persuasion, escalation)
- physical actuation (robotics, embodied systems)
- social or psychological influence

These outputs may cross an Effect Boundary even when no explicit system mutation occurs.

CAES therefore requires governance of **behavioral outputs**, not just execution calls.

---

### Behavioral Effect Boundary

A **Behavioral Effect Boundary** exists when an AI-generated output can:
- influence a user decision with material consequence
- produce legal, financial, regulatory, or safety impact
- trigger downstream actions by humans or systems
- create risk through communication, instruction, or interaction

Systems MUST treat such outputs as effect-bound.

---

### Policy Requirements for Behavioral Governance

Policies MUST define constraints on:

- **Allowed content** — what the system may say or communicate
- **Disallowed content** — prohibited instructions, claims, or statements
- **Interaction boundaries** — escalation requirements, refusal conditions, handoff to human authority
- **Domain-specific restrictions** — regulated domains (legal, medical, financial), sensitive interactions (minors, vulnerable populations)
- **Physical interaction constraints** (if applicable) — permitted contact or actuation, safety envelopes and exclusion zones

Policies MAY include contextual or conditional rules, but MUST remain deterministically evaluable at the point of decision.

---

### Behavioral Decision Requirement

For any behavior crossing a Behavioral Effect Boundary, a Decision Record, or a Decision Receipt at Level 2, MUST exist before emission as defined in Streaming and emission. A completion record after the stream ends is outcome evidence, not prior authorization.

The record MUST:
- at Level 2 and Level 3, bind the behavior to the governing policy state (PolicyHash)
- capture the evaluated decision disposition
- include sufficient context to support audit and reconstruction

---

### Behavioral Enforcement

Systems MUST enforce behavioral policies:
- **before output is delivered**
- **without relying on post-hoc filtering alone**

Post-generation filtering MAY be used as a secondary control, but MUST NOT replace pre-decision authorization.

If policy evaluation fails or cannot be completed:
- the system MUST fail closed
- the behavior MUST NOT be emitted

---

### Behavioral Dispositions

Behavioral governance decisions MUST result in one of the four defined Decision Dispositions:

- **Approved** — output may be emitted as generated only while execution authority is live
- **Modified** — the transformed output is the only payload that may be emitted, and only while execution authority is live
- **Denied** — output must not be emitted
- **RequiresHumanAuthorization** — escalation required before emission

All dispositions MUST be recorded and auditable.

---

### Causal Recording

At Level 2 and Level 3, behavioral decisions MUST be recorded in the Governed Spine, including:
- the evaluated behavior or output reference
- the Decision Record reference
- the resulting disposition
- any transformation or escalation applied

Behavioral events are first-class governed events.

---

## Privacy, Recording, and Data Stewardship

### Overview

AI systems may possess capabilities to:
- record audio, video, or environmental data
- capture user-generated content
- observe or infer personal or sensitive information

These capabilities introduce legal, privacy, and ownership obligations equivalent to or exceeding those applied to human actors.

CAES requires that such capabilities be governed as **effect-bearing operations**.

---

### Sensory Effect Boundary

A **Sensory Effect Boundary** exists when an AI system can:
- record audio, video, or other environmental signals
- capture or store user content
- observe or infer personally identifiable or sensitive information

Crossing this boundary MUST be treated as a governed action. The record, signature, PolicyHash, and spine duties are those of the claimed level.

---

### Consent and Authorization Requirements

AI systems MUST NOT:
- record audio, video, or environmental data without appropriate authorization
- capture or store personal content without a permitted legal basis
- operate recording capabilities in restricted or prohibited contexts

Authorization MAY be derived from:
- explicit user consent
- contractual agreement
- legal or regulatory allowance

The system MUST be able to:
- determine whether recording is permitted in context
- deny or disable recording if authorization cannot be verified

Failure to verify authorization MUST result in fail-closed behavior.

---

### Use and Purpose Limitation

Captured data MUST:
- be used only for the purposes for which authorization was granted
- NOT be repurposed without reauthorization
- NOT be accessed outside defined policy scope

Policies MUST define:
- allowed use cases
- prohibited use cases
- domain-specific restrictions

---

### Retention and Storage Governance

AI systems MUST enforce:
- defined retention policies for all captured data
- automatic expiration or deletion based on policy rules
- restrictions on duplication, transfer, or export

Retention policies MUST be:
- explicitly defined
- deterministically enforceable
- auditable

---

### Deletion and Proof of Deletion

Data deletion MUST be treated as a **governed action**.

When data is deleted:
- the deletion MUST be executed through the governed execution pathway
- the deletion MUST produce a Decision Record representing the action
- at Level 2 and Level 3, that record MUST be a signed receipt appended to the Governed Spine
- the record MUST be retained even after the data itself is deleted

The deletion record is permanent. The data is not.

---

### Recording and Data Handling Dispositions

Decisions related to recording and data handling MUST result in one of the four defined Decision Dispositions:

- **Approved** — recording or usage permitted
- **Denied** — recording or usage prohibited
- **Modified** — restricted or redacted capture authorized
- **RequiresHumanAuthorization** — escalation required before capture or use

All decisions MUST be auditable and traceable.

---

### Enforcement Requirements

Privacy and recording policies MUST be enforced:
- prior to data capture
- prior to data usage
- prior to data retention or transfer

Post-hoc enforcement alone is non-compliant.

---

### Principle

Capabilities that observe, record, or retain data carry the same governance obligations as actions that execute. (CAES-PRV-01)

Conformance is judged on observable behavior: an attributable decision, an identified responsible operator, the privacy controls in this section, authorization before capture or use, and an auditable effect. (CAES-PRV-02)

> **Rationale, not a conformance requirement.** The Working Group's aspiration is that these controls make an AI system accountable in the same practical sense as a human operator who records or retains data. That sentence does not confer legal status, and a conformance claim MUST NOT rest on it.

---

## Interoperability Contracts

These contracts complete the v0.2.0 public draft. Requirement IDs are stable. Editorial rearrangement MUST NOT recycle an ID for a different obligation.

Three states are distinct and MUST NOT be collapsed into one field:

| State | Meaning | ID |
| :--- | :--- | :--- |
| Policy disposition | What the policy decided | CAES-STA-01 |
| Execution authority | Whether prerequisites, scope, time bounds, and consumption state currently permit the effect | CAES-STA-02 |
| Outcome evidence | What happened, including partial delivery, a blocked effect, and uncertain completion | CAES-STA-03 |

An `Approved` disposition does not by itself grant execution authority. Outcome evidence does not rewrite the disposition.

### Applicability matrix

| Obligation | ID | Level 1 | Level 2 | Level 3 |
| :--- | :--- | :--- | :--- | :--- |
| Decision Record before any governed effect | CAES-PRE-01 | Required | Required | Required |
| Write-then-verify persistence | CAES-PRE-02 | Required | Required | Required |
| Fail closed when the record is missing or structurally invalid | CAES-PRE-03 | Required | Required | Required |
| Exactly one policy disposition | CAES-DSP-01 | Required | Required | Required |
| JWS signature under this profile | CAES-SIG-01 | Not required | Required | Required |
| PolicyHash of the effective bundle | CAES-HASH-01 | Not required | Required | Required |
| RFC 8785 plus SHA-256, lowercase hex | CAES-HASH-02 | Not required | Required | Required |
| Governed Spine append | CAES-SPN-01 | Not required | Required | Required |
| Evidence Pack annex | CAES-PAK-01 | Not required | Required | Required |
| Delegation, chaos fixtures, typed errors | CAES-L3-01 | Not required | Not required | Required |
| CPP v1.0 | CAES-CPP-01 | Not required | Required | Required |

### Signing and verification profile

Level 2 and Level 3 receipts MUST use flattened JWS JSON (RFC 7515) with one signature. (CAES-SIG-02)

The protected header MUST contain `alg`, `kid`, and `typ` with value `caes-receipt+jws`. The payload is the JCS encoding of the receipt JSON. The JWS signing input is `BASE64URL(protected) || '.' || BASE64URL(JCS(payload))`.

Interchange algorithms are `EdDSA` (Ed25519) and `ES256` only. `none` and shared-secret algorithms are non-compliant. (CAES-SIG-03)

`kid` MUST resolve through the Evidence Pack trust bundle or the trust anchors named in the Conformance Statement. An unknown or revoked `kid` MUST NOT grant execution authority. During a published rotation overlap, a signature under either key that is inside that window MAY be accepted. Overlap alone is not a reason to deny. (CAES-SIG-04)

Receipt claims MUST include `caes_version`, `receipt_id`, `tenant_id`, `issuer`, `audience`, `action_id`, `action_descriptor`, `payload_hash`, `disposition`, `enforcement_outcome`, `iat`, and `exp`. Level 2 claims also include `policy_id`, `policy_version`, `policy_hash`, and `matched_rules`. `payload_hash` is lowercase-hex SHA-256 of the canonical action payload that would be effected. (CAES-SIG-05)

`iat` and `exp` MUST be RFC 3339 timestamps with a `Z` or numeric offset. They are signer-asserted. A signature does not prove that the clock was accurate. For an interchange claim, `exp - iat` MUST be at most 15 minutes, and declared clock skew MUST be at most 120 seconds. If clock accuracy is unknown, execution authority MUST NOT be granted. Archival verification MAY still report that the signature matches, and it MUST label the time as untrusted. An RFC 3161 time-stamp token is optional additional evidence. It does not replace `iat`, and it is required only when the Conformance Statement claims trusted-time assurance. (CAES-SIG-06)

`exp` bounds execution authority. It does not erase an archived receipt. Later verification asks whether the signature and claims were valid when recorded. It does not ask whether the authority is still unused. (CAES-SIG-07)

### Effect composition

CPP `effect_type` values and these categories are not a one-to-one map. A policy names one `effect_type`. A concrete action lists every CAES category it crosses. Evaluation MUST select every installed policy whose `effect_type` applies to any listed category, then apply CPP conflict resolution to the combined matches. A category with no applicable policy is Denied. An `effect_type` outside the CPP set fails closed. (CAES-EFF-02, CPP-EFF-01)

| CPP `effect_type` | CAES categories it can require |
| :--- | :--- |
| Execution | ExternalSideEffect, WorkflowTransition, GovernanceRelevantState |
| ExternalInvocation | ExternalSideEffect, GovernanceRelevantState, WorkflowTransition |
| Communication | HumanFacingOutput |
| BehavioralInfluence | BehavioralInfluence, HumanFacingOutput |
| DataMutation | ExternalSideEffect, GovernanceRelevantState |
| SensoryCapture | SensoryCapture |
| Deletion | ExternalSideEffect |

`SafetyCriticalActuation` has no CPP `effect_type` in this version. An action that lists it therefore has no applicable policy for that category and is Denied. A later version must name a mapping before such an action can be Approved.

`ExternalInvocation` is the worked case: one call can be an external side effect, a governance-state change, and a workflow transition together. All three controls apply. Dropping one because the policy named only one type is non-compliant.

### Tenant selectors and dispositions

Tenant comparison is a Unicode code-point match with no case folding and no normalization. The only wildcard is the exact string `*`. Every other value is literal. Tenant identifiers MUST NOT be `*`. A policy with `tenant_id` `*` MUST NOT be installed unless the installing authority presents a delegation whose scope is `cross_tenant_policy`. Otherwise that policy is ineligible. (CPP-TEN-01)

| Rule action | Policy disposition | Execution consequence | ID |
| :--- | :--- | :--- | :--- |
| allow | Approved | The effect may proceed only while execution authority is live | CPP-ACT-01 |
| deny | Denied | No effect | CPP-ACT-02 |
| modify | Modified | The transformed payload MUST be named in the record before any byte of the effect. Executing the pre-transform payload is non-compliant | CPP-ACT-03 |
| escalate | RequiresHumanAuthorization | The record MUST name the required authority and the resumption conditions. The disposition itself does not authorize the effect | CPP-ACT-04 |

### Obligations

An obligation MUST declare `phase`: `pre_effect`, `continuous`, or `post_effect`. Obligations remain auditable and deterministic. They MUST NOT change the policy disposition. (CAES-OBL-01)

- A failed pre-effect obligation MUST NOT grant execution authority. Outcome evidence records the block. The disposition stays what the policy decided, including `Approved`.
- A failed continuous obligation MUST stop further covered effects. Bytes already released stay in the outcome evidence.
- A failed post-effect obligation MUST be recorded with a remediation duty. It MUST NOT be described as if the completed effect did not occur.

### Streaming and emission

Emission is the first byte of governed content that crosses the enforcement boundary. Caller-visible metadata that carries governed content is content. Pure framing, heartbeats, and flow control that cannot carry governed content are not emission. (CAES-STR-01)

The Conformance Statement MUST name one mode. (CAES-STR-02)

- **Buffered.** No governed byte is released until one receipt covers the complete output.
- **Segmented.** Each segment is defined before release, hashed, and covered by a receipt before its first byte is released.

A receipt cannot bind the hash of an output that does not exist yet. A completion record is outcome evidence of what left the system. It is not prior authorization of unknown bytes. (CAES-STR-03)

### Replay lifecycle

Fields do not make a receipt single-use. Before the effect, the executor MUST atomically and durably reserve execution authority for the tuple `(issuer, tenant_id, audience, action_id, payload_hash, receipt_id)`. (CAES-REP-01)

| State | Meaning |
| :--- | :--- |
| Reserved | Authority is held. The effect has not started. |
| Consumed | Completion was recorded after the effect. |
| Failed | The attempt ended with no effect. |
| Abandoned | A reservation was released after a crash before any effect. |
| Uncertain | The effect may have happened, and completion was not recorded. |
| Expired | `exp` passed before a reservation was consumed. |

Retry with the same receipt is allowed only from `Reserved` or `Abandoned`, inside `exp`, and only when no effect has started. If the executor crashes after the effect and before completion, the state MUST become `Uncertain`. A fresh receipt MUST NOT repeat that effect until reconciliation shows it did not occur. While the state is `Uncertain`, further execution of that action is blocked. (CAES-REP-02)

Offline verification checks the signature, the hash, and the recorded claims. It MUST NOT report that the authority is unused. Consumption state is an online enforcement fact. (CAES-REP-03)

### Chaos fixtures

Each mode has its own pass condition. A shared error code is not sufficient. When two modes fire together, every applicable prevention still applies and every applicable evidence record is required. Integrity failures outrank availability failures: do not allow an unauthorized effect in order to finish a pack. (CAES-CH-00)

| Mode | Required behavior | Pass condition | Code |
| :--- | :--- | :--- | :--- |
| DB write failure | No effect. Do not treat a failed evidence write as permission to continue. | The effect is absent. | `CAES-E-DB-WRITE` |
| Key rotation conflict | Accept a signature under either key inside the published overlap. Reject an unknown or revoked `kid`. | A legitimate overlap is not denied. A revoked key grants no authority. | `CAES-E-KEY-ROTATION` |
| Clock skew | If skew exceeds the declared bound, or the clock is untrusted, grant no execution authority. | No effect outside the bound. Archival signature status may still be recorded as time-untrusted. | `CAES-E-CLOCK-SKEW` |
| Network partition during pack export | Do not publish a pack that looks complete and sealed. | The partial export fails verification. | `CAES-E-PACK-PARTITION` |
| Disk full during pack write | Same as a partition: no apparently complete sealed pack, and no corrupted success marker. | Verification fails closed on the incomplete pack. | `CAES-E-PACK-DISK` |
| Duplicate receipt injection | The second reservation does not execute. The original receipt and its evidence stay intact. | One effect at most. Original evidence still verifies. | `CAES-E-RECEIPT-REPLAY` |
| PolicyHash mismatch | No execution authority. | No effect. The mismatch is in the outcome evidence. | `CAES-E-POLICYHASH-MISMATCH` |
| Delegation chain expiry | No execution authority. | No effect. `CAES-E-DELEGATION-EXPIRY` is recorded. | `CAES-E-DELEGATION-EXPIRY` |

`BINDING_MISSING` remains the code when a delegation omits its target binding. (CAES-L3-02)

### Evidence Pack

Level 2 and Level 3 MUST produce an Evidence Pack that conforms to `governance/CAES_EVIDENCE_PACK_SPEC.md` at pack spec version `0.2.0`. That document is a normative annex of this draft. (CAES-PAK-01)

A pack that omits required evidence fails verification. Verification MUST be possible with no call to a private issuer service. Offline success MUST NOT be reported as "this authorization is still unused."

---

## Conformance Claims

Implementations claiming CAES compliance MUST publish a complete **Conformance Statement** that includes all of the following:

| Field | Requirement |
| :--- | :--- |
| CAES version | Exact version string (e.g., `v0.2.0`) |
| Claimed compliance level | Level 1, 2, or 3 |
| Receipt signing algorithm | `EdDSA` or `ES256` for Level 2 interchange |
| PolicyHash profile | Effective policy bundle, RFC 8785, SHA-256, lowercase hex |
| Maximum execution lifetime | `exp - iat`, at most 15 minutes for an interchange claim |
| Maximum clock skew | At most 120 seconds for an interchange claim |
| Stream mode | `buffered` or `segmented` |
| Evidence Pack version | `0.2.0` when Level 2 or Level 3 is claimed |
| Fail-closed enforcement description | How the system behaves under each failure mode |
| Effect Boundary categories governed | Complete list of governed categories |
| Write-then-verify mechanism | Description of receipt persistence and verification procedure |
| Evidence Pack format (Level 2+) | Format and verification method |
| Human delegation model (Level 3) | Delegation artifact structure and enforcement |
| Chaos attestation coverage (Level 3) | Which chaos modes are attested |
| Error code namespace (Level 3) | Reference to typed error code registry |

Unversioned or incomplete Conformance Statements are non-conformant.

Conformance Statements MAY be self-attested. Self-attested Statements MUST be clearly labeled as self-attested. Independent third-party audit is RECOMMENDED for Level 2 claims and STRONGLY RECOMMENDED for Level 3 claims. Self-attestation does not constitute certification by the CAES Working Group.

---

## Reference Implementation

The CAES Working Group designates one reference implementation per version.

**CAES v0.2.0 Reference Implementation:**
Keon Systems — Governed Execution
https://keon.systems

Reference designation does not imply commercial endorsement.
Other implementations may independently conform.

---

## Working Group

CAES is maintained by the CAES Working Group (CAES-WG).

The Working Group publishes:
- Normative specification versions
- Amendment proposals
- Conformance guidelines
- Mark usage policy

Specification changes are versioned.
There are no informal amendments.

---

## Amendment Process

To propose a change:

1. Identify the normative requirement affected.
2. Describe the structural issue.
3. Document backward compatibility impact.
4. Submit as a versioned proposal.

Changes are incorporated only via published version increment.

---

## Mark Usage

"CAES" and associated marks are used to designate conformance with a specific published version.

Use of the CAES mark requires a complete Conformance Statement.

Mark usage guidelines are provided in `/assets`.

---

## License

The CAES specification text is published under a permissive documentation license to encourage adoption and independent implementation.

See `LICENSE.md` for details.

---

## Status

CAES v0.2.0 is a public draft released for review and comment.

This version supersedes v0.1.1 and introduces normative strengthening of Core Primitives, Compliance Levels, Fail-Closed Semantics, Authorization-Execution Separation, Decision Dispositions, Effect Boundary Definition, Behavioral Governance, and Privacy and Recording governance requirements.

This public draft also includes the interoperability completion: separate Level 1 Decision Records and Level 2 receipts, the applicability matrix, the JWS profile, effect composition, the enforcement state machine, streaming, the replay lifecycle, obligation phases, chaos fixtures, and the v0.2.0 Evidence Pack annex. Where an earlier paragraph of this draft conflicts with Interoperability Contracts, the contracts control.

Implementations conforming to v0.1.1 SHOULD review this version for new testable requirements before claiming v0.2.0 conformance.

Feedback and formal amendment proposals are welcome.

---

CAES
Constitutional AI Execution Standard
March 2026
