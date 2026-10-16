# CPP

## Constitutional Policy Protocol

**v1.0 Normative Specification**

| Field | Value |
|---|---|
| Status | Public Draft |
| Required For | CAES Level 2+ Conformance |
| Governing Body | CAES Working Group |
| Canonical Source | `caes-standards-org/CPP-v1.0.md` (this file) |

> **Canonical Statement**
>
> A policy is not a configuration.
> It is a deterministic contract whose outcome can be proven, reproduced, and independently verified.

*Constitutional AI Execution Standard | Public Draft*

---

## Document Map

This public draft defines the canonical policy protocol required for CAES Level 2+ conformance. This document is the canonical CPP specification; the following map reflects its normative structure.

1. Purpose
2. Scope
3. Core Requirements
4. Canonical Policy Model
5. Deterministic Evaluation (Non-Negotiable)
6. PolicyHash (Canonical Binding) — Why This Matters
7. Evaluation Output (Normative) — Requirements
8. Decision Binding (Critical)
9. Conflict Resolution (Deterministic)
10. Deny-by-Default Enforcement
11. Obligations Model — Requirements
12. Change Control — Prohibited
13. Behavioral Policy Enforcement — Requirement
14. Lifecycle Governance (Data + Deletion)
15. Non-Compliant Patterns
16. Conformance Requirements
17. CAES Integration
18. Canonical Statement
19. Tenant Selectors
20. Effect Composition
21. Action and Disposition Mapping
22. Design Rationale (Non-Normative)

> **Protocol Thesis**
>
> CAES defines how decisions must be enforced. CPP defines how decisions must be determined.

---

## 1. Purpose

CPP defines the **canonical structure, evaluation semantics, and verification model** for policies governing AI systems operating under CAES.

> If policy is not standardized, governance is not provable.

CAES defines *how decisions must be enforced*. CPP defines *how decisions must be determined*.

Together, they form the complete execution constitution.

## 2. Scope

CPP applies to all policy systems used to evaluate:

- execution actions
- behavioral outputs
- data operations (capture, use, retention, deletion)
- communication with material consequence
- any output crossing a CAES Effect Boundary

CPP is **mandatory for CAES Level 2 and Level 3 systems**.

## 3. Core Requirements

A CPP-compliant policy system **MUST** be:

### 3.1 Deterministic
- identical inputs -> identical outputs
- no stochastic or LLM-only evaluation paths

### 3.2 Versioned
- every policy instance MUST include a version identifier
- versions MUST be immutable

### 3.3 Immutable
- policies MUST NOT be modified in-place
- updates MUST produce a new version

### 3.4 Hashable
- policies MUST produce a canonical **PolicyHash**

### 3.5 Auditable
- rule evaluation MUST be traceable
- matched rules MUST be explicitly recorded

### 3.6 Portable
- policies MUST be evaluable outside the originating system

## 4. Canonical Policy Model

A CPP policy **MUST** conform to the following structure:

```json
{
  "policy_id": "string",
  "version": "string",
  "effect_type": "Execution | Communication | BehavioralInfluence | DataMutation | SensoryCapture | Deletion | ExternalInvocation",
  "scope": {
    "tenant_id": "string | wildcard",
    "actor_roles": ["string"],
    "resource_types": ["string"]
  },
  "rules": [
    {
      "rule_id": "string",
      "priority": 0,
      "condition": "deterministic expression",
      "action": "allow | deny | modify | escalate",
      "constraints": {}
    }
  ],
  "obligations": [
    {
      "type": "log | notify | require_human | redact | delay",
      "parameters": {}
    }
  ],
  "evaluation": {
    "mode": "deny_by_default",
    "conflict_resolution": "priority | most_restrictive | first_match"
  }
}
```

## 5. Deterministic Evaluation (Non-Negotiable)

Policy evaluation **MUST NOT** depend on:

- LLM interpretation alone
- probabilistic scoring
- implicit reasoning chains

Allowed:

- LLM-assisted *input classification* (pre-policy)
- LLM-assisted *suggestions*

Not allowed:

- LLM deciding authorization outcome

> If evaluation cannot be reproduced byte-for-byte, it is non-compliant.

## 6. PolicyHash (Canonical Binding)

Each policy instance MUST produce a deterministic hash of the effective policy bundle (CPP-HASH-01):

