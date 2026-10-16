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
| [Published site](https://red-desert-07272640f.1.azurestaticapps.net/) | Azure Static Web App `swa-caes-eastus2` |
| [spec/caes-v0.2.0.html](spec/caes-v0.2.0.html) | HTML rendering of CAES v0.2.0 and CPP v1.0 |
| [caes-standard-v0.2.0-draft.md](caes-standard-v0.2.0-draft.md) | Canonical CAES v0.2.0 text |
| [CPP-v1.0.md](CPP-v1.0.md) | Canonical CPP v1.0 text |
| [CAES_v0.2.0_CHANGES.md](CAES_v0.2.0_CHANGES.md) | What changed from v0.1.1, and why |
| [governance/CAES_EVIDENCE_PACK_SPEC.md](governance/CAES_EVIDENCE_PACK_SPEC.md) | Evidence Pack annex 0.2.0 |
| [conformance/CAES_v0.2.0_FIXTURES.md](conformance/CAES_v0.2.0_FIXTURES.md) | Interoperability fixtures for this draft |
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

Categories are not mutually exclusive. One action can cross several of them. The composition rules are in the CAES draft under Interoperability Contracts.

---

## Core Primitives

CAES formalizes three foundational objects. All three are required for Level 2. Level 1 requires a Decision Record. A Level 2 Decision Receipt is a signed Decision Record.

1. **Decision Record and Decision Receipt**
   The Decision Record is the pre-effect artifact. It names one disposition and binds the action. At Level 1, checking it means a required field is present and the write-then-verify readback succeeded. A signature is not a Level 1 check. The Decision Receipt adds the JWS profile and the PolicyHash. Denial records use the persistence rules of the claimed level. Signing applies at Level 2 and Level 3.

2. **PolicyHash**
   Required at Level 2 and Level 3. `lowercase-hex(SHA-256(JCS(effective_policy_bundle)))`, with JCS as RFC 8785. The bundle is the declared policy after imports are resolved and defaults are expanded. It is not a Level 1 blocker.

3. **Governed Spine**
   An append-only, causally ordered, tenant-scoped record of governed events. Spine requirements apply at Level 2 and Level 3.

These primitives form the minimum constitutional substrate for AI execution. The applicability matrix in the draft is controlling where an earlier summary is shorter.

---

## CPP v1.0

The Constitutional Policy Protocol is the canonical policy contract for systems under CAES.

> A policy is not a configuration. It is a deterministic contract whose outcome can be proven, reproduced, and independently verified.

CPP is optional at Level 1 and **required at Level 2 and Level 3**. A CPP policy is versioned, immutable, and hashable. Evaluation is deterministic and deny-by-default. The canonical text is [CPP-v1.0.md](CPP-v1.0.md).

---

## Compliance Levels

CAES defines three conformance tiers. Each tier is a strict superset of the previous. Partial conformance claims are non-compliant.

### Level 1 — Receipt-Bounded Execution
All effect-boundary actions require a structurally checked Decision Record before the effect. Fail-closed enforcement. PolicyHash, JWS signatures, the Governed Spine, and the Evidence Pack are not Level 1 blockers.

### Level 2 — Verifiable Evidence Chain
All Level 1 requirements, plus the JWS profile (`EdDSA` or `ES256`), PolicyHash, an append-only spine, and an Evidence Pack at pack spec `0.2.0`. CPP v1.0 is required.

### Level 3 — Full Constitutional Conformance
All Level 1 and Level 2 requirements, plus human authority delegation, complete causal invariants, effect classification, the chaos fixtures, and structured error codes. CPP v1.0 is required. Each chaos mode has its own pass condition.

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
- Receipt signing algorithm (`EdDSA` or `ES256` for Level 2 interchange)
- PolicyHash profile (effective bundle, RFC 8785, SHA-256, lowercase hex)
- Maximum execution lifetime (`exp - iat`, at most 15 minutes for an interchange claim)
- Maximum clock skew (at most 120 seconds for an interchange claim)
- Stream mode (`buffered` or `segmented`)
- Evidence Pack version (`0.2.0` when Level 2 or Level 3 is claimed)
- Fail-closed enforcement description
- Effect Boundary categories governed
- Write-then-verify mechanism
- Evidence Pack format (Level 2 and above)
- Human delegation model (Level 3)
- Chaos attestation coverage (Level 3)
- Error code namespace (Level 3)

Unversioned or incomplete claims are non-conformant. Statements may be self-attested, and self-attested statements must be labeled as such. Self-attestation is not certification by the CAES Working Group.

Where CPP applies, the implementation must also publish its policy schema, canonicalization method, hashing algorithm, evaluation engine, conflict-resolution strategy, obligation model (including phase), tenant-selector rules, and the action-to-disposition mapping.

Interoperability fixtures for this draft are in [conformance/CAES_v0.2.0_FIXTURES.md](conformance/CAES_v0.2.0_FIXTURES.md). The Evidence Pack annex is [governance/CAES_EVIDENCE_PACK_SPEC.md](governance/CAES_EVIDENCE_PACK_SPEC.md).

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
