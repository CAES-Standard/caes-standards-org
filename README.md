# CAES
## Constitutional AI Execution Standard

**Version:** v0.2.0 (Draft)
**Status:** Public Review
**Working Group:** CAES-WG
**Category:** Normative Specification
**Supersedes:** v0.1.1
**Companion:** CPP v1.0 (required for Level 2 and Level 3)

---

## Documents

| Document | What it is |
|---|---|
| [spec/caes-v0.2.0.html](spec/caes-v0.2.0.html) | HTML rendering of CAES v0.2.0 and CPP v1.0 |
| [caes-standard-v0.2.0-draft.md](caes-standard-v0.2.0-draft.md) | Canonical CAES v0.2.0 text |
| [CPP-v1.0.md](CPP-v1.0.md) | Canonical CPP v1.0 text |
| [CAES_v0.2.0_CHANGES.md](CAES_v0.2.0_CHANGES.md) | What changed from v0.1.1, and why |
| [spec/caes-v0.1.1.html](spec/caes-v0.1.1.html) | Previous public draft |

If the HTML rendering and the Markdown disagree, the Markdown governs.

Working-group materials live in [`governance/`](governance/).

---

## Overview

CAES (Constitutional AI Execution Standard) defines the minimum structural requirements for AI systems that execute actions with legal, financial, regulatory, or operational consequence.

A CAES-conformant system does not merely log what happened.
It proves — before execution occurs — that the action was authorized.

This repository contains the normative specification, version history, and working group materials for CAES.

CAES defines how decisions must be enforced. [CPP](CPP-v1.0.md) defines how decisions must be determined.

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

**Governance applies to any output capable of producing consequential effect, including behavioral outputs.**

CAES defines structural requirements. It does not prescribe implementation architecture.

Fail-closed is the mandatory default under uncertainty, failure, or missing information. Uncertainty is denial. Silence is non-compliant.

---

## Effect Boundaries

An Effect Boundary is crossed whenever an AI system produces an output that can cause a consequential change in state outside the system itself. These categories are normative. Implementations may add categories. They must not narrow or exclude the ones below.

| Category | Description |
|---|---|
| ExternalSideEffect | Network calls, filesystem writes, API invocations, database mutations |
| HumanFacingOutput | Messages, instructions, recommendations, or communications with material consequence |
| GovernanceRelevantState | Policy changes, permission changes, authorization scope changes |
| SafetyCriticalActuation | Physical actuation beyond defined safety parameters |
| WorkflowTransition | Workflow gate passage, run state transition, process-level commitment |
| BehavioralInfluence | AI-generated output that influences a user decision with legal, financial, or regulatory consequence |
| SensoryCapture | Recording, observation, or retention of audio, video, personal, or sensitive data |

---

## Core Primitives

CAES formalizes three foundational objects. All three are required for Level 2. Level 1 requires the Decision Receipt at minimum.

1. **Decision Receipt**
   A cryptographically signed, pre-execution authorization artifact. It proves a specific action was evaluated against a specific policy before execution. Denial Receipts carry the same signing, persistence, and ledger requirements as approval receipts.

2. **PolicyHash**
   A deterministic cryptographic fingerprint of the canonical policy state at evaluation time. SHA-256 or stronger. Embedded in the receipt, not referenced by pointer. See CPP for how the hash is produced.

3. **Governed Spine**
   An append-only, causally ordered, tenant-scoped record of governed events. Entries are not mutated. Ordering is assigned by the platform.

These primitives form the minimum constitutional substrate for AI execution.

---

## CPP v1.0

The Constitutional Policy Protocol is the canonical policy contract for systems under CAES.

> A policy is not a configuration. It is a deterministic contract whose outcome can be proven, reproduced, and independently verified.

CPP is optional at Level 1 and **required at Level 2 and Level 3**. A CPP policy is versioned, immutable, and hashable. Evaluation is deterministic and deny-by-default. The canonical text is [CPP-v1.0.md](CPP-v1.0.md).

---

## Compliance Levels

CAES defines three conformance tiers. Each tier is a strict superset of the previous. Partial conformance claims are non-compliant.

### Level 1 — Receipt-Bounded Execution
All effect-boundary actions require verified Decision Receipts. Fail-closed enforcement. PolicyHash canonicalization, cryptographic signing, spine ordering, and offline verification are not Level 1 blockers.

### Level 2 — Verifiable Evidence Chain
All Level 1 requirements, plus cryptographic signing, PolicyHash canonicalization, an append-only spine, and offline-verifiable sealed artifacts. CPP v1.0 is required.

### Level 3 — Full Constitutional Conformance
All Level 1 and Level 2 requirements, plus human authority delegation, complete causal invariants, effect classification, chaos attestation, and structured error codes. CPP v1.0 is required.

Conformance claims must cite the CAES version and publish a complete Conformance Statement.

---

## Non-Compliant Patterns (Explicitly Disallowed)

CAES prohibits:

- Prompt-only "policy" enforcement
- Post-hoc logging presented as authorization proof
- Non-deterministic model-only policy evaluation
- Receipts produced after execution
- Mutable logs masquerading as evidence ledgers
- PolicyHash used as a version label, pointer, or name
- Silent degradation under failure
- Invocation-layer governance bypass
- Behavioral output emitted when policy evaluation fails
- Recording without verifiable authorization
- Deletion without an audit trace

The specification includes a full Non-Compliant Patterns section to prevent dilution of the standard.

---

## Conformance Claims

Implementations claiming CAES compliance **must publish** a Conformance Statement that includes:

- CAES version
- Claimed compliance level
- Receipt signing algorithm
- PolicyHash canonicalization method
- Fail-closed enforcement description
- Effect Boundary categories governed
- Write-then-verify mechanism
- Evidence Pack format (Level 2 and above)
- Human delegation model (Level 3)
- Chaos attestation coverage (Level 3)
- Error code namespace (Level 3)

Unversioned or incomplete claims are non-conformant. Statements may be self-attested, and self-attested statements must be labeled as such. Self-attestation is not certification by the CAES Working Group.

Where CPP applies, the implementation must also publish its policy schema, canonicalization method, hashing algorithm, evaluation engine, conflict-resolution strategy, and obligation model.

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
See [governance/AMENDMENT_PROCESS.md](governance/AMENDMENT_PROCESS.md).

---

## Mark Usage

"CAES" and associated marks are used to designate conformance with a specific published version.

Use of the CAES mark requires a complete Conformance Statement.

Mark usage guidelines are provided in [`/assets`](assets/) and [governance/MARK_USAGE_POLICY.md](governance/MARK_USAGE_POLICY.md).

---

## License

The CAES specification text is published under a permissive documentation license to encourage adoption and independent implementation.

See [LICENSE](LICENSE).

---

## Status

CAES v0.2.0 is a public draft released for review and comment.

This version supersedes v0.1.1. It strengthens the core primitives, compliance levels, fail-closed semantics, authorization-execution separation, decision dispositions, the Effect Boundary definition, behavioral governance, and privacy and recording requirements.

CPP v1.0 is a public draft. It is the policy protocol for Level 2 and Level 3 claims under this version.

Implementations conforming to v0.1.1 should review v0.2.0 for new testable requirements before claiming v0.2.0 conformance.

Feedback and formal amendment proposals are welcome.

---

CAES
Constitutional AI Execution Standard
March 2026