```text
PolicyHash = lowercase-hex(SHA-256(JCS(effective_policy_bundle)))
```

JCS is RFC 8785. The bundle is the declared policy after imports are resolved and defaults are expanded. Runtime context is excluded. Array order that changes meaning MUST be preserved. Duplicate property names are rejected. Strings are not Unicode-normalized.

### Requirements

- The hash input MUST be those JCS bytes, not a version label or a pointer. (CPP-HASH-02)
- The digest MUST be SHA-256, encoded as lowercase hexadecimal. (CPP-HASH-03)
- PolicyHash MUST be embedded in Level 2 and Level 3 Decision Receipts, immutable after evaluation, and independently recomputable from the bundle. (CPP-HASH-04)
- Level 1 CAES records are not required to carry this hash. See the CAES applicability matrix.

### Why This Matters

PolicyHash is:

> the cryptographic fingerprint of governance itself

Without it:

- audits become interpretive
- policy drift becomes invisible
- decisions become non-reproducible

## 7. Evaluation Output (Normative)

Every policy evaluation MUST produce:

```json
{
  "policy_id": "...",
  "policy_version": "...",
  "policy_hash": "...",
  "matched_rules": [
    {
      "rule_id": "...",
      "action": "...",
      "priority": 0
    }
  ],
  "disposition": "Approved | Denied | Modified | RequiresHumanAuthorization",
  "enforcement_outcome": "authorized | blocked | denied | uncertain | not_applicable"
}
```

### Requirements

- `matched_rules` MUST NOT be empty for Approved or Modified decisions. (CPP-OUT-01)
- `disposition` is the policy decision and MUST be one of the four CAES values. It MUST NOT be overwritten when enforcement later blocks the effect. (CPP-OUT-02)
- `enforcement_outcome` records whether authority was granted, blocked, denied, or uncertain. (CPP-OUT-03)
- At Level 2 and Level 3 the output MUST be bound into the Decision Receipt. A Level 1 Decision Record MUST still carry `disposition` and the action binding. (CPP-OUT-04)

## 8. Decision Binding (Critical)

At Level 2 and Level 3, CPP outputs MUST be embedded into the CAES Decision Receipt.

Each such receipt MUST include:

- `policy_id`
- `policy_version`
- `policy_hash`
- `matched_rules`
- `disposition`

A Level 1 Decision Record MUST carry `disposition` and the action binding. It is not required to carry `policy_hash`. (CPP-OUT-04)

> A decision without policy binding is not proof.

## 9. Conflict Resolution (Deterministic)

Policy systems MUST define a deterministic resolution strategy:

Allowed:

- priority-based
- most-restrictive-wins
- first-match

Not allowed:

- ambiguous rule resolution
- implicit precedence
- runtime-dependent ordering

## 10. Deny-by-Default Enforcement

CPP systems MUST enforce:

```
default_disposition = Denied
```

If:

- no rule matches
- evaluation fails
- context is incomplete

Then: -> **Denied**

> Absence of permission is denial.

## 11. Obligations Model

Policies MAY define obligations that accompany decisions.

Examples:

- logging requirements
- human approval workflows
- redaction transformations
- notification triggers

### Requirements

- Each obligation MUST set `phase` to `pre_effect`, `continuous`, or `post_effect`. (CPP-OBL-01)
- Obligations MUST NOT override disposition. (CPP-OBL-02)
- Obligations MUST be auditable and deterministic. (CPP-OBL-03)
- A failed `pre_effect` obligation MUST leave execution authority ungranted.
- A failed `continuous` obligation MUST stop further covered effects.
- A failed `post_effect` obligation MUST be recorded with a remediation duty and MUST NOT be treated as undoing a completed effect.

## 12. Change Control

Policy lifecycle MUST follow:

**Immutable History**
- no in-place edits

**Version Creation**
- every change = new version

**Traceability**
- all versions MUST remain accessible

### Prohibited

- silent policy updates
- retroactive mutation
- version aliasing

## 13. Behavioral Policy Enforcement

CPP MUST support governance of:

- AI-generated responses
- recommendations
- instructions
- persuasive outputs

### Requirement

Behavioral outputs MUST:

- pass policy evaluation
- produce Decision Receipt
- bind output to policy

> Words can be actions. CPP governs both.

## 14. Lifecycle Governance (Data + Deletion)

CPP MUST support policies for:

**Data Capture**
- consent validation
- scope enforcement

**Data Usage**
- purpose limitation

**Retention**
- time-bound policies

**Deletion**
- pre-authorized only
- receipt-bound

> Deletion without pre-authorization is non-compliant.

## 15. Non-Compliant Patterns

CPP explicitly prohibits:

### Policy Design Failures
- hidden logic in application code
- non-versioned policies
- runtime mutation

### Evaluation Failures
- LLM-only decision making
- non-deterministic evaluation
- missing rule trace

### Governance Failures
- execution without policy binding
- behavioral output without evaluation
- silent policy fallback

## 16. Conformance Requirements

A CPP-compliant system MUST publish:

- canonical policy schema
- canonicalization method, which for interchange is RFC 8785 over the effective policy bundle
- hashing algorithm, which for interchange is SHA-256 lowercase hex
- evaluation engine description
- conflict resolution strategy
- obligation execution model, including the phase of each obligation type
- tenant-selector rules and whether cross-tenant wildcards are installed
- the action-to-disposition mapping in section 21

## 17. CAES Integration

CPP is REQUIRED for:

| CAES Level | CPP Requirement |
|---|---|
| Level 1 | Optional |
| Level 2 | Required |
| Level 3 | Required |

CPP enables:

- PolicyHash binding
- deterministic audit
- offline verification
- cross-system governance portability

## 18. Canonical Statement

> A policy is not a configuration. It is a deterministic contract whose outcome can be proven, reproduced, and independently verified.

## 19. Tenant Selectors

`tenant_id` is either a literal tenant identifier or the exact wildcard `*`. (CPP-TEN-01)

- Comparison is code point by code point. No case folding. No Unicode normalization.
- There is no other pattern syntax and no escape syntax.
- A tenant identifier MUST NOT be `*`.
- A policy that uses `*` MUST NOT be installed unless the installer presents a delegation scoped `cross_tenant_policy`. Without that delegation the policy is ineligible, and a category that then has no policy is Denied.

## 20. Effect Composition

A policy carries one `effect_type`. An action can cross several CAES categories. The categories are not a one-to-one image of `effect_type`. (CPP-EFF-01)

The executor MUST list every CAES category the concrete action crosses and MUST evaluate every eligible policy whose `effect_type` applies to any listed category. Conflict resolution then runs on the combined matches. A crossed category with no eligible policy is Denied. An unknown `effect_type` fails closed.

| `effect_type` | Categories that remain in force |
| :--- | :--- |
| Execution | ExternalSideEffect, WorkflowTransition, GovernanceRelevantState |
| ExternalInvocation | ExternalSideEffect, GovernanceRelevantState, WorkflowTransition |
| Communication | HumanFacingOutput |
| BehavioralInfluence | BehavioralInfluence, HumanFacingOutput |
| DataMutation | ExternalSideEffect, GovernanceRelevantState |
| SensoryCapture | SensoryCapture |
| Deletion | ExternalSideEffect |

`SafetyCriticalActuation` has no CPP `effect_type` in this version. An action that lists it has no applicable policy for that category and is Denied.

`ExternalInvocation` may therefore require ExternalSideEffect, GovernanceRelevantState, and WorkflowTransition controls on the same call. Naming one type does not drop the others.

## 21. Action and Disposition Mapping

| Rule `action` | Disposition | What must be true before an effect | ID |
| :--- | :--- | :--- | :--- |
| allow | Approved | Execution authority is live | CPP-ACT-01 |
| deny | Denied | No effect | CPP-ACT-02 |
| modify | Modified | The transformed payload is identified and is the payload the receipt covers | CPP-ACT-03 |
| escalate | RequiresHumanAuthorization | The required authority and the resumption conditions are recorded. The disposition is not itself authority | CPP-ACT-04 |

Producing `RequiresHumanAuthorization` does not complete human authorization. Resumption requires a new execution authority bound to that decision.

## 22. Design Rationale (Non-Normative)

Deterministic, verifiable policy allows governance claims to be checked
independently rather than accepted on trust. Where policy exists only as
informal or mutable configuration, conformance cannot be reproduced or
audited after the fact.

Under CPP:

- policy is expressed as a deterministic, hashable contract
- decisions are replayable against the exact policy state that produced them
- governance evidence is portable and independently verifiable across systems

This section is informative. It imposes no requirements beyond the normative
sections above and does not modify the Canonical Statement (§18).
